---
layout: post
title:  "contenteditable的div一些笔记"
date:   2018-02-23 10:31:00 +0800
categories: web
tag: web
---
* content
{:toc}








# 需求

需要做一个聊天的场景，反正就是仿照qq或者微信，还要是在线的，好吧，那输入框自然而然的想到了input，搞了半天发现只能输入文字，没法输入其他的比如图片或者表情。想到了web版本的微信，web版本的qq还有各种编辑器。这时候再上什么高端的编辑器好像不太合适，好吧，我能遇到的问题，前人肯定解决过。搜索了一圈，原来直接用div 然后设置contenteditable就行，里面填充的就是标准的html标签。
最终效果如下图

![/styles/images/20180223144411.png]({{ '/styles/images/20180223144411.jpg' | prepend: site.baseurl  }})

## 解决

1 表情

表情需要插入如 img 类似html标签。在github上找了个开源的可以选择表情的，转换成标准的img标签显示，然后在发送过程中 再把标签用正则换成[[smile]]或者::smile::
最后在展示的根据解析 再还原成img标签，这样就完成了展示，当然如果发送端和接受端引用同一段表情地址也行，典型的直接接入腾讯的表情也行，这里我用了标准的emoji表情。
人人为我，我为人人 表情库的地址奉上。
https://github.com/rishiqing/vue-emoji
就是这么简单。把正则贴上，没系统学过正则 瞎捉摸的+瞎找的

      replaceContent (content) {
        let _this = this
        let htm = content.replace(/<img\s*[^>]*(.*?)>/g, function ($1, $2) {
          // console.log($2)
          let el = document.createElement('div')
          el.innerHTML = $1;
          let imgVal = el.getElementsByTagName('img')[0].title;
          // console.log('imgVal' + imgVal)
          if (imgVal === '') {
            return $1
          }
          return $1.replace($1, '[:' + imgVal + ':]')
        });
        let text = htm.replace(/\[:(.+?):\]/g, function (word, ww) {
          let newWord = word.replace(/\[:|:\]/g, '');
          return word.replace(word, _this.$refs.emoji.getImgStrByName(newWord).outerHTML)
        })
        console.log(text)
        return text;
      },


2 图片

选择本地图片发送这样的用例就不说了，选择图片，上传得到地址，然后组装img标签 发送。
这里我想实现类似微信 直接粘贴 所见即所得 直接发送。
研究了下 vue中直接捕获 @paste="onPaste"事件
然后获取类型 具体直接看代码

      onPaste (e) {
        var items = e.clipboardData.items;
        for (var i = 0; i < items.length; ++i) {
          if (items[i].kind === 'file' &&
                items[i].type.indexOf('image/') !== -1) {
            var blob = items[i].getAsFile();
            window.URL = window.URL || window.webkitURL;
            var blobUrl = window.URL.createObjectURL(blob);

            var img = document.createElement('img');
            img.src = blobUrl;
            var logBox = document.getElementById('sendMsgbox');
            logBox.appendChild(img);
          }
        }
      }

调研资料得知早期的qq空间就是这么实现的，。

欢迎交流 feiqiyun@126.com

