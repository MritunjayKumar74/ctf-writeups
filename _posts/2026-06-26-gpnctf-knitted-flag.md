---
layout: post
title: "Knitted Flag"
date: 2026-06-25 00:00:00 +0000
categories: [GPNCTF 2026, Miscellaneous]
tags: [misc, forensics, knitout, python, pixel-art]
---

# Sanity Check — GPNCTF

**Category:** Miscellaneous  
**Flag:** `GPNCTF{conGraTuL4tiONs_You_hAVE_UNdER5700D_kni7OUt_4Nd_uNRAvE1eD_7HE_T4blECL07h5}`

---

## Challenge Description

> I got a new knitting machine to help me with the tablecloths for the restaurant but I accidentally dropped my flag into it. Can you help me unravel it?

**Category:** Miscellaneous

---

## Solution

We're given a `pattern.k` file. The `.k` extension and the `;!knitout-2` magic string at the top reveal it's a **knitout** file — a standard format for knitting machine instructions.

```bash
cat pattern.k | head -5
# ;!knitout-2
# ;;Carriers: 1 2 3 4 5 6
```

The file has 22344 lines and 19555 knit instructions across 20 needles (f1–f20) and 5 carriers. With those dimensions it's almost certainly encoding the flag as pixel art knitted into the fabric — each carrier represents a color, and the pattern of colors across rows forms visible text.

We wrote a Python script to parse the knit instructions and render them as an image, splitting rows by direction changes (`+` → `-`) which matches how a knitting machine passes left and right across the bed:

```python
from PIL import Image

rows = []
row = {}
prev_dir = None

with open('pattern.k', 'r') as f:
    for line in f:
        line = line.strip()
        if not line.startswith('knit'):
            continue
        parts = line.split()
        if len(parts) < 4 or not parts[2].startswith('f'):
            continue
        direction, needle_num, carrier = parts[1], int(parts[2][1:]), int(parts[3])
        if prev_dir is not None and direction != prev_dir and row:
            rows.append(dict(row))
            row = {}
        row[needle_num] = carrier
        prev_dir = direction

if row:
    rows.append(row)

colors = {
    1: (255, 255, 255),
    2: (0, 0, 0),
    3: (0, 0, 0),
    4: (0, 0, 0),
    5: (0, 0, 0),
}

scale = 3
img = Image.new('RGB', (20 * scale, len(rows) * scale), (150, 150, 150))
pixels = img.load()

for y, row in enumerate(rows):
    for x in range(1, 21):
        color = colors.get(row.get(x, 1), (128, 128, 128))
        for dy in range(scale):
            for dx in range(scale):
                pixels[(x-1)*scale + dx, y*scale + dy] = color

img.save('pattern_visual.png')
```

Opening `pattern_visual.png` reveals the flag knitted as pixel art text across the fabric.

---

## Flag

```
GPNCTF{conGraTuL4tiONs_You_hAVE_UNdER5700D_kni7OUt_4Nd_uNRAvE1eD_7HE_T4blECL07h5}
```
