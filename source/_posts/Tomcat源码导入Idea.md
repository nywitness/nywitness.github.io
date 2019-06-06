---
title: Tomcat源码导入Idea
date: 2019-06-06 14:18:45
tags: tomcat
---



一直以来，使用tomcat的方式都是通过双击startup.bat文件，对内部的处理细节知之甚少。最近突然心血来潮，想要一探究竟，自然而然就想到看看tomcat源码。搜罗了一圈，广为流传的版本是使用新增pom.xml文件管理。由于tomcat官方采用的是ant管理的方案，也提供了编译tomcat的[说明](<https://tomcat.apache.org/tomcat-8.0-doc/building.html>)，这里记录一下尝鲜过程。

