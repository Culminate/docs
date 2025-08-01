---
title: Безопасная разработка
description: 
published: 1
date: 2025-08-01T20:34:48.693Z
tags: 
editor: markdown
dateCreated: 2025-08-01T20:34:48.693Z
---

# Безопасная разработка

## libc++ hardening

- Options included in both:
  - -D_FORTIFY_SOURCE=3
  - -D_GLIBCXX_ASSERTIONS
  - -Wl,-z,relro,-z,now
  - -fstack-protector-strong
  - -fstack-clash-protection
- Options included only by GCC -fhardened:
  - -ftrivial-auto-var-init=zero
  - -fPIE -pie
  - -fcf-protection=full (x86 GNU/Linux only)
- Options included only by the OpenSSF guide:
  - -O2 -Wall -Wformat -Wformat=2 -Wconversion -Wimplicit-fallthrough
  - -Werror=format-security
  - -fstrict-flex-arrays=3
  - -Wl,-z,nodlopen
  - -Wl,-z,noexecstack
  - -Wl,--as-needed
  - -Wl,--no-copy-dt-needed-entries


https://libcxx.llvm.org/Hardening.html
https://sourceware.org/glibc/manual/2.40/html_node/Dynamic-Linker-Hardening.html
https://www.gnu.org/software/libc/manual/html_node/Source-Fortification.html
https://best.openssf.org/Compiler-Hardening-Guides/Compiler-Options-Hardening-Guide-for-C-and-C++.html
https://gcc.gnu.org/wiki/LibstdcxxDebugMode

## санитайзеры

...

## юнит тесты

...

## -W флаги компиляции

...

