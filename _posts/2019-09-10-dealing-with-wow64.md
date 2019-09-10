---
layout: post
published: false
featured: true
comments: false
title: Dealing with WOW64
headline: Dealing with WOW64
description: Understanding the Windows on Windows (WOW) subsystem.
modified: '2019-03-01'
categories:
  - hacking
  - windows
tags: hacking windows wow64 powershell sysnative
---
## The Problem

Exploit code can be challenging to make work, it can be rough around the edges, specific to certain environments and libraries, and have syntax errors and bugs.  Some Windows-based exploits require an understanding and navigation of 32-bit application compatibility on 64-bit systems, which Windows calls "Windows on Windows" or WOW64.  This also applies to Powershell, which has both 32-bit and 64-bit versions of powershell.exe.

## The Solution

The 64-bit versions of Windows provide an emulation layer for 32-bit applications that provide access to 32-bit versions of libraries, executables, directories and registry keys so that these apps can run transparently without modification. 

This emulation layer is called WOW64 (Windows on Windows) and by understanding how it works, you can run 64-bit versions of apps from a 32-bit process and vice versa.

## Detecting 64-bit Windows

There are several ways to detect a 64-bit version of Windows, one of which is to use the _set_ command to display environment variables and look for processor architecture. Another simple way is to verify the presence of the _Program Files (x86)_ directory, which only exists on 64-bit versions of Windows.

## File Redirection for 32-Bit Apps

Windows 64-bit libraries and executables are located in the _%SYSTEMROOT%\system32_ directory.  The name may seem confusing, but was done to maintain backwards compatibility because of hard coded paths.  

Windows 32-bit libraries and executables are located in the _%SYSTEMROOT%\syswow64_ directory, but instead of having to modify paths in source code, WOW64 transparent redirects 32-bit app calls from _%SYSTEMROOT%\system32_ to _%SYSTEMROOT%\syswow64_.

Here's what you really need to know:

A 32-bit app can also explicitly access 64-bit versions using the %SYSTEMROOT%\sysnative path (not visible in Windows Explorer).