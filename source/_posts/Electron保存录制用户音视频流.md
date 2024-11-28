title: Electron保存录制用户音视频流
author: Billy Yin
tags: []
categories: []
date: 2021-12-12 22:16:00
---
# 概念

### 获取音视频流
[getUserMedia 官方文档](https://developer.mozilla.org/zh-CN/docs/Web/API/MediaDevices/getUserMedia)
>它返回一个 [`Promise`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise) 对象，成功后会`resolve`回调一个 [`MediaStream`](https://developer.mozilla.org/zh-CN/docs/Web/API/MediaStream) 对象。若用户拒绝了使用权限，或者需要的媒体源不可用，`promise`会`reject`回调一个  `PermissionDeniedError` 或者 `NotFoundError` 。

[MediaStream](https://developer.mozilla.org/zh-CN/docs/Web/API/MediaStream)

### MediaRecorder
[官方文档](https://developer.mozilla.org/zh-CN/docs/Web/API/MediaRecorder)
[MediaRecorder/ondataavailable](https://developer.mozilla.org/zh-CN/docs/Web/API/MediaRecorder/ondataavailable)
ondataavailable 每次拿到的是Blob，如何给到下游步骤以持续处理并保存呢？

### Stream

### ReadableStream
[官方文档](https://developer.mozilla.org/zh-CN/docs/Web/API/ReadableStream)

pipe
[](https://nodejs.org/zh-cn/knowledge/advanced/streams/how-to-use-stream-pipe)

### ffmpeg
https://www.ffmpeg.org/
