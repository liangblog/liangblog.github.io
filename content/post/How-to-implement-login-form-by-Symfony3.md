---
title: How to implement login form by Symfony3
date: 2017-03-21 22:24:43
tags:
- Ubuntu
- Symfony
- Login
- Security
---

### My computer develop environment
```
Ubuntu 16.04.1 LTS

PHP 7.0.15-0ubuntu0.16.04.4 (cli) ( NTS )
Symfony Installer version 1.5.9
Composer version 1.3.2 2017-01-27 18:23:41

mysql> select version();
+-----------+
| version() |
+-----------+
| 5.7.12    |
+-----------+
```
<!--more-->

### Create a Symfony project
```
symfony new login
```
project name is "login" then wait for symfony download and init this project

Downloading Symfony...
```
191 KiB/5.3 MiB ▓▓░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░    3%  
```

then you will get a project structure such as 
```
login/
├── app
├── bin
├── composer.json
├── composer.lock
├── phpunit.xml.dist
├── README.md
├── src
│   └── AppBundle
│       ├── AppBundle.php
│       └── Controller
│           └── DefaultController.php
├── tests
├── var
├── vendor
└── web

```

### Config the database
```
vim app/config/parameters.yml
```

```
# This file is auto-generated during the composer install
parameters:
    database_host: 127.0.0.1
    database_port: null
    database_name: symfony
    database_user: root
    database_password: root_pwd
    mailer_transport: smtp
    mailer_host: 127.0.0.1
    mailer_user: null
    mailer_password: null
```

you should makesure your database connection information is correct in this file.   


### Create database in mysql by symfony command tool  
```
cd login; php bin/console doctrine:database:create  
```


#### check database in mysql and you can see a new database named symfony which based on your config file app/config/config.yml, it loaded the other config file app/config/parameters.yml

```
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| symfony            |
+--------------------+
```

### Create user_info entity and user_auth entity class
```
php bin/console doctrine:generate:entity --entity AppBundle:User
```

```
The Entity shortcut name [AppBundle:User]:
Configuration format (yml, xml, php, or annotation) [annotation]: 
New field name (press <return> to stop adding fields): truename
Field type [string]: 
Field length [255]: 
Is nullable [false]: 
Unique [false]: 

New field name (press <return> to stop adding fields): 
  Entity generation            
```

```
php bin/console doctrine:generate:entity --entity AppBundle:UserAuth
```

```
The Entity shortcut name [AppBundle:UserAuth]:
Configuration format (yml, xml, php, or annotation) [annotation]: 

New field name (press <return> to stop adding fields): userId
Field type [string]: integer
Is nullable [false]: 
Unique [false]: 

New field name (press <return> to stop adding fields): identifier
Field type [string]: 
Field length [255]: 
Is nullable [false]: 
Unique [false]: 

New field name (press <return> to stop adding fields): credential
Field type [string]: 
Field length [255]: 
Is nullable [false]: 
Unique [false]: 

New field name (press <return> to stop adding fields):                      
  Entity generation  
```

After than you can see the directory structure was changed.
Entity class with mapping information in Symfony help doctrine to build data table in database.
After doctrine schema update, doctrine can automatically create all database tables for every entity.
Repository was generated for the Doctrine ORM.

```
./src/
└── AppBundle
    ├── AppBundle.php
    ├── Controller
    │   └── DefaultController.php
    ├── Entity
    │   ├── UserAuth.php
    │   └── User.php
    └── Repository
        ├── UserAuthRepository.php
        └── UserRepository.php
```

### Create the database table based on entity class

```
php bin/console doctrine:schema:update --force
```

```
mysql> show tables;
+-------------------+
| Tables_in_symfony |
+-------------------+
| user              |
| user_auth         |
+-------------------+
```

