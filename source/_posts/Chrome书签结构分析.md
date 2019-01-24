---
title: Chrome书签结构分析
date: 2019-01-24 16:02:10
tags: 
    - Chrome
    - 书签
categories:
    - Chrome Extension
---
>最近几天整理Chrome书签，发现结构有点乱。琢磨着写个脚本，把Chrome书签转成Markdown文档，然后把每年阅读、收藏的文章以书签形式归档一下。

浏览器之间能自由的导入和导出书签，是因为统一了书签的结构。了解书签结构格式，不仅可以方便我们对导出的书签进行编辑、合并、替换等操作，还可以自己手工创建适合各种浏览器的书签。这里我导出Chrome浏览器的书签作为参考分析：

```html
<!DOCTYPE NETSCAPE-Bookmark-file-1>
<!-- This is an automatically generated file.
     It will be read and overwritten.
     DO NOT EDIT! -->
<META HTTP-EQUIV="Content-Type" CONTENT="text/html; charset=UTF-8">
<TITLE>Bookmarks</TITLE>
<H1>Bookmarks</H1>
<DL><p>
    <DT><H3 ADD_DATE="1483867303" LAST_MODIFIED="1540376281" PERSONAL_TOOLBAR_FOLDER="true">书签栏</H3>
    <DL><p>
        <DT><H3 ADD_DATE="1467684027" LAST_MODIFIED="1483867914">生活链接</H3>
        <DL><p>
            <DT><A HREF="https://kyfw.12306.cn/otn/" ADD_DATE="1386292463" ICON="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABAAAAAQCAYAAAAf8/9hAAAC7UlEQVQ4jaWTXWiTdxjFz/mnSSohXS1Ju9YEFqkzpHE1euUEN6032krnxgYb4td2VT8mg6Fe7GbFjSm4DxwMiwreSe1IRaQ32zQV23XOxqr1i7Yo1TSrmUnjm6TJ+/6fXQxq1nnnuXx4zu/iec4hXiARUQA4f0xSz9+tmG/MTEzs+PPo4c5UPF6vZ4ugzQZlt8O7PJJIj49/8UogcKocxDKzd6yvr+tG10/tUiqJJ7yMzR27xOF2c7SrS5JDAwQpwa3bfw20btpCcmoOICLqYX/s58tfHWqvdLuxav8BuLy1SF79A9o04W1uhs3hxPCRb2AkHuGN3Z/2+lvWv0tSEwBmZ2bau3dsj5ozGVn3ZSdLzwwZPvYDpVgQpUibssmS9z6gb+06ufL5Z1T2Cnn7+x83Oz2eXgUAY/39HdnpaVS/vpTuRT4MfvctZ/M5lExNy9KwLIv3z55BIfWEDWveQimb5WTsYsfcER8MDbksCDyhJpkc+p35nCGBN1cz/OFHopTi5KWLMn4uysTgoNSEwnzQd0FSt2655gBGOg1LC0BSaw3TslgVCKAmGCQBGMkpmlENbWmCgBZh0cg+f6NyOGBqQeL2bQnu20faHXIzGiWopNrv4/UTJ8QSoXdFRFIjI7QgAqUAAAoA6sNhmFrjYTxO4+lTrPz4ExZyOSRHRzl95y6e/Z1iYGMbql4LcOK3XyACunz+54BgS0uPstthlky50NmJV5uaxBMKwdRaLMuC01srja2tuPz1ITHSaZgC8a9Ze7U8Bwtix4+PDJw82aiUQlVdHRY2NCC0cQOqvLW40d0NI5mEkXgEBaCxZf3U6gMHIySnKgCAZL5YLL6fefy4734sVhduaxO708lCZkYKmSzrIytEWyXeO39eqn2+3Mo9e/+bxLI4+6/19PQOnj4dySQSsCkFm1JQBFwLa7B88zvDka3bdjocjvj/ulBeKDOfX/XX2Jhr1jD+fVVlJTyLFxsL3O6BFzXypfQP9MtgN1/s0W0AAAAASUVORK5CYII=">12306</A>
        </DL><p>
        <DT><H3 ADD_DATE="1467710882" LAST_MODIFIED="1516610024">git</H3>
        <DL><p>
            <DT><A HREF="http://www.bootcss.com/p/git-guide/" ADD_DATE="1467709151">git 使用简易指南</A>
            <DT><A HREF="http://danielkummer.github.io/git-flow-cheatsheet/index.zh_CN.html" ADD_DATE="1467709171">git-flow 备忘清单</A>
            <DT><A HREF="http://marklodato.github.io/visual-git-guide/index-zh-cn.html" ADD_DATE="1467709175">图解Git</A>
            <DT><A HREF="http://www.jianshu.com/p/9b6ceb3b7d35" ADD_DATE="1476841970" ICON="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABAAAAAQCAYAAAAf8/9hAAACqklEQVQ4jT2TzW5bVRSFv73vOffa7Y2N7dbFLU3pX8oASoXKnwRCYsQMCQlEHwDBO/QRKmbMmCAmTBAPAKIIIQRMKlFEhdSGJCCn0KAkjuPE1/eesxmcwHRJa++z1/qOjG/dNATMBBGw0IAoogoiEGPSNEsaAhhmhoigiGAGgmHRyDp9tFVgIRAXFSaKG55BixzDsLrCYkREMDM0uQWzAL6k/877LD13FVyOf3yElCcY3HiP9sXzhNkB/uwFsnZaIALOABEhVDXlC6+Q9zuEE0/y2BuX8f0lmlmDOsiXn6LbO8exq9dZ/PIdO9/ehqxARZQ4n+HPPU35zApbn34EnTOwu8bej3dwrcg/n31CfukaTDfZ+vhDpnd/RlwOZqiFGu2O6Lz0IntffUF2egVZ7FBtT8mXR0y//5pmMqXaWMWyAj8YECa7cBSnYob4jP0fvuTgt3uEWYXv9ai3HmHVHDdcpnz2GvV4leLKywzeehdXtlI7IjhxnrjzF4EWvbc/oFm/y+SnO7QurlD/8QBTRz56gur3X6E1RBeeZncf8hZihpoZknlQwfVHaAHzB2uU11/FDXroUp/jz79G+8J5LATMwCzViAgqgAnQBOJsnzDbR7zDmhrRDBElHu6xePgnlrD5nwEgKYktQATRhIY1FYvNDRbjDawBbR9PsCVfegHggIRl5hDvsRCRwqFll3z5MrHJyNrHsLqBpgEKDEOOJjnMEANU0TxHVFGfUW+uU63dx8xRnT6Ldk/iTg0RPUAs3Q+GSxinSsAQ56k37rH99ypxfggWmXxzSOf1NymvXGLv9ueYZskfQca3bpphIA4/HBG2HxIXdYpH06ZYHaLdU+Qn+1Tr90Ey5AgllwJRsEA9XoPMI5rBf8HGiBZtbLbNfPIIyYtkTl+YfwFB8jD/X7vjBgAAAABJRU5ErkJggg==">利用Github和CocoaPods搭建私人Pod开发环境 - 简书</A>
            <DT><A HREF="http://blog.csdn.net/victor_barnett/article/details/51211282" ADD_DATE="1516609567" ICON="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABAAAAAQCAIAAACQkWg2AAABWklEQVQ4jbVSwUoCURQ9V2fGmUwxagLNwAiDQaHc+AFu/Qt3FtRnuLf+oMCN4GL20c6FEhiU4UKDIFHLJipwGvW2GBUHqXDRhQvvXs55953zLlVBWCZcS6EBCM6S7SRADIWk8DYpCg+tUb9vtlo8MBcJUBKJzWzWn06LweB8f/Rm1NbW5wnsUpRwPr+RyYAWVbH11HY8iQRhV9d9qZRdGqVSv1AY3NXHH+8kK+JWyO3zOwjq4dEM/Xhy3D09ozn3zEZjdibbVu2mpsTjAMxm8zYaZWb6we6JrbKmzS5j5l++ZkIYGwYAEGRNI0HgPwmvxSIAAnkikZ2LcykcZjDADADskj0MdmhwBwJ7V5cr+wcAmBnMX60H67lHgiCqqtXp3CeTsFVVQVVQBbj2rrZzOav/Ml4IQ9crUyRNl286UZK8yaQciwmqSm73eGAOe93PcnlQr9sT6N+39RsgVo7oiKSelAAAAABJRU5ErkJggg==">SourceTree里GitFlow的使用 - CSDN博客</A>
        </DL><p>
        <DT><H3 ADD_DATE="1467683920" LAST_MODIFIED="1541571524">iOS</H3>
        <DL><p>
            <DT><H3 ADD_DATE="1467971905" LAST_MODIFIED="1542857718">大牛blog</H3>
            <DL><p>
                <DT><A HREF="http://blog.ibireme.com/" ADD_DATE="1447205476" ICON="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABAAAAAQCAYAAAAf8/9hAAADWElEQVQ4jW2TT2gcdRzF3+83Mzv7Z7Zjqm3tujTTcTDYhdH4Q9EWS2i6K0GE1rIJlLZIPSgFPYgHEcTePHmwtx4UQYo0lhwaMZgYzSElXcioDQ4eMq6rTpbNpnGz62b/zO5vfh6aQATf5X0P7/C+8HkAQPBAyo6TnZsAIOx0XgcAy7LU/8nJFICwbTthGJAASJZlJRljACCy45c+OnhYXz5+auxxz/O6QyeGkntzAEICQDEMQyqVSoFpmslYLNZ2XTc4PX7pQyrJVyHEr5TSeLNaPbn43XQFgDBNM14sFv/ZrUEAENM0kwMDAy3HcXqj5y6+HUtqn7Qbjfz81I1b2YnXfpYo0ZqN7rOry/PB+vp6J51OR3Td5wQAtSxL8zy9DTi9U2fP5+P79MlOs/E6lOhPpNMWzfXNP/cdfeyuCHljtvLH82nPU3Rdl1zXbVIAkud5TcDpj5w5/7SaiH8ZtFvvd8L+oqpIP0Imo0tL3/69tVF9mSrKkeyhI5/5vt92XbeVASIUQC+dTqsAk9V49FPeC2bW/qpcTya05V4QXJ+7deNjxtgjhbmvVztb9VdlJXJx9NyFMwCIOzISUsMwor7vB9n8sePg4VO1Wv1KavDwVNgPf5m9+fmbpmnqjuPUGGP6wvRXi2G//4WkyO8B4FhYCGnJMPoAOAhegiwtJ6KxQwTkRLsdvJEBIsVicTuVSql1uS4AkF4QzFGJDmYyL+wHQGlmY4MCgCCSjDBskyh9mFDS7FbWai7Qs21bBQCvMLwNgMqKWhOhiGhaXAHQp/V6XQIACL4K4Ilqc+ue4FxLHh3MAxDdbpeWy+UOY8UBABwhfxJArVCYrzIGhfq+37YsS201N2cFIQcOqg/leEhe4UH3N+TzkqZpHcbYfsdxNg3DiBKJvsMFbgMQAHvAfSqVUsrlcis7fuEDORK72mnUT/5we/LODvMSAG7bduLRzDOTlEjP1crl4cLCjA+A7pJIGWOa4zj13MTla3JEfqvP+c2Qd7+hQq4J9I/JinoFVIo07t8/e2dm6q5t2wnOV3p7SPS2GWOq4zit3MTlMSqJdyEwJAAFIJsAma6Ufr92b+n7Ndu2EwCwsrKyDQAyAAqAWBbUPfOGncslhl8cO4D/Ss5kENl97198lGjU3MC7AAAAAABJRU5ErkJggg==">Garan no dou | 一只魔法师的工坊</A>
            </DL></p>
        </DL></p>
        <DT><A HREF="http://blog.csdn.net/li6185377/article/details/51774911" ADD_DATE="1474211165" ICON="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABAAAAAQCAIAAACQkWg2AAABWklEQVQ4jbVSwUoCURQ9V2fGmUwxagLNwAiDQaHc+AFu/Qt3FtRnuLf+oMCN4GL20c6FEhiU4UKDIFHLJipwGvW2GBUHqXDRhQvvXs55953zLlVBWCZcS6EBCM6S7SRADIWk8DYpCg+tUb9vtlo8MBcJUBKJzWzWn06LweB8f/Rm1NbW5wnsUpRwPr+RyYAWVbH11HY8iQRhV9d9qZRdGqVSv1AY3NXHH+8kK+JWyO3zOwjq4dEM/Xhy3D09ozn3zEZjdibbVu2mpsTjAMxm8zYaZWb6we6JrbKmzS5j5l++ZkIYGwYAEGRNI0HgPwmvxSIAAnkikZ2LcykcZjDADADskj0MdmhwBwJ7V5cr+wcAmBnMX60H67lHgiCqqtXp3CeTsFVVQVVQBbj2rrZzOav/Ml4IQ9crUyRNl286UZK8yaQciwmqSm73eGAOe93PcnlQr9sT6N+39RsgVo7oiKSelAAAAABJRU5ErkJggg==">《优雅的插入开屏广告》-- 不改动任何一行代码 - li6185377的专栏 - 博客频道 - CSDN.NET</A>
</DL><p>
```

