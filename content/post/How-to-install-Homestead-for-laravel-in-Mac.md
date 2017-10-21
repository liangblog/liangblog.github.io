---
title: How to install Homestead for laravel in Mac
date: 2017-09-03 13:49:14
tags:
- Mac 
- brew
- vagrant 
- homestead
- laravel
---

### My computer develop environment
```
$ sw_vers 
ProductName:    Mac OS X
ProductVersion: 10.12.6
BuildVersion:   16G29

$ brew -v
Homebrew 1.3.0
Homebrew/homebrew-core (git revision 83c2; last commit 2017-08-03)
```
<!--more-->

### Install tools

Vagrant uses Virtualbox to manage the virtual dependencies. You can directly download virtualbox and install or use homebrew for it.
```
$ brew cask install virtualbox
$ brew cask install vagrant
```
Vagrant-Manager helps you manage all your virtual machines in one place directly from the menubar.
```
$ brew cask install vagrant-manager
```
After this virtualbox、vagrant and vagrant-manager have been installed under brew cask list
```
$ brew cask list
vagrant            Vagrant-Manager            virtualbox
```


### Add Homestead box to vagrant
```
$ vagrant box add laravel/homestead
```
The speed is too slow to cause I can't download this box, if you also meet this situation. Get it by 'wget' then put it to the box 's directory 

Get download url in infomation
```
Bringing machine 'homestead-7' up with 'virtualbox' provider...
==> homestead-7: Box 'laravel/homestead' could not be found. Attempting to find and install...
    homestead-7: Box Provider: virtualbox
    homestead-7: Box Version: >= 3.0.0
==> homestead-7: Loading metadata for box 'laravel/homestead'
    homestead-7: URL: https://vagrantcloud.com/laravel/homestead
==> homestead-7: Adding box 'laravel/homestead' (v3.0.0) for provider: virtualbox
    homestead-7: Downloading: https://vagrantcloud.com/laravel/boxes/homestead/versions/3.0.0/providers/virtualbox.box
```
Download laravel vagrant box 
```
$ cd ~/.vagrant.d/boxes/; wget -c https://vagrantcloud.com/laravel/boxes/homestead/versions/3.0.0/providers/virtualbox.box
```
Add offline box 
```
$ vagrant box add laravel/homestead ./virtualbox.box
```

### Install Homestead
```
$ cd ~/
$ git clone https://github.com/laravel/homestead.git Homestead
$ cd Homestead
$ bash init.sh
```

### Config Homestead
Because we use virtualbox as virtual environment

```
$ cd ~/Homestead
$ vim Homestead.yaml  
```

So change the provider:
```
    provider: virtualbox
```

Homestead using the nginx website server 
Look at the config file about share directory and nginx site directory

```
folders:
    - map: ~/code/php/laravelDemo
      to: /home/vagrant/Code

sites:
    - map: homestead.app
      to: /home/vagrant/Code/public
```

I will share my directory "~/code/php/laravelDemo" on the hard disk to virtual directory 
And the nginx will direct site directory to "/home/vagrant/Code/public" in it's config file named "homestead.app"

Now start the homestead virtual machine
```
$ vagrant up
```

If vagrant can not find the local box and show a error message "Box 'laravel/homestead' could not be found", please check the laravel virtualbox verion first.
```
$ vagrant box list
laravel/homestead (virtualbox, 0)    
```
My box's version is 0, which was downloaded before 
So modify the Homestead script

```
$ vim ~/Homestead/scripts/homestead.rb
```

change box min version in the script

```
#        config.vm.box_version = settings["version"] ||= ">= 3.0.0"
        config.vm.box_version = settings["version"] ||= ">= 0"
```

then 
```
$ vagrant up
```

Now, you can see a virtual machine running in VirtualBox  
Open the url "http://192.168.10.10/" in browser that you can see some information about laravel

### Add homestead to your project
```
$ cd ~/code/php/AnotherLaravelDemo;
$ composer require laravel/homestead --dev
$ php vendor/bin/homestead make
```

There will be a new Homestead.yaml file under your project
```
$ vagrant up
```
You can see the new project config information for this project

