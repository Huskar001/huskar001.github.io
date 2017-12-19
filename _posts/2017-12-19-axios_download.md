---
layout: post
title:  "vue之axios流文件下载和播放"
date:   2017-12-19 10:31:00 +0800
categories: web
tag: web
---
* content
{:toc}








# 下载问题

最近我负责前端，本来文件下载传输一个url 定义个a标签click不就完事了。哪里知道服务器文件也是从其他地方下载过来的，希望传输后删除 不占用空间。我们的前端是用vue，http请求用的axios。好吧，搜了一圈 ，说不支持post下载。找了好多资料，明明的response里面能看到wav的数据就是播放不出来，后来找了个函数，转换字符串转byte的，居然能播放 但是声音明显不对，杂音很大，应该是丢失了一些数据。

## 解决

1 axios请求 post 参数是
(method) AxiosInstance.post(url: string, data?: any, config?: AxiosRequestConfig): AxiosPromise
第一个url 第二个请求数据 第三个配置 这边需要加一个配置{responseType: 'arraybuffer'}
具体代码

	export const batchDownRecordFiles = params => {
	  config.requestInfo = params;
	  return axios.post(`${baseVCC}/v1/batchDownRecordFiles`, config, {responseType: 'arraybuffer'})
	}

返回结果处理

	let transferPrams = {
		contactId: '1223',
		filePath: '1/0/20171213/106/1027463.V3'
	}

	API.downRecordFile(transferPrams)
	.then(res => {
		console.log('downRecordFile');
		if (res.status === 200) {
			var filename = decodeURI(res.headers['content-disposition']).match(/fileName=(\S*)/)[1];
			var blob = new Blob([res.data], {type: 'audio/wav'});// audio/wav //application/octet-stream
			var link = document.createElement('a');
			link.href = window.URL.createObjectURL(blob);
			link.download = filename;
			link.click();
		}
	})

这样就可以下载流数据的录音文件了，我看网上还有下载excel的，估计都是小文件，大文件没试过。
好了 又遇到新问题，希望能直接点击播放。本来html5播放是这样写的

	<audio v-if="audiosrc" autoplay="autoplay" loop="loop" :src="audiosrc" controls="controls"></audio>
	
传递一个src=‘xxxx.wav’
可是这边没有xxx.wav啊然后又去找能不能直接播放blob呢 答案是可以的把blob处理下
具体代码

	this.audiosrc = window.URL.createObjectURL(blob);

就是这么简单。
遇到问题多读文档 原先被误导把axios参数弄错了 以为把配置写在data里，搞了好久。
后台代码就不贴了，无非是读取流的，理论上讲，还可以播放视频，但是还是尽量小文件，大型文件的流数据播放 还是要专门的框架的。
欢迎交流 feiqiyun@126.com