从 Chrome 的书签我们可以看出，引用书签的 html 标签主要为 *DL* 和 *DT* ，其中 *DL* 引入开始，*DT* 标注书签或者文件夹。
其中文件夹有 *ADD_DATE* 、*LAST_MODIFIED* 、*FolderName*（ *DT* 标签中的 value 值），并且有 *H3* 标签样式标注是文件夹，文件夹下可以有子文件夹，也可以有书签块。用 Objective-C 来描述，如下：

```objective-c
@interface BookmarkFolderModel : NSObject

@property (nonatomic, copy) NSString *name;
@property (nonatomic, assign) NSTimeInterval addDate;
@property (nonatomic, assign) NSTimeInterval lastModified;

@end
```

书签有 *ADD_DATE* 、*LAST_MODIFIED* 、*HrefName*（*DT* 标签中的 value 值），并且有 *A* 标签标注是书签，书签下不能有子文件夹、子书签块，是 Chrome 书签文件的基本元数据，可以挂载在文件夹下，也可以直接挂载在跟块下。用 Objective-C 来描述，如下：
```objective-c
@interface BookmarkModel : NSObject

@property (nonatomic, copy) NSString *href;
@property (nonatomic, assign) NSTimeInterval addDate;
@property (nonatomic, copy) NSString *icon;
@property (nonatomic, copy) NSString *name;

@end
```

书签整理脚本另起文章，敬请期待...
