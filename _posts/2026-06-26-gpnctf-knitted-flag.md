---
layout: post
title: "Knitted Flag"
date: 2026-06-25 00:00:00 +0000
categories: [GPNCTF 2026, Miscellaneous]
tags: [misc, forensics, knitout, python, pixel-art]
---

# Knitted Flag — GPNCTF

**Category:** Miscellaneous  
**Flag:** `GPNCTF{conGraTuL4tiONs_You_hAVE_UNdER5700D_kni7OUt_4Nd_uNRAvE1eD_7HE_T4blECL07h5}`

---

## Challenge Description

> I got a new knitting machine to help me with the tablecloths for the restaurant but I accidentally dropped my flag into it. Can you help me unravel it?

**Category:** Miscellaneous

---

## Solution

I was given a `pattern.k` file. The `.k` extension and the `;!knitout-2` magic string at the top revealed that it's a **knitout** file — a standard format for knitting machine instructions.

```bash
cat pattern.k | head -5
# ;!knitout-2
# ;;Carriers: 1 2 3 4 5 6
```

I then generated a Python script to parse the knit instructions and render them as an image.

```python
from PIL import Image

rows = []
current_row = [0] * 21

prev_direction = None
needle_set = set()

with open('pattern.k', 'r') as f:
    lines = f.readlines()

knit_lines = []
for line in lines:
    line = line.strip()
    if not line.startswith('knit'):
        continue
    parts = line.split()
    if len(parts) < 4:
        continue
    needle = parts[2]
    if not needle.startswith('f'):
        continue
    direction = parts[1]
    needle_num = int(needle[1:])
    carrier = int(parts[3])
    knit_lines.append((direction, needle_num, carrier))

row = {}
prev_dir = None
for direction, needle_num, carrier in knit_lines:
    if prev_dir is not None and direction != prev_dir and len(row) > 0:
        rows.append(dict(row))
        row = {}
    row[needle_num] = carrier
    prev_dir = direction

if row:
    rows.append(row)

print(f"Total rows: {len(rows)}")

colors = {
    1: (255, 255, 255),
    2: (0, 0, 0),
    3: (0, 0, 0),
    4: (0, 0, 0),
    5: (0, 0, 0),
}

width = 20
height = len(rows)
scale = 3

img = Image.new('RGB', (width * scale, height * scale), (150, 150, 150))
pixels = img.load()

for y, row in enumerate(rows):
    for x in range(1, 21):
        carrier = row.get(x, 1)
        color = colors.get(carrier, (128, 128, 128))
        for dy in range(scale):
            for dx in range(scale):
                pixels[(x-1)*scale + dx, y*scale + dy] = color

img.save('pattern_visual.png')
print("Saved pattern_visual.png")
```

Opening `pattern_visual.png` reveals the flag knitted as pixel art text across the fabric.

![Pattern Visual](../assets/img/gpnctf-2026/knitted-flag/pattern_visual.png)

---

## Flag

```
GPNCTF{conGraTuL4tiONs_You_hAVE_UNdER5700D_kni7OUt_4Nd_uNRAvE1eD_7HE_T4blECL07h5}
```
