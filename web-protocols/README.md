# Web protocols (2 points)

Ahoy, officer,

almost all interfaces of the ship's systems are web-based, so we will focus the exercise on the relevant protocols. Your
task is to identify all webs on given server, communicate with them properly and assembly the control string from
responses.

May you have fair winds and following seas!

The webs are running on server `web-protocols.cns-jv.tcc`.

## Hints

* Be aware that `curl` tool doesn't do everything it claims.
* Correct response contains at least one cute cat picture.
* Use VPN to get access to the server.

## Solution

Let's scan the server first to see what listens there.

_Note: It's important to remember that all ports are not scanned by default so it needs to be enforced eg. via `-p-`
argument._

```console
$ nmap -p- web-protocols.cns-jv.tcc
Nmap scan report for web-protocols.cns-jv.tcc (10.99.0.122)
Host is up (0.0087s latency).
Not shown: 65530 closed tcp ports (reset)
PORT     STATE SERVICE
5009/tcp open  airport-admin
5011/tcp open  telelpathattack
5020/tcp open  zenginkyo-1
8011/tcp open  unknown
8020/tcp open  intu-ec-svcdisc

Nmap done: 1 IP address (1 host up) scanned in 15.67 seconds
```

We see that some ports are open. Let's check them from the highest one.

```console
$ curl -v web-protocols.cns-jv.tcc:8020
*   Trying 10.99.0.122:8020...
* Connected to web-protocols.cns-jv.tcc (10.99.0.122) port 8020 (#0)
> GET / HTTP/1.1
> Host: web-protocols.cns-jv.tcc:8020
> User-Agent: curl/7.85.0
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 400 Bad Request
< Server: nginx/1.22.1
< Date: Thu, 12 Oct 2023 10:36:28 GMT
< Content-Type: text/html
< Content-Length: 255
< Connection: close
<
<html>
<head><title>400 The plain HTTP request was sent to HTTPS port</title></head>
<body>
<center><h1>400 Bad Request</h1></center>
<center>The plain HTTP request was sent to HTTPS port</center>
<hr><center>nginx/1.22.1</center>
</body>
</html>
```

Something is listening here, but the response indicates it's HTTPS not HTTP.

```console
$ curl -kv https://web-protocols.cns-jv.tcc:8020
*   Trying 10.99.0.122:8020...
* Connected to web-protocols.cns-jv.tcc (10.99.0.122) port 8020 (#0)
* ALPN: offers h2
* ALPN: offers http/1.1
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* TLSv1.2 (IN), TLS handshake, Certificate (11):
* TLSv1.2 (IN), TLS handshake, Server key exchange (12):
* TLSv1.2 (IN), TLS handshake, Server finished (14):
* TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
* TLSv1.2 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.2 (OUT), TLS handshake, Finished (20):
* TLSv1.2 (IN), TLS handshake, Finished (20):
* SSL connection using TLSv1.2 / ECDHE-RSA-AES256-GCM-SHA384
* ALPN: server accepted h2
* Server certificate:
*  subject: C=US; ST=Oregon; L=Portland; O=Company Name; OU=Org; CN=www.example.com
*  start date: Aug 20 19:33:36 2023 GMT
*  expire date: Aug 19 19:33:36 2024 GMT
*  issuer: C=US; ST=Oregon; L=Portland; O=Company Name; OU=Org; CN=www.example.com
*  SSL certificate verify result: self signed certificate (18), continuing anyway.
* Using HTTP2, server supports multiplexing
* Copying HTTP/2 data in stream buffer to connection buffer after upgrade: len=0
* h2h3 [:method: GET]
* h2h3 [:path: /]
* h2h3 [:scheme: https]
* h2h3 [:authority: web-protocols.cns-jv.tcc:8020]
* h2h3 [user-agent: curl/7.85.0]
* h2h3 [accept: */*]
* Using Stream ID: 1 (easy handle 0x1f2cd1e0440)
> GET / HTTP/2
> Host: web-protocols.cns-jv.tcc:8020
> user-agent: curl/7.85.0
> accept: */*
>
* Connection state changed (MAX_CONCURRENT_STREAMS == 128)!
< HTTP/2 200
< server: nginx/1.22.1
< date: Thu, 12 Oct 2023 10:39:50 GMT
< content-type: text/html; charset=utf-8
< content-length: 542004
< set-cookie: SESSION=Ui00MzNBfQ==; Path=/
<
iVBORw0KGgoAAAA...(output truncated)...AAASUVORK5CYII=
```

We managed to obtain a big base64-encoded string (which decodes to a cute cat PNG picture as the hint indicates) and a
session cookie that also looks like something base64-encoded.

```console
$ echo Ui00MzNBfQ== | base64 -d
R-433A}
```

Indeed, it seems we have a part of the flag. Let's check the other port.

```console
$ curl -v web-protocols.cns-jv.tcc:8011
*   Trying 10.99.0.122:8011...
* Connected to web-protocols.cns-jv.tcc (10.99.0.122) port 8011 (#0)
> GET / HTTP/1.1
> Host: web-protocols.cns-jv.tcc:8011
> User-Agent: curl/7.85.0
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Server: nginx/1.22.1
< Date: Thu, 12 Oct 2023 10:43:53 GMT
< Content-Type: text/html; charset=utf-8
< Content-Length: 523172
< Connection: keep-alive
< Set-Cookie: SESSION=LXJ2YnEtYWJJ; Path=/
<
iVBORw0KGgoAAAA...(output truncated)...ABJRU5ErkJggg==
```

Here we go... another image and another session. Let's try to decode this one too.

```console
$ echo LXJ2YnEtYWJJ | base64 -d
-rvbq-abI
```

It looks like another flag fragment.

The call to port `5020` yields the same content (image and a cookie) as the one on `8020` (even though directly, without
the need for https) and tha call to port `5011` yields the same content as the on port `8011`.

```console
$ curl -v web-protocols.cns-jv.tcc:5009
*   Trying 10.99.0.122:5009...
* Connected to web-protocols.cns-jv.tcc (10.99.0.122) port 5009 (#0)
> GET / HTTP/1.1
> Host: web-protocols.cns-jv.tcc:5009
> User-Agent: curl/7.85.0
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 400 Bad Request
* no chunk, no close, no size. Assume close to signal end
<
Unsupported protocol version
```

Curl seems to be making request using `HTTP/1.1`, however, it seems a different version is required. If we take a closer
look at port numbers, we realize that the last 2 characters are `09`, `11` and `20` which could indicate HTTP protocol
versions (`0.9`, `1.1` and `2`). Upon closer inspection of the requests above we realize, that the response on
port `8020` has really been served via `HTTP/2`.

Since the hint indicates that `curl` might not be able to do everything it claims (eg issuing `HTTP/0.9` request), let's
issue one manually.

```text
─$ echo -e "GET / HTTP/0.9\r\n\r\n" | nc web-protocols.cns-jv.tcc 5009
HTTP/0.9 200 OK

SESSION=RkxBR3trckx0; iVBORw0KGgoAAAA...(output truncated)...ABJRU5ErkJggg==
```

We can decode the third cat picture if we want to, but probably it's just the session ID that we're after.

```console
$ echo RkxBR3trckx0 | base64 -d
FLAG{krLt
```

Now we have all the fragments, so we can just concatenate them together (either the encoded parts in the order of
increasing protocol version number or just the partially decoded results since they seem to decode just fine even on
their own).

```console
$ echo RkxBR3trckx0LXJ2YnEtYWJJUi00MzNBfQ== | base64 -d
FLAG{krLt-rvbq-abIR-433A}
```
