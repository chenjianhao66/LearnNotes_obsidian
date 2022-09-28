

## 嗨，建浩，欢迎回到 Obsidian ！😊

```dataviewjs
// 获取近些时间（5）天内的文章更新情况
let createsAndModifys = dv.pages("").file.sort(t => t.ctime || t.mtime,'desc')
let modifys = dv.pages("").file.sort(t => t.mtime,'desc')
let createds = dv.pages("").file.sort(t => t.ctime,'desc')

// 计算距离4天的间隔毫秒时间戳
let fourDuration = 1000*60*60*24*4
let fourDayBefore = new Date() - fourDuration
let count = 0
for (let obj of createds) {
	if (fourDayBefore > obj.ctime || fourDayBefore > obj.mtime) continue;
	count = count + 1
}
dv.paragraph("近些时间你更新/创建了 **"+count + "** 篇文章")

if (count <= 2) {
	dv.paragraph("学而不思则罔，思而不学则殆；你懈怠了啊，sunshine;不要忘记了你的初心。💪")
} else {
	dv.paragraph("已经完成目标啦，不要让学习成为一件公式化、让人讨厌和厌倦的事；")
	dv.paragraph("所以，放下 *Obsidian* 去做一些喜欢做的事情吧！！🚴‍♂️")
}

```

## Overview
```dataviewjs
let array = dv.pages("").file.sort(t => t.cday)
let ftMd = array[0]
let total = parseInt([new Date() - ftMd.ctime] / (60*60*24*1000))
let year = parseInt(total / 365)
let month = parseInt((total % 365) / 30)
let totalMonth = parseInt(year * 12 + month)
dv.paragraph("来 *Obsidian* 已经 **"+ year + "** 年 **"+ month +"** 个月啦，一共编写了 **"+ array.length +"** 篇文章，平均每个月编写 **"+ parseInt(array.length / totalMonth) +"** 篇文章呢👍")
```


```dataviewjs
let ftMd = dv.pages("").file.sort(t => t.cday)[0]
let duration = new Date() - ftMd.ctime
let date = new Date(new Date() - duration)

let dateString = date.getFullYear() + " 年 " + parseInt(date.getMonth() + 1) + " 月 "+ date.getDate()+" 日"
let total = parseInt(duration / (60*60*24*1000))
date = new Date()
let nowDateString = date.getFullYear() + " 年 " + parseInt(date.getMonth() + 1) + " 月 "+ date.getDate()+" 日"

let totalDays = total+" 天前的"+ dateString +"，在 *Obsidian* 创建了第一篇文章，而 "+ nowDateString +" 的今天，你我的故事还将在 *Obsidian* 上继续。"
dv.paragraph(totalDays)

```


```dataviewjs
let ftMd = dv.pages("").file.sort(t => t.cday)[0]
let duration = new Date() - ftMd.ctime
let createdTime = new Date() - duration
let createdYear = new Date(createdTime).getFullYear()
let Copyright = "Copyright @" + createdYear + "-" + new Date().getFullYear()
let author = " Author by 陈建浩"

dv.span(Copyright + author)
```