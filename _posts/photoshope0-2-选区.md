title: photoshope-2-选区
date: 2014-09-20 15:41:13
tags:
- photoshop
- 传智播客
---

# 选区 #

作用：针对选区进行编辑

可以在创建选区的工具属性栏上找到布尔运算, 针对已经存在的选区进行布尔运算
* 新选区
  * 在选区内拖拽, 移动选区MM
  * 在画面任意位置单击, 取消选择
* 加   : shift
* 减   : alt
* 相交 : shift + alt

在选区工具下, 要确保选择的是第一项新选区

更改图层顺序：`ctrl + [` 向上移动, `ctrl + [` 向下移动,
`ctrl + shift + [` 到最底层, `ctrl + shift + ]` 到最顶层

视图对齐, 确保有吸附功能

# 撤销 #

* 最后两步之间撤销, `ctrl + Z`
* 一直往回后退, `ctrl + alt + Z`
* 重做, `ctrl + shift + Z`

# 自由变换 #

切换快捷方式, `Ctrl + T`

可控制点
* 边角点
* 中心点

使用技巧
* 等比缩放 `Shift` + 拖拽边角点
* 控制中心点不动, 按住 `Alt`
* 按住`Alt` + 拖拽中心点, 两边对称同时缩放
* 旋转, 移动中心点, 在外边拖动
  * `shift` 约束15度
  * 如果图像太小, 拾取不到中心点时, 按住 `Alt` 识别中心点

# 复制方法 #

位移复制, 在V移动工具下, 按住`alt`, 鼠标左键拖拽

原位复制
1. 拖拽图层到新建图层
2. `ctrl + j`, 如果带有选区, 会复制选区内的像素到新的图层

使用情景: 制作编辑的副本, 快速恢复

恢复到画面初识打开的状态: `F12`

# 对齐和分布 #

在V移动工具下使用

对齐, 控制物件距离

分布, 控制物件距离
* 按顶分布
* 按左分布

缺点, 不同间距的不同变不能进行分布设置
<!-- 在AI中, 分布控制面板, 由设置分布间距的方法 -->

# 文字输入 #

切换文字输入工具 `T`, 可以填充前景色或背景色

结束文字输入, `ctrl + 回车`, `esc`取消本次输入

调节文字大小, `ctrl + T` 或直接更改字体大小

更改文字颜色, 针对文本层直接填充前背景色

修改文字内容, 选择要修改的文字内容, 设置属性

双击文本图层, 调出字符面板
* 标准文本配置 100, 100, 0, 0, 0
* 调节间距 `alt + 左右`, 调节行距 `alt + 上下`

段落选项: 修改避头尾法则和段落格式

windows 安装字体, `c://windows/fonts`

# 合并图层 #

`ctrl + E`
* 选择一个图层, 向下合并
* 选择多个图层, 合并所选图层

图层转选择: `CTRL` + 单击图层预览


# 练习制作 UI #

从底部到上逐层制作

所有跟屏幕相关的以像素为单位

1. 新建画布, 640*920, IPone4 手机屏幕大小, 填充颜色
2. 使用参考线上下分屏, 定位同心圆点(使用方形选框辅助)
3. 根据同心圆点, 画圆环, 调节图层不透明度(0-9, 连敲调节)
4. 插入小松鼠元素, 变换大小(自由变换 `Ctrl+T`)
5. 制作颜色块, 使用图层转选择, 自由变换
6. 制作光影效果, 使用单行选择框, 上灰下白, 表现凹下效果 (上白下灰, 表现凸起效果)


# 图片处理方式 #

三种处理方式
* 脚本 源图片
* 挖版
* 羽化版

## 使用羽化版 ##


1. M 选框工具
2. 羽化快捷键, `ctrl + alt + D` 或 `shift + F6`

变换选区 `alt + s t`, 针对选区进行大小变换
* 约束等比: shift(拖拽边角点)
* 约束中心点: alt

自由变换外框: `ctrl + t`, 针对像素进行大小变化
* 如果在选区的条件下, 自由变换像素, 背景色填充(可以制作透视图)

切换到画笔工具 B
* 画笔工具是以当前的前景色进行绘制
* 调节笔尖大小: `[` 和 `]`
* 调节笔尖硬度(模糊, 柔和): `shift + [` 和 `shift + ]`

# 制作气泡 #

1. 新建画布(任意大小)
2. 针对背景色填充黑色
3. 绘制合适大小正圆
4. 针对选区新建图层, 并填充白色
5. 针对选区进行 变换选区, 把选区适当缩小
6. 针对缩小后的选区执行 羽化, 数值适当控制
7. 用该选区, 针对之前填充得到的白色圆, 进行删除
8. 使用画笔工具添加高光部分


# 套索工具 #
切换到套索工具 `L`

不规则形状, 选框

1. 套索工具, 绘制曲线, 按住左键拖拽
2. 多边形套索工具, 绘制直线, 不断单击, 双击可以直接完成连线,
退格可以取消上一次选择, `ESC`取消所有绘制
3. 磁性套索工具

小技巧, 绘制过程中同时需要直线和曲线, 在绘制过程中, `ALT`键暂时切换



# 制作百货公司标牌 #

1. 新建画布 650mm*150mm, 或通过赋值模板来新建画布
2. 填充背景图层
3. 在素材中, 使用选区工具选中楼体, 拖拽楼体到新建画布
4. 楼体通过变形工具变形
5. 画箭头
  * 使用自定义形状工具, 来新建箭头标签, 通过变形调整箭头, `ctrl+回车`, 把贝塞尔曲线变成选区
  * 手工画箭头, 多边形套索工具 + `shift` 画水平线
6. 画出正圆形, 将箭头放到正圆上, 用图层转选择(`ctrl+选择图层预览`),
并删除箭头图层
7. 选择正圆形图层, 删除箭头选区
8. 输入文字`百货公司`, `Department store`

# 魔术棒工具 #
针对颜色边缘形状创建选区

切换到魔术棒 `W`
* 容差值, 与单击点色彩的差异不超过
* 连续, 要求颜色是区域是连续在一起的

使用情景:
1. 主体颜色相对单一
2. 背景颜色相对单一
3. **主体和背景之间的颜色反差较大, 由相对清晰的边缘**

反选: `ctrl + shift + I`

# 图层样式面板 #
`F7` 打开图层面板

投影
* 混合模式: 正片叠底
* 不透明度: 75度
* 角度, 常用30度,120度, 使用全局光(只有一个光源, 默认),
距离(投影和原物件距离), 扩展(投影的虚实程度, 羽化), 大小(投影大小)

复制图层效果: `alt`键拖动 或 选择图层再组合键 `alt + l y c` `alt + l y p`

清除图层样式: `alt + l + y + a`


TODO: 制作木乃伊
1. 用魔术棒工具选择木乃伊
2. 新建文字, 双击文字图层, 打开图层样式面板(可以设置UI样式)
3. 给文字增加


# Tip #

常有面板: 图层 + 通道 + 路径 + 历史记录

背景色可以不用新建图层, 可以直接填充到背景层上

切换吸管工具 `I`

photoshop缺点: 会*放大失真*, 但是画面细腻色彩丰富
