---
layout: post
title: 一些脚本
categories: language
tags: shell
---

在load超过2或者磁盘利用超过85%的情况下报警

    #!/bin/bash
    #chuangjian.li

    load=`top -n 1 | sed -n '1p' | awk '{print $11}'`
    load=${load%\,*}
    disk_usage=`df -h | sed -n '2p' | awk '{print $(NF - 1)}'`
    disk_usage=${disk_usage%\%*}
    overhead=`expr $load \> 2.00`
    if [ $overhead -eq 1 ]; then
     echo "system load is overhead"
    fi
    if [ $disk_usage -gt 85 ]; then
     echo "disk is nearly full,need more disk space"
    fi
    exit 0

通过脚本来进行日志文件与数据库的同步 (有问题)

    #!/bin/bash
    #chuangjian.li
    DB_FILE=/home/lcj/dbtest
    MYSQL=/usr/bin/mysql
    while read LINE
    do
     OLD_IFS="$IFS"
     IFS=" "
     filed_arr=($LINE)
     IFS="$OLD_IFS"
     STATEMENT="insert into user(name,age) values('${field_arr[0]}','${field_arr[1]}');"
     echo $STATEMENT
     $MYSQL test -u root -p -e "${STAEMENT}"
    done < $DB_FILE
    exit 0