After than we can make a web api method to  create a user account instead of a register form (i'm just want to simplify this tutorial)

### Create a Method to sign up my user account
```
vim src/AppBundle/Controller/UserController.php
```

```
namespace AppBundle\Controller;

use AppBundle\Entity\User;
use AppBundle\Entity\UserAuth;
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Symfony\Component\HttpFoundation\Response;

class UserController extends Controller
{

    /**
     * @Route("/user/create",name="user_create")
     */
    public function createAction()
    {
        $em = $this->getDoctrine()->getManager();

        $username = "useraccount";
        $password = "userpassword";


        $User = new User();
        $User->setTruename($username);

        $em->persist($User);
        $em->flush();


        $UserAuth = new UserAuth();
        $UserAuth->setUserId($User->getId());
        $UserAuth->setIdentifier($username);

        $password = $this->get('security.password_encoder')
            ->encodePassword(new \AppBundle\Security\UserAuth($UserAuth), $password);

        $UserAuth->setCredential($password);

        $em->persist($UserAuth);
        $em->flush();
        return new Response("success~");
    }
}
```

Here, we mapping this controller to "/user/create" route. This method will save a user info and a user auth info.

At line 34, you can see how Symfony encodered password and we must create a security userauth class to finish this job.  

```
mkdir -p src/AppBundle/Security
vim src/AppBundle/Security/UserAuth.php
```

```
namespace AppBundle\Security;

use Symfony\Component\Security\Core\User\UserInterface;

class UserAuth implements UserInterface
{
    private $UserAuth;

    public function __construct(\AppBundle\Entity\UserAuth $UserAuth)
    {
        $this->UserAuth = $UserAuth;
    }

    /**
     * Returns the roles granted to the user.
     *
     * <code>
     * public function getRoles()
     * {
     *     return array('ROLE_USER');
     * }
     * </code>
     *
     * Alternatively, the roles might be stored on a ``roles`` property,
     * and populated in any number of different ways when the user object
     * is created.
     *
     * @return (Role|string)[] The user roles
     */
    public function getRoles()
    {
        return ['ROLE_ADMIN'];
    }

    /**
     * Returns the password used to authenticate the user.
     *
     * This should be the encoded password. On authentication, a plain-text
     * password will be salted, encoded, and then compared to this value.
     *
     * @return string The password
     */
    public function getPassword()
    {
        return $this->UserAuth->getCredential();
    }

    /**
     * Returns the salt that was originally used to encode the password.
     *
     * This can return null if the password was not encoded using a salt.
     *
     * @return string|null The salt
     */
    public function getSalt()
    {
    }

    /**
     * Returns the username used to authenticate the user.
     *
     * @return string The username
     */
    public function getUsername()
    {
        return $this->UserAuth->getIdentifier();
    }

    /**
     * Removes sensitive data from the user.
     *
     * This is important if, at any given point, sensitive information like
     * the plain-text password is stored on this object.
     */
    public function eraseCredentials()
    {
    }
}
```

The key named 'ROLE_ADMIN' in method getRoles() approved this Class can be accessed through the Symfony security roles.

Then, update the configuration file app/config/security.yml and tell Symfony security component which encryption to be used.

```
vim app/config/security.yml
```

```
security:
    ...
    encoders:
        AppBundle\Security\UserAuth: bcrypt
```

After the visit [http://localhost:8000/user/create](http://localhost:8000/user/create), it will be added a user name 'useraccount' and password is 'userpassword' in database.Of course, the password is stored in encrypted mode.

```
mysql> select * from user;
+----+-------------+
| id | truename    |
+----+-------------+
|  3 | useraccount |
+----+-------------+
mysql> select * from user_auth;
+----+-------------+--------------------------------------------------------------+--------+
| id | identifier  | credential                                                   | userId |
+----+-------------+--------------------------------------------------------------+--------+
|  3 | useraccount | $2y$13$dLcFHGIKXZ2gl8VbVchD3OpiSVw923DFcyVaiDVnzf9d0tx5nyUny |      3 |
+----+-------------+--------------------------------------------------------------+--------+
```

### Create a page that required login to access
```
vim src/AppBundle/Controller/UserController.php
```

```
class UserController extends Controller
{
    ...
    
    /**
     * @Route("/user/list",name="user_list")
     */
    public function listAction()
    {
        return new Response("user list ~~~");
    }
}    
```

Authentication is required when accessing  "/user/list"
```
vim app/config/security.yml
```

```
security:
    ...
    access_control:
        - { path: ^/user/list, roles: ROLE_ADMIN }
```

### Tell the firewall in Symfony security component to be verified by account using the way to submit a password form  
```
    providers:
        webservice:
            id: app.webservice_user_provider

    firewalls:
        ...
        main:
            anonymous: ~

#            http_basic: ~
            form_login:
                login_path: login
                check_path: login
                use_referer: true
```
Pay attention to line 10, i'm using the "form_login" not "http_basic: ~", you can customize the page for your login form in this way. 
* "login_path": controller for the route redirect to "/login"

```
vim src/AppBundle/Controller/SecurityController.php
```

```
namespace AppBundle\Controller;

use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;

use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Symfony\Component\HttpFoundation\Request;

class SecurityController extends Controller
{

    /**
     * @Route("/login",name="login")
     * @param Request $request
     * @return \Symfony\Component\HttpFoundation\Response
     */
    public function loginAction(Request $request)
    {
        $authenticationUtils = $this->get('security.authentication_utils');

        // get the login error if there is one 
        $error = $authenticationUtils->getLastAuthenticationError();

        // last username entered by the user
        $lastUsername = $authenticationUtils->getLastUsername();

        return $this->render(
            'security/login.html.twig',
            array(
                // last username entered by the user
                'last_username' => $lastUsername,
                'error'         => $error,
            )
        );
    }
}

```
Just copy this contents in this file and you don't need any of modification. 


### Create a Login form 
```
mkdir -p app/Resources/views/security/
vim app/Resources/views/security/login.html.twig
```

```
{% if error %}
    <div>{{ error.messageKey|trans(error.messageData, 'security') }}</div>
{% endif %}

<form action="{{ path('login') }}" method="post">
    <label for="username">Username:</label>
    <input type="text" id="username" name="_username" value="{{ last_username }}" />

    <label for="password">Password:</label>
    <input type="password" id="password" name="_password" />

    {#
        If you want to control the URL the user
        is redirected to on success (more details below)
        <input type="hidden" name="_target_path" value="/account" />
    #}

    <button type="submit">login</button>
</form>

```

Symfony needs "providers" to check the credential submitted in form. 

### Create a user auth service for identifying use login

The name of this service is the same as the id of providers in security.yml.

Arguments in this service which offered Symfony doctrine entity manager is the only place missing in official documents.

Through passing this parameter to "UserAuthProvider", user can be checked exist or not in database in "loadUserByUsername" method 

```
vim app/config/services.yml
```

```
services:
    app.webservice_user_provider:
        class: AppBundle\Security\UserAuthProvider
        arguments:
            - "@doctrine.orm.entity_manager"
```

### Create identify provider for password checking
```
vim src/AppBundle/Security/UserAuthProvider.php
```

```
namespace AppBundle\Security;

use AppBundle\Entity\UserAuth;
use Doctrine\ORM\EntityManager;
use Symfony\Component\Security\Core\Exception\UnsupportedUserException;
use Symfony\Component\Security\Core\Exception\UsernameNotFoundException;
use Symfony\Component\Security\Core\User\UserInterface;
use Symfony\Component\Security\Core\User\UserProviderInterface;

class UserAuthProvider implements UserProviderInterface
{
    /**
     * @var EntityManager
     */
    protected $dm;

    public function __construct(EntityManager $dm)
    {
        $this->dm = $dm;
    }

    public function loadUserByUsername($username)
    {
        /** @var UserAuth $UserAuth */
        $UserAuth = $this->dm->getRepository('AppBundle:UserAuth')->findOneBy(['identifier' => $username]);
        if (!$UserAuth) {
            throw new UsernameNotFoundException("Username \"{$username}\" does not exist.");
        }
        return new \AppBundle\Security\UserAuth($UserAuth);
    }

    public function refreshUser(UserInterface $User)
    {
        if (!$User instanceof \AppBundle\Security\UserAuth) {
            throw new UnsupportedUserException('Instances of "' . get_class($User) . '" are not supported.');
        }

        return $this->loadUserByUsername($User->getUsername());
    }

    public function supportsClass($class)
    {
        return $class === 'AppBundle\Entity\UserAuth';
    }
}
```

OK, after input account and password in [http://localhost:8000/user/list](http://localhost:8000/user/list) ,login success information will be right here!

