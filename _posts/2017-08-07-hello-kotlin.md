---
layout: post
title:  "Hello Kotlin-Getting started with Android and Kotlin"
date:   2017-08-07 10:56:02 +0800
categories: android
tag: android
---
* content
{:toc}

Kotlin 是一个基于 JVM 的新的编程语言，由 JetBrains 开发。
Kotlin可以编译成Java字节码，也可以编译成JavaScript，方便在没有JVM的设备上运行。
JetBrains，作为目前广受欢迎的Java IDE IntelliJ 的提供商，在 Apache 许可下已经开源其Kotlin 编程语言。
Kotlin已正式成为Android官方支持开发语言。

<img src="{{ '/styles/images/kotlin.png' | prepend: site.baseurl }}" width="546" />

管他什么优点缺点，既然能成为google支持的语言，想必不会差的，就如AndroidStudio代替eclipse一样，先来个Helloworld吧~！

安装Kotlin插件
--------------

AndroidStudio 从version 3.0开始已经把kotlin插件打包了，如果是低于这个版本，则需要手动安装 安装方法
File | Settings | Plugins | Install JetBrains plugin… and then search for and install Kotlin.
安装完AndroidStudio需要重启。

再提示配置Kotlin的时候 选择自动配置就好了。

转换代码
----------

选中需要转换的Java文件, 如MainActivity.java,
使用Command+Shift+A, 启动Action, 输入Convert, 找到命令, 即可转换, 如

<img src="{{ '/styles/images/convertkotlin.png' | prepend: site.baseurl }}" width="546" />

或
选择Code -> Convert Java File to Kotlin File, 也可以使用快捷键.
把.kt的文件剪切到kotlin文件夹下, 即可使用.
推荐Kotlin文件和Java文件分开存放, 不过放在一起也可以使用.

转换后的代码

	class MainActivity : AppCompatActivity() {
		override fun onCreate(savedInstanceState: Bundle?) {
			super.onCreate(savedInstanceState)
			setContentView(R.layout.activity_main)
		}
	}
	
随后我按照网上的代码，写一个view然后加一个id然后直接引用id发现报错
查了下才知道 需要加上

	apply plugin: 'kotlin-android-extensions'
	
然后
         
	android:id="@+id/my_textview"
	
代码中 可以直接

	my_textview.text = "hello kotlin"
	
函数定义

	override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        my_textview.text = "hello kotlin"
        my_textview.setOnClickListener {
            Toast.makeText(this,"hello kotlin",Toast.LENGTH_LONG).show()
            b();
        }
    }
    fun MainActivity.b() {
        this.my_textview.setText("hello fun b")
    }
	


kotlin参考链接 [http://kotlinlang.org/docs/tutorials/kotlin-android.html](http://kotlinlang.org/docs/tutorials/kotlin-android.html "kotlin")

进阶学习还有一些好用的框架参考[ http://kotlinlang.org/docs/tutorials/android-frameworks.html]( http://kotlinlang.org/docs/tutorials/android-frameworks.html "kotlin")

Dagger 依赖注入 现在叫Dagger2了

Butterknife view的注入 省去遍地都用findviewbyid了 能节省很多代码量

Data Binding 这个跟c# wpf中用法差不多 当年用c#就觉得这个很方便 把数据和view绑定在一起 随动

Auto-parcel https://github.com/frankiesardo/auto-parcel 通过注解及工具类自动完成实体类 Parcelable 及值传递

DBFlow  代替sqlite的 个人感觉不如GreenDao好 毕竟改了数据结构

接下来 会试用一下部分框架。