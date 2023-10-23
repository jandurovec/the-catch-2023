# Sonar logs (2 points)

Ahoy, officer,

each crew member must be able to operate the sonar and understand its logs. Your task is to analyze the given log file,
and check out what the sonar has seen.

May you have fair winds and following seas!

Download [the logs](sonar_logs.zip).
(MD5 checksum: `b946f87d0231fcbdbc1e76e27ebf45c7`)

## Hints

* The entry in the log consists of two parts - the timestamp and the message.
* Update: Be aware that some devices do not use up-to-date libraries - this sonar, for example, is based on python and
  uses an old `pytz` library version 2020.4.

## Solution

The extracted log file contains various entries that look like this

```console
$ head -10 sonar.log
2023-10-01 22:51:22 Pacific/Honolulu - Transmitters activated
2023-10-02 00:32:51 US/Pacific - Receiver 5 up
2023-10-02 00:33:45 America/Vancouver - Preprocessor up
2023-10-02 00:52:22 America/Creston - Object detected in depth 65 (0x41)
2023-10-02 01:30:48 America/Belize - Power generator up
2023-10-02 01:34:28 America/Inuvik - Graphical UI up
2023-10-02 01:34:59 America/Mazatlan - Transmitters activated
2023-10-02 01:42:58 Mexico/BajaSur - Transmitters activated
2023-10-02 01:49:54 US/Pacific - Object detected in depth 114 (0x72)
2023-10-02 01:51:29 America/Sitka - Transmitters activated
```

The lines with `Object detected` look suspicious. Let's check how many there are.

```console
$ grep "Object detected" sonar.log | wc -l
25
```

25 is good as it matches the expected flag length. Let's try extracting and converting them.

```console
$ grep -o "Object detected in depth [0-9]\+" sonar.log | awk '{printf "%c", $5}'
ArGbjF{wm2WLYWL3Z}-o-A-KG
```

Unfortunately when extracting them and converting them to characters using depth as ASCII code, the flag does not look
OK. It seems the characters are correct, however, they are shuffled.

On the other hand, the logs seem to contain also timezone information. Even though the log looks sorted based on time,
it is not sorted if the time value represents a local time in a given timezone. Let's write a program to sort the lines
based on real time.

_Note: As the hint suggests, the expected sorting in the file uses timezone info from a particular version of `pytz`
library. That means, that sorting using some other language or using newer lib versions might not yield the result the
authors anticipated._

```python
import re
from datetime import datetime

import pytz

data_re = re.compile(r"Object detected in depth (\d+)")


class SonarData:
    def __init__(self, orig):
        d, self.message = orig.split(" - ")
        last_space = d.rindex(" ")
        self.time = d[:last_space]
        self.tz = pytz.timezone(d[last_space + 1:])
        self.utc = self.tz.localize(datetime.strptime(self.time, "%Y-%m-%d %H:%M:%S")).astimezone(tz=pytz.utc)

    def __repr__(self):
        match = data_re.search(self.message)
        if match:
            return chr(int(match.group(1)))
        else:
            return ""


f = open("sonar.log", "r")
data = map(lambda l: SonarData(l), f.read().splitlines())
data = filter(lambda d: str(d) != "", data)

for o in sorted(data, key=lambda x: x.utc):
    print(o, end="")
```

The characters now appear in the correct order and the flag is revealed.

```console
$ python analyze-sonar.py
FLAG{3YAG-2rbj-KWoZ-LwWm}
```
