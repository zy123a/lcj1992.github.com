---
layout: post
title: 一些命令组合
categories: linux
tags: command linux
---

*   查看请求访问量前10的ip地址

        cat access.log  | cut -f1 -d " " | sort | uniq -c | sort -k 1 -n -r | head -10

*   统计200请求的占比

        export total_line=`wc -l access.2015-11-08.log | cut -f1 -d " "` && export not_found_line=`awk '$9=='200' {print $9}' access.2015-11-08.log | wc -l` && expr $not_found_line \* 100  / $total_line