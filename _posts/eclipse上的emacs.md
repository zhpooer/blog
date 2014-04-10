title: eclipse上的emacs
date: 2014-04-05 20:19:30
tags:
- eclipse
- emacs
---
清明闲来无事，又要开始折腾 emacs 和我的编程环境。  
作为一个 emacser ，小指头总会忍受各种折磨，下面贴出我在Linux下解决办法 ^_^

~~~~~
;; 修改 ~/.XmodMap 
;; 交换右ctrl键和Caps键
keysym Menu = Super_L
remove Lock = Caps_Lock
remove Control = Control_R
keysym Control_R = Caps_Lock
keysym Caps_Lock = Control_R
add Lock = Caps_Lock
add Control = Control_R
~~~~~

这个还是很有效果的，比以前使用左下角Ctrl轻松不少，不过用久了肩膀还是疼啊！！!
泪奔,投奔vim党。

以上是开胃菜，言归正传，身为一个Java程序猿，在IDE上编程总会有各种不适，从前研究过在
emacs上使用emacs-jde编写java；在emacs上通过eclime使用eclipse的功能；或者在eclipse上用vim插件；
均因为种种原因以失败告终，我可悲可叹的emacser人生，最后决定用 IdeaJ，他的按键绑定可以设置的很有emacs风格，
不过还是和eclipse的使用习惯有一定的不习惯，特别是项目管理。直到今天，我发现了 [把Eclipse打造成Emacs]("http://blog.csdn.net/lzw_java/article/details/17016279")

先按装好插件Emacs+，具体的我就不多说了，秀一下我的按键绑定吧：

1. Ctrl+C Ctrl+C : run
2. Shift+Ctrl+/ : redo
3. Ctrl+Q : quick fix
4. Shift+Alt+' :  Previous View
5. Alt+': next View
6. Alt+P : Previous Tab
7. Alt+N: Next Tab
8. Alt+O: Next Editor
