---
title: Shared library
description: 
published: true
date: 2023-09-06T09:10:26.816Z
tags: 
editor: markdown
dateCreated: 2023-07-31T11:06:00.958Z
---

# Shared Library

## Windows vs Linux

Разделяемые библиотеки в Windows и Linux различаются фундаментально.

В Linux разделяемая библиотека (so) ведёт себя как единное приложение с исполняемым файлом и другими библиотеками. Код в исполняемом файле и библиотеках в Linux подчиняется стандартам C++ как единная программа.
В Windows разделяемая библиотека (dll) ведёт себя как отдельное приложение со своей памятью.
В Linux можно сделать как в Windows и заставить разделяемую библиотеку работать как отдельное приложение с функциями для вызова.
В Windows нельзя сделать как в Linux.

# Links

https://amir.rachum.com/shared-libraries/
https://seclists.org/fulldisclosure/2010/Oct/257
http://chenweixiang.github.io/2015/12/18/linux-series-05-libraries.html
https://community.broadcom.com/symantecenterprise/viewdocument/dynamic-linking-in-linux-and-window-1?CommunityKey=1ecf5f55-9545-44d6-b0f4-4e4a7f5f5e68&tab=librarydocuments