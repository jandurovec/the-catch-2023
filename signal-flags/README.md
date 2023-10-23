# Signal flags (5 points)

Ahoy, officer,

after a reboot at 2023-10-02 11:30:18 UTC, the On-board signal flag recognition system (OBSF-RS) has malfunctioned. The
signal flags are no more recognized and the only working function is to generate and save schematic views, which are
created every time a ship in the vicinity changes its signaling. We need to know what the surrounding ships signaled and
if we have missed something important.

May you have fair winds and following seas!

Download the [schematic views](signal_flags.zip).
(MD5 checksum: `c1820a3c41a0d6a0daacd84b69fd1b57`)

## Hints

* Two consecutive signalization of the same vessel in a time interval of less than 2 minutes are part of one message.

## Solution

When we download and extract the provided archive, we can see it contains a lot of images. Each image seems to contain
the following information:

* timestamp
* ship ID
* signal flag count
* signal type
* signal flags
* ensign flag

What we need to do is to extract this information from images and read/re-assemble the messages. For text OCR we can use
e.g. `tesseract-ocr`.

We probably could do the same for signal flags, however, I was not able to find any pre-trained model, so it seemed
easier to just write the "OCR" myself (not super efficient, but works :wink:), especially since the signal flags

* are surrounded by distinctive green border (and other image parts do not come inside)
* can be read top to bottom
* always have the same width (not height though)

After extracting the message fragments I realized that some of the fragments consist of actual characters and some of
them match the `0x[0-9a-f]+` pattern which indicates they are bytes and can be converted to ASCII.

Then it was just a matter of sorting messages by time and grouping them by ship ID to get individual messages.

The final code I ended up with is as follows.

_Note: In the end I realized that I don't need to use signal type, ensign flags and flag count (OK, I could optimize
here...), so these parts have been removed from the program._

```python
import os
from hashlib import md5
from pathlib import Path

import pytesseract
from PIL import Image

GREEN = (0, 196, 0)
OUTLINE_WIDTH = 3
ALPHABET_FLAGS_DIR = "alphabet"
ALPHABET_FLAG_WIDTH = 81
SOURCE_DIR = "."
TIMESTAMP_POS = (270, 250, 630, 290)
SHIP_ID_POS = (300, 300, 440, 335)


def is_tl_flag_corner(px, x, y):
    return (px[x, y] == GREEN
            and px[x - 1, y] != GREEN
            and px[x, y - 1] != GREEN
            and all(px[x + OUTLINE_WIDTH, y + dy] == GREEN for dy in range(OUTLINE_WIDTH))
            and all(px[x + dx, y + OUTLINE_WIDTH] == GREEN for dx in range(OUTLINE_WIDTH)))


def extract_flag_size(px, tl_x, tl_y):
    w = 1
    h = 1
    while all(px[tl_x + w, tl_y + dy] == GREEN for dy in range(OUTLINE_WIDTH)):
        w += 1
    while all(px[tl_x + dx, tl_y + h] == GREEN for dx in range(OUTLINE_WIDTH)):
        h += 1
    return w, h


def extract_naval_flags(img, alphabet, errors):
    message = ''
    wid, hgt = img.size
    px = img.load()
    for y in range(1, hgt - OUTLINE_WIDTH):
        for x in range(1, wid - OUTLINE_WIDTH):
            if is_tl_flag_corner(px, x, y):
                w, h = extract_flag_size(px, x, y)
                flag = img.crop((x, y, x + w, y + h))
                md5hash = md5(flag.tobytes()).hexdigest()
                if w == ALPHABET_FLAG_WIDTH:
                    if md5hash in alphabet:
                        message += alphabet[md5hash]
                    else:
                        flag.save(Path(ALPHABET_FLAGS_DIR, md5hash + ".png"))
                        errors.append(md5hash)
                else:
                    # print(f'Ignoring flag at {x, y} because of incorrect size ({w}x{h})')
                    pass
                break
    return message


def decode_hex(s):
    result = ''
    for i in range(2, len(s), 2):
        result += chr(int(s[i:i + 2], 16))
    return result


def load_dict(path):
    result = {}
    for file in list(Path(path).rglob("*.png")):
        filename, _ = os.path.splitext(os.path.basename(file))
        result[md5(Image.open(str(file)).tobytes()).hexdigest()] = filename
    return result


def solve_signal_flags():
    alphabet = load_dict(ALPHABET_FLAGS_DIR)
    unknown_flags = []
    data = []
    ships = set()
    for file in list(Path(SOURCE_DIR).rglob("signalization_*.png")):
        img = Image.open(str(file))
        time = pytesseract.image_to_string(img.crop(TIMESTAMP_POS)).strip()
        ship_id = pytesseract.image_to_string(img.crop(SHIP_ID_POS)).strip()
        message = extract_naval_flags(img, alphabet, unknown_flags)
        if message.startswith('0x'):
            message = decode_hex(message)
        data.append((time, ship_id, message))
        ships.add(ship_id)
    if unknown_flags:
        exit(f'Unknown flags detected {unknown_flags}. Rename files from hashes to actual names and re-run')

    split_comms = {}
    for comm in sorted(data):
        ship_id = comm[1]
        message = comm[2]
        if ship_id in split_comms:
            split_comms[ship_id] += message
        else:
            split_comms[ship_id] = message
    for ship_id, message in split_comms.items():
        print(f'#{ship_id}: {message}')


solve_signal_flags()
```

After the first run the program saves the identified unique flags into a separate directory and needs user to rename
them according to their meaning, i.e. the file name is used as OCR result (luckily, all flags are alpha-numeric and
there are no spaces, slashes, periods,...).

When we run a program we get the following

```console
$ python read_flags.py
#259798: honeymoon
#304622: mCNS Josef Verich, nice flagCNS Josef Verich, is Mr. Caletka on board?Have you seen Mr. Caletka?
#367676: xParty on the bridge at 2100 ZULU.Free pinot gris for everyone!
#487078: vOur coffee machine is broken, we need a mechanic.vf
#567874: ekHey, what is your wifi Password?Relay: Have you seen Mr. Caletka?
#627912: peRelay: Have you seen Mr. Caletka?
#717609: CNS Josef verich, are your nets ok, too? ;-)CNS Josef verich, you can improve them by RkxBR3tsVHJHLTNvWG4tYW9aTi1aNHFNfQ== !
#745387: kklCan you sell us some ipv4 addresses?Relay: Have you seen Mr. Caletka?
#782535: bvCNS Josef Verich, can your network guy set up a router for us?bv
```

We can see that ship `#717609` is broadcasting something to improve our nets. Let's decode it.

```console
$ echo RkxBR3tsVHJHLTNvWG4tYW9aTi1aNHFNfQ== | base64 -d
FLAG{lTrG-3oXn-aoZN-Z4qM}
```
