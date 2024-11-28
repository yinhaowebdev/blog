title: nodejs 批量下载文件
author: Billy Yin
tags: []
categories: []
date: 2021-12-12 22:24:00
---

要点：
1 分批下载
2 windows文件名校验
3 EventEmitter 

```
function downloadList (list){
  let curSonglist = []
    curSonglist =  list.splice(0, Math.min(10, list.length))
    if(curSonglist.length){
      let downloadManager = new Donload(curSonglist)
      downloadManager.start()
      downloadManager.on('finish',()=>{
        console.log(' on finish')
        downloadList(list)
      })
    }
}


class Donload extends EventEmitter {
  constructor(list){
    super()
    this.list = list
    this.length = list.length
    this.count = 0
  }

  start(){
      this.list.forEach(e=>{
        this.count++
        this.donwloadSong(e)
      })
  }

  donwloadSong(song){
    let artists = []
    if(song.singers && song.singers.length){
      artists = song.singers.map(e=>{
        return e.name
      })
    }
   
    this.download(song.url, `${artists.join('&')}-${song.title}`)
  }

  download(url,name){
    name=name.replace(/[\\\/:*?"<>|]+/g, "")
    const stream = fs.createWriteStream(`./songs/${name}.mp3`)
        request(url).pipe(stream)
        stream.on('close',()=>{
          this.handleEnd()
        })
  }

  handleEnd(){
    this.count--
    if(!this.count){
      this.emit('finish')
    }
  }
}
```