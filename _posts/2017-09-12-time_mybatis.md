---
layout: post
title:  "mybatis中Date和DateTime字段"
date:   2017-09-12 10:30:02 +0800
categories: web
tag: web
---
* content
{:toc}

最近做web开发使用spring cloud 使用MyBatis做的数据持久层。发现mysql存入的时间是带时分秒的，但是读出来再存进去，就丢失了时分秒。
后来发现存的时候使用 java.util.date 能顺利的存进去时分秒。读出来的时候必须使用 TIMESTAMP 而不是DATE

	<result column="UPDATEDATE" property="updateDate" jdbcType="TIMESTAMP" />

困扰的小问题，顺利解决。


