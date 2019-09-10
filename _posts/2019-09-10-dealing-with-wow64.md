---
layout: post
published: true
categories:
  - personal
featured: false
comments: false
title: Dealing with WOW64
---
## The Problem

Exploit code in general is a problem, it can be rough around the edges, specific to certain environments and libraries, have syntax errors and bugs.  Some Windows-based exploits require an understanding and navigation of 32-bit application compatibility on 64-bit systems, which Windows calls "Windows on Windows" or WOW64.  This also applies to Powershell, which has both 32-bit and 64-bit versions of powershell.exe.

## The Solution

The 64-bit versions of Windows provide an emulation layer for 32-bit applications that provide access to 32-bit versions of libraries, executables, directories and registry keys so that these apps can run transparently without modification. 

This emulation layer is called WOW64 (Windows on Windows) and understanding how it works you can run 64-bit versions of apps from a 32-bit process and vice versa.