# Naval chef's recipe (2 points)

Ahoy, officer,

some of the crew started behaving strangely after eating the chef's speciality of the day - they apparently have
hallucinations, because they are talking about sirens wailing, kraken on starboard, and accussed the chef being
reptilian spy. Paramedics are getting crazy, because the chef refuses to reveal what he used to make the food. Your task
is to find his secret recipe. It should be easy as the chef knows only security by obscurity and he has registered
domain `chef-menu.galley.cns-jv.tcc`. May you have fair winds and following seas!

The chef's domain is `chef-menu.galley.cns-jv.tcc`.

## Hints

* Use VPN to get access to the server.

## Solution

Let's check what the server returns when queried

```console
$ curl -v chef-menu.galley.cns-jv.tcc
*   Trying 10.99.0.32:80...
* Connected to chef-menu.galley.cns-jv.tcc (10.99.0.32) port 80 (#0)
> GET / HTTP/1.1
> Host: chef-menu.galley.cns-jv.tcc
> User-Agent: curl/7.85.0
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Date: Thu, 12 Oct 2023 20:35:31 GMT
< Server: Apache
< X-Powered-By: PHP/8.2.10
< Vary: Accept-Encoding
< Content-Length: 562
< Content-Type: text/html; charset=UTF-8
<
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
  <title>301 Moved Permanently</title>
  <meta http-equiv="refresh" content="0;url=https://chef-menu.galley.cns-jv.tcc">
</head><body>
<h1>Moved Permanently</h1>
<p>The document has moved <a href="https://chef-menu.galley.cns-jv.tcc">here</a>.</p>
<p style="display: none">The secret ingredient is composed of C6H12O6, C6H8O6, dried mandrake, FLAG{ytZ6-Pewo-iZZP-Q9qz}, and C20H25N3O. Shake, do not mix.</p>
<script>window.location.href='https://chef-menu.galley.cns-jv.tcc'</script>
</body></html>
```

The flag is clearly visible, nothing more to do. I guess this task has been targeted at users opening the address in the
browser that automatically follows `<meta http-equiv="refresh" ...>` instruction. :man_shrugging:
