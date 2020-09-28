---
title: 整点休息提醒脚本
categories: 未分类
date: 2020-09-27 21:59:55
tags: [code]
---

最近腰有点不舒服，体检报告出来说是腰部的脊椎有点外凸，看来是人到中年身体开始走下坡路了，平时还是要多注意休息和锻炼。腰椎不好，久坐会是个问题，所以就简单弄了个脚本，准点就会弹窗提醒，看到提醒就站起来稍微动一动吧。
<!-- more -->

用法是：拷贝到一个bat文件中，随便找个文件夹放好，双击运行下，之后整点就会有提示。  
如果提示的文字是乱码，大概率是编码的问题，把文件编码换成 ANSI 后再把代码粘贴进去就好了。

    @echo off


    set ReminderName=HourRestReminder

    if "%1" equ "" (
        :: install
        echo.                           -----------------------------------------------
        echo.                                   author : Zen XA
        echo.                                   mail   : wszxayx@gmail.com
        echo.                           -----------------------------------------------
        schtasks /create /tn %ReminderName% /tr "'%0' 1" /sc DAILY /st 09:00 /du 12:00 /ri 60 /f
        schtasks /run /tn %ReminderName%
        pause

    ) else ( 
        :: reminder
        for /l %%i in (9,-1,0) do (
        cls
        color %%i
        echo. 
        echo. 
        echo. 
        echo. 
        echo. 
        echo. 
        echo. 
        echo.                           -----------------------------------------------
        echo.                                         休息一下，有益身心。 %%i
        echo.                                         休息一下，有益身心。 %%i
        echo.                                         休息一下，有益身心。 %%i
        echo.                                         休息一下，有益身心。 %%i
        echo.                                         休息一下，有益身心。 %%i
        echo.                                         休息一下，有益身心。 %%i
        ping 127.1 -n 2 >nul
        )
        exit

    )

    exit