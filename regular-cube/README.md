# Regular cube (2 points)

Ahoy, officer

knowledge of regular expressions is crucial for all sailors from CNS fleet. A 3D crossword puzzle is available to
enhance this skill.

May you have fair winds and following seas!

Download [the regular cube crossword](regular_cube.zip).
(MD5 checksum `e0fccb92b8c46b80e75e350503a528ff`)

## Hints

* https://www.rexegg.com/regex-quickstart.html

## Solution

The PDF document extracted from the archive contains the crossword with various regular expressions.

_Note: Unfortunately the document is not very printer friendly :frowning:_

This just needs to be solved manually, incrementally adding known characters until we know them all. The result looks
as follows. We can easily spot the flag in the middle layer.

```text
FOFOO
HELLO
OGTQM
HCATE
OVVWO

FWALO
PTHEB
CATCH
YTHQJ
HAYHO

FLAG{
NICE-
NAVY-
BLUE-
CUBE}

FOZGL
VCESP
GNETB
VDQPK
MVMPX

FHICE
MAYEM
GBYEQ
QERZQ
GQWSY
```
