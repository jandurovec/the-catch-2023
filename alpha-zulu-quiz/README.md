# Alpha-Zulu quiz (3 points)

Ahoy, officer

your task is to pass a test that is part of the crew drill. Because of your rank of third officer, you must lead the
crew by example and pass the test without a single mistake.

May you have fair winds and following seas!

The quiz webpage is available at http://quiz.cns-jv.tcc.

## Hints

* Use VPN to get access to the quiz web.

## Solution

The website hosts a quiz form that asks us to identify various blobs (in random order). They can be identified by some
known file beginnings/substrings (`MZ`, `PK`), using e.g. `file` tool, by length (hashes) or simply by the process of
elimination.

The answers are:

![Base32 encoded data](base32-encoded-data.png)
![Base64 encoded data](base64-encoded-data.png)
![.NET ViewState value](dotnet-viewstate-value.png)
![ELF binary](elf-binary.png)
![Encoded PowerShell command](encoded-powershell-command.png)
![GZip hex stream](gzip-hex-stream.png)
![Java Serialized data](java-serialized-data.png)
![Java Serialized hex stream](java-serialized-hex-stream.png)
![JSON Web Token](json-web-token.png)
![Linux x86 Shellcode](linux-x86-shellcode.png)
![MD5 sum](md5-sum.png)
![Microsoft EVTX file signature](ms-evtx-file-signature.png)
![Microsoft Windows executable binary](ms-windows-executable-binary.png)
![PHP serialized object](php-serialized.png)
![SHA1 sum](sha1-sum.png)
![UNIX timestamp](unix-timestamp.png)
![UTF-16 Little Endian encoded data](utf-16-le-encoded-data.png)
![Wordpress hash](wordpress-hash.png)
![XOR obfuscated string](xor-obfuscated-string.png)
![ZIP archive](zip-archive.png)

After successfully answering all the questions the quiz continues, but the flag is revealed.
![FLAG](flag.png)
