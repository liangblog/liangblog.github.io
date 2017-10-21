---
title: JQuery-file-upload plugins work with Symfony3
date: 2017-06-14 11:27:41
tags:
- PHP
- Symfony
- JQuery 
---

### My computer develop environment
```
Ubuntu 16.04.1 LTS

PHP 7.0.15-0ubuntu0.16.04.4 (cli) ( NTS )
Symfony Installer version 1.5.9
Composer version 1.3.2 2017-01-27 18:23:41

Symfony Installer version 1.5.9

mysql> select version();
+-----------+
| version() |
+-----------+
| 5.7.12    |
+-----------+
```
<!--more-->

### About what i want in my project
For a new project requirement, people can upload their article picture ***more then 10M*** that only 2M in the past.
Out of consideration for browser compatibility, we choose the jquery-file-upload plugins [blueimp](https://github.com/blueimp/jQuery-File-Upload/) at last

Here is the sample tutorial about how this plugin works with a symfony3 project

## code in html
```
<input type="file" 
       accept="image/jpg,image/jpeg,image/png,image/gif"
       class="uploadBigPictures" name="file">
       
<script type="text/javascript" src="{{ asset('js/jquery-ui/jquery.ui.widget.js') }}"></script>
<script type="text/javascript" src="{{ asset('js/jquery-file-upload/jquery.iframe-transport.js') }}"></script>
<script type="text/javascript" src="{{ asset('js/jquery-file-upload/jquery.fileupload.js') }}"></script>
```

```
<script>
 $('.uploadBigPictures').fileupload({
                        maxChunkSize: 15000000, //set the max file size is very important!
                        url: uploadUrl,
                        success: function (data) {
                            // do what upload request finished 
                        },
                        progressall: function (e, data) {
                            // show the progress during uploading
                            var progress = parseInt(data.loaded / data.total * 100, 10);
                            $('#progress .bar').css(
                                'width',
                                progress + '%'
                            );
                        }
                    });
</script>                    
```

Pay attention to the setting of '***maxChunkSize***' in this script code, i have uploaded 12M picture for testing without setting up at the beginning. but empty file content was received in image server api.
i checked the settings about the file uploaded max size in nginx and apache configuration file but it dons't help, it 's wasting my time 

## code in php

```

    /**
     * @Route(
     *     "/image/big/upload",
     *     name="img_big_upload"
     * )
     * @internal param Request $Request
     * @param Request $request
     * @return JsonResponse
     */
    public function uploadBigAction(Request $request)
    {
        try {
            $uploadedFile = $request->files->get('file');

            $ext = '.' . $uploadedFile->guessExtension();
            $imgName = time() . $ext;

            $dir = $this->urlToDirPath($imgName);
            $fileName = $this->urlToFileName($imgName) . $ext;

            $uploadedFile->move($dir, $fileName);

            return $this->json([
                'success' => true,
            ]);
        } catch (\Exception $e) {
            return $this->json(['success' => false, 'message' => $e->getMessage()]);
        }
    }
```

