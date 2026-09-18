Reflected XSS arises when an application receives data in an HTTP request and includes that data within the immediate response in an unsafe way.

Suppose a website has a search function which receives the user-supplied search term in a URL parameter :

```shell
https://insecure-website.com/search?term=gift
```

The application echoes the supplied search term in the response to this URL :

```shell
<p>You searched for: gift</p>
```

Assuming the application doesn't perform any other processing of the data, an attacker can construct an attack like this :

```shell
https://insecure-website.com/search?term=<script>/*+Bad+stuff+here...+*/</script>
```

This URL results in the following response :

```shell
<p>You searched for: <script>/* Bad stuff here... */</script></p>
```