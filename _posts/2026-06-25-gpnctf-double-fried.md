---
layout: post
title: "Double Fried"
date: 2026-06-25 00:00:00 +0000
categories: [GPNCTF 2026, Miscellaneous]
tags: [misc, forensics, pcap, syslog]
---

## Challenge Description

> I was planning to go to dinner with a friend but something felt off. Can you help me sort everything out?

**Category:** Miscellaneous

---

## Solution

I was given a `.tar.gz` archive. When i extracted it, it revealed a single file — `kitchen_log.pcap`, a network packet capture. The challenge description hints at needing to "sort everything out". So, when i opened the PCAP in Wireshark (or `tshark`), it revealed Syslog traffic from a device called `kitchen-01`. Each packet contains a log message tagged sequentially as `R0001`, `R0002`, etc. These packets were out of order.

```bash
    1   0.000000    10.0.0.10 → 192.168.1.1  Syslog 119 USER.INFO: 1 2023-11-15T00:00:00.000Z kitchen-01 kitchen 1337 R0001 - KITchen ready!
    2   2.356112    10.0.0.10 → 192.168.1.1  Syslog 120 USER.INFO: 1 2023-11-15T00:00:02.356Z kitchen-01 kitchen 1337 R0002 - Awaiting orders
    3   5.121706    10.0.0.10 → 192.168.1.1  Syslog 134 USER.INFO: 1 2023-11-15T00:00:05.122Z kitchen-01 kitchen 1337 R0003 - Got order for a loaded binary
    4   7.602726    10.0.0.10 → 192.168.1.1  Syslog 121 USER.INFO: 1 2023-11-15T00:00:07.603Z kitchen-01 kitchen 1337 R0004 - Order delivered!
    5  10.400774    10.0.0.10 → 192.168.1.1  Syslog 131 USER.INFO: 1 2023-11-15T00:00:10.401Z kitchen-01 kitchen 1337 R0005 - WARNING: Sous chef injured
    6  12.987238    10.0.0.10 → 192.168.1.1  Syslog 134 USER.INFO: 1 2023-11-15T00:00:12.987Z kitchen-01 kitchen 1337 R0006 - Order for some fries received
    7  15.206323    10.0.0.10 → 192.168.1.1  Syslog 119 USER.INFO: 1 2023-11-15T00:00:15.206Z kitchen-01 kitchen 1337 R0007 - Sent out fries
    8  17.677496    10.0.0.10 → 192.168.1.1  Syslog 124 USER.INFO: 1 2023-11-15T00:00:17.677Z kitchen-01 kitchen 1337 R0008 - Received fries back
    9  19.887974    10.0.0.10 → 192.168.1.1  Syslog 213 USER.INFO: 1 2023-11-15T00:00:19.888Z kitchen-01 kitchen 1337 R0009 - You're completely right! I'm sorry, I should not have added that sous chef's finger to the fries. I'll retry
   10  22.114171    10.0.0.10 → 192.168.1.1  Syslog 131 USER.INFO: 1 2023-11-15T00:00:22.114Z kitchen-01 kitchen 1337 R0010 - Fryer oil exploded. yay...
   11  24.371086    10.0.0.10 → 192.168.1.1  Syslog 125 USER.INFO: 1 2023-11-15T00:00:24.371Z kitchen-01 kitchen 1337 R0011 - Fries sent out again
   12  26.724189    10.0.0.10 → 192.168.1.1  Syslog 109 USER.INFO: 1 2023-11-15T00:00:26.724Z kitchen-01 kitchen 1337 F0001 - Beep
   13  29.474768    10.0.0.10 → 192.168.1.1  Syslog 126 USER.INFO: 1 2023-11-15T00:00:29.475Z kitchen-01 kitchen 1337 R0012 - Noticed weird beeping
   14  31.988467    10.0.0.10 → 192.168.1.1  Syslog 151 USER.INFO: 1 2023-11-15T00:00:31.988Z kitchen-01 kitchen 1337 R0013 - Received an order for a very delicious flag :)
   15  34.358962    10.0.0.10 → 192.168.1.1  Syslog 153 USER.INFO: 1 2023-11-15T00:00:34.359Z kitchen-01 kitchen 1337 R0014 - For security I'll send out the flag char by char
   16  36.676942    10.0.0.10 → 192.168.1.1  Syslog 121 USER.INFO: 1 2023-11-15T00:00:36.677Z kitchen-01 kitchen 1337 R0015 - Have fun with it
   17  41.405084    10.0.0.10 → 192.168.1.1  Syslog 106 USER.INFO: 1 2023-11-15T00:00:41.405Z kitchen-01 kitchen 1337 R0016 - G
   18  41.493306    10.0.0.10 → 192.168.1.1  Syslog 106 USER.INFO: 1 2023-11-15T00:00:41.611Z kitchen-01 kitchen 1337 R0018 - N
   19  41.611235    10.0.0.10 → 192.168.1.1  Syslog 106 USER.INFO: 1 2023-11-15T00:00:41.493Z kitchen-01 kitchen 1337 R0017 - P
   20  41.698824    10.0.0.10 → 192.168.1.1  Syslog 106 USER.INFO: 1 2023-11-15T00:00:41.742Z kitchen-01 kitchen 1337 R0020 - T
   21  41.742073    10.0.0.10 → 192.168.1.1  Syslog 106 USER.INFO: 1 2023-11-15T00:00:41.699Z kitchen-01 kitchen 1337 R0019 - C
   22  41.809686    10.0.0.10 → 192.168.1.1  Syslog 106 USER.INFO: 1 2023-11-15T00:00:41.938Z kitchen-01 kitchen 1337 R0022 - {
   23  41.937540    10.0.0.10 → 192.168.1.1  Syslog 106 USER.INFO: 1 2023-11-15T00:00:41.810Z kitchen-01 kitchen 1337 R0021 - F
...
```
Notice that the syslog messages starting from R0016 contains tha flag format, but they are out of order. I used `tshark` to extract and sort the Syslog messages by record number:

