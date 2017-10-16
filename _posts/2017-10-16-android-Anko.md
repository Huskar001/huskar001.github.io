---
layout: post
title:  "Kotlin&Anko"
date:   2017-10-16 10:35:00 +0800
categories: android
tag: android
---
* content
{:toc}








# 什么是Anko

Android开发者 经常会收集一些开发的代码，即代码收集帖，而Kitlin+Anko 可以让我们远离收集帖。
Anko 是 JetBrains 针对 Kotlin 推出的 Android 开发库，其目的是通过 Kotlin 让 Android 开发更加简单。（https://www.kotlinresources.com/library/anko/）

## 模块

Anko 主要有四个模块：

- Anko Commons: a lightweight library full of helpers for intents, dialogs, logging and so on;
- Anko Layouts: a fast and type-safe way to write dynamic Android layouts;
- Anko SQLite: a query DSL and parser collection for Android SQLite;
- Anko Coroutines: utilities based on the kotlinx.coroutines library.

View.setOnClickListener 方法可以说是广大 Android 开发者写得最多的方法之一了，如果你是用 Kotlin，那么代码看起来应该是类似这样的：

	button.setOnClickListener(object : View.OnClickListener{
	 override fun onClick(v: View) {
	 }
	})

通过使用 Anko 可以把代码缩减为：

	button.onClick { }

当我们希望跳转到新的 Activity 时，代码类似这样：

	val intent = Intent(this, MainActivity::class.java)
	intent.putExtra("id", 5)
	intent.putExtra("name", "John")
	startActivity(intent)

而通过 Anko：

	startActivity<mainactivity>("id" to 5, "name" to "John")

Anko 还封装了一些常用的功能，让我们无需定义 Intent：

	browse("https://makery.co")
	share("share", "subject")
	email("hello@makery.co", "Great app idea", "potato")

通过 Anko，Android 中的尺寸单位换算也变得无比简单：

	val dpAsPx = dip(10f)
	sp(15f)

处理多线程一直都不太容易，但在 Android 开发中我们经常需要面对。在 Anko 中的做法会相当的简洁：

	doAsync {
	 //IO task or other computation with high cpu load
	 uiThread {
	   toast("async computation finished")
	 }
	}

layout

	verticalLayout {
		val name = editText()
		button("Say Hello") {
			onClick { toast("Hello, ${name.text}!") }
		}
	}

数据库的使用 

	fun getUsers(db: ManagedSQLiteOpenHelper): List<User> = db.use {
		db.select("Users")
				.whereSimple("family_name = ?", "John")
				.doExec()
				.parseList(UserParser)
	}

使用anko

	dependencies {
		compile "org.jetbrains.anko:anko:$anko_version"
	}

如果只是使用部分功能

	dependencies {
		// Anko Commons
		compile "org.jetbrains.anko:anko-commons:$anko_version"

		// Anko Layouts
		compile "org.jetbrains.anko:anko-sdk25:$anko_version" // sdk15, sdk19, sdk21, sdk23 are also available
		compile "org.jetbrains.anko:anko-appcompat-v7:$anko_version"

		// Coroutine listeners for Anko Layouts
		compile "org.jetbrains.anko:anko-sdk25-coroutines:$anko_version"
		compile "org.jetbrains.anko:anko-appcompat-v7-coroutines:$anko_version"

		// Anko SQLite
		compile "org.jetbrains.anko:anko-sqlite:$anko_version"
	}

android support支持的控件

	dependencies {
		// Appcompat-v7 (only Anko Commons)
		compile "org.jetbrains.anko:anko-appcompat-v7-commons:$anko_version"

		// Appcompat-v7 (Anko Layouts)
		compile "org.jetbrains.anko:anko-appcompat-v7:$anko_version"
		compile "org.jetbrains.anko:anko-coroutines:$anko_version"

		// CardView-v7
		compile "org.jetbrains.anko:anko-cardview-v7:$anko_version"

		// Design
		compile "org.jetbrains.anko:anko-design:$anko_version"
		compile "org.jetbrains.anko:anko-design-coroutines:$anko_version"

		// GridLayout-v7
		compile "org.jetbrains.anko:anko-gridlayout-v7:$anko_version"

		// Percent
		compile "org.jetbrains.anko:anko-percent:$anko_version"

		// RecyclerView-v7
		compile "org.jetbrains.anko:anko-recyclerview-v7:$anko_version"
		compile "org.jetbrains.anko:anko-recyclerview-v7-coroutines:$anko_version"

		// Support-v4 (only Anko Commons)
		compile "org.jetbrains.anko:anko-support-v4-commons:$anko_version"

		// Support-v4 (Anko Layouts)
		compile "org.jetbrains.anko:anko-support-v4:$anko_version"
	}

最后 github地址 [https://github.com/Kotlin/anko](https://github.com/Kotlin/anko)