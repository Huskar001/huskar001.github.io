---
layout: post
title:  "Better-scroll遇到的问题"
date:   2018-08-20 14:25:00 +0800
categories: web
tag: web
---
* content
{:toc}








# Better-scroll
[https://github.com/ustbhuangyi/better-scroll](https://github.com/ustbhuangyi/better-scroll "链接")


better-scroll 是一个移动端滚动的解决方案，它是基于 iscroll 的重写，反正是改进了不少吧。better-scroll 也很强大，不仅可以做普通的滚动列表，还可以做轮播图、picker 等等。
用法

	import BScroll from 'better-scroll'
	const wrapper = document.querySelector('.wrapper')
	const scroll = new BScroll(wrapper)



## 滚动原理

浏览器的滚动条大家都会遇到，当页面内容的高度超过视口高度的时候，会出现纵向滚动条；当页面内容的宽度超过视口宽度的时候，会出现横向滚动条。也就是当我们的视口展示不下内容的时候，会通过滚动条的方式让用户滚动屏幕看到剩余的内容。

当内容的大小改变的时候 需要重新计算betterscroll的大小。

## 问题

项目组同事使用betterscroll做了下拉刷新，因为是聊天界面，下拉刷新加载之前的聊天记录。后来发现一件事，对方发图过来。会无法加载滑动。我观察了代码。在接收到消息后做了个延时处理。
延时刷新。我想是不是来不及，时间不够长，然后加了时间，果然正常，但是考虑到延时时间无法固定，比如网络异常导致一张图加载个5秒很正常。想到vue的nexttick，嗯，换成试试。结果仍然失望。
怎么样选择刷新的时机。我想我遇到的问题，肯定有人遇到过，果然有人写了个图片的控件，等图片加载完再回调刷新。我这里不用那么复杂，直接调用img的onload回调刷新就行了。


## 进阶
观察了官方的文档。指导文档说明 vue中数据发生改变调用nexttick去刷新

      requestData().then((res) => {
        this.data = res.data
        this.$nextTick(() => {
          this.scroll = new Bscroll(this.$refs.wrapper, {})
        })
      })
    }

说明思路是对的。
不过如果图片确实加载比较慢，在聊天后又加载，就会出现闪动的现象，观察微信或者qq，发现都是控件先占位，然后慢慢加载。这个是个好主意，可以把图片的分辨率隐藏在图片名字中，比如 xxx/xxx/aaa_280x320.jpg 这样根据分辨率先占用空间再说。简单的思路，复杂的可以自己封装成控件。如微信或者qq那样做出加载的动画。

欢迎交流 feiqiyun@126.com