```bash
tshark -r kitchen_log.pcap -T fields -e syslog.msg | grep -oP 'R\d{4}.*' | sort -V
```

Output - 

```bash
R0001 - KITchen ready!
R0002 - Awaiting orders
R0003 - Got order for a loaded binary
R0004 - Order delivered!
R0005 - WARNING: Sous chef injured
R0006 - Order for some fries received
R0007 - Sent out fries
R0008 - Received fries back
R0009 - You're completely right! I'm sorry, I should not have added that sous chef's finger to the fries. I'll retry
R0010 - Fryer oil exploded. yay...
R0011 - Fries sent out again
R0012 - Noticed weird beeping
R0013 - Received an order for a very delicious flag :)
R0014 - For security I'll send out the flag char by char
R0015 - Have fun with it
R0016 - G
R0017 - P
R0018 - N
R0019 - C
R0020 - T
R0021 - F
R0022 - {
R0023 - N
R0024 - i
R0025 - c
R0026 - E
R0027 - ,
R0028 -
R0029 - y
R0030 - o
R0031 - U
R0032 -
R0033 - f
R0034 - o
R0035 - u
R0036 - N
R0037 - D
R0038 -
R0039 - 0
R0040 - u
R0041 - T
R0042 -
R0043 - W
R0044 - h
R0045 - O
R0046 -
R0047 - d
R0048 - I
R0049 - d
R0050 -
R0051 - n
R0052 - 0
R0053 - t
R0054 -
R0055 - B
R0056 - E
R0057 - l
R0058 - O
R0059 - N
R0060 - 6
R0061 -
R0062 - t
R0063 - h
R0064 - E
R0065 - r
R0066 - E
R0067 - }
R0068 - You now have received your flag
R0069 - I hope it tastes good.
```

---

## Flag

```
GPNCTF{NicE, yoU fouND 0uT WhO dId n0t BElON6 thErE}
```

---
