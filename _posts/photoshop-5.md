title: photoshop-5-画笔类工具
date: 2014-09-23 09:03:07
tags:
- 传智播客
- photoshop
---

# 画笔(B) #


笔刷, 载入笔刷

以直线进行绘制, `shift + click`

选项介绍
* 不透明度, 一次绘制, 有重合的地方不会叠加
* 流量, 有叠加, 配合喷枪使用
* 间距, 画笔由无数个圆组成, 调节圆圈的间距

调出画笔面板 `F5`
* 形状动态
  * 大小抖动
  * 最小直径
  * 角度抖动, 平面中转
    * 控制>方向, 根据拖拽角度变换
  * 圆度抖动, 沿z轴变化
* 散布, 发散开来
  * 数量
  * 数量抖动
* 颜色动态
  * 前背景抖动
  * 色相抖动 (饱和, 亮度, 纯度)
* 传递
  * 不透明度, 越实越前, 越虚越远, 增加空间感

定义笔刷(虚线绘制方法: 打开间距, 形状动态>角度抖动>控制>方向)
* 绘制基础笔刷造型
* 绘制选区
* 编辑 > 定义画笔预设

> 定义笔刷必须定义黑色, 否则会有透明度变化

# 历史记录画笔工具 (Y)#

被涂改的位置, 会恢复到预定义的(初识)状态,
使画面的某一部分不受满画面效果影响

# 橡皮擦工具 (E) #

擦除像素. 在背景层, 会用背景色覆盖,
如果将背景层装换为普通层, 被擦除的像素会变成透明


# 模糊工具组 #

模糊工具, 通过拖拽的位置, 控制模糊的范围(可以用 滤镜>高斯模糊, 替换使用)

锐化, 与模糊效果相反(可以用 滤镜>USM锐化替换使用)

涂抹, 通过涂抹更改像素位置, 通过强度控制效果


# 减淡工具组 (O) #

减淡工具, 变亮

加深工具, 变暗

海绵工具, 模拟加吸水效果, 修改饱和度







# 字符面板 #

字符属性标配: 0 0 0 100 100

* 行距调节, 选中需要调节行距的文字 `alt + 上下`
* 字间距调节, `alt + 左右`, 选择所有文字或调整光标左右两个文字

# 段落面板 #

推荐配置
* 首行缩进, 当前文字大小乘二
* 行首不能有标点符号, 通过设置避头避尾法则(严格或宽松)
* 段落文本右边对齐, 设置 最后一行左对齐

# 重复复制 #

一直重复第一次的复制效果
1. 绘制基础造型
2. 原位复制`ctrl + J`(也可以不做原位复制)
3. `ctrl + t`, 调出调节外框
4. 做位移或旋转
5. 敲回车确定自由调节外框的操作
6. `Ctrl + Shift + Alt + T`, 如果带选区,
不会自动新建图层 

<!-- qiuziti.com -->

# Tips #

选区内的像素剪切到新图层 `ctrl + shift + J`

栅格化: 将非普通层转化为普通层(字体栅格化前, 进行备份)
* 避免字体丢失
* 为了执行某些操作, 如滤镜

网页设计避免用高纯度 高饱和度的颜色, 如用红色(但是暗红色)

色相: 赤橙黄绿青蓝紫(有色系), 黑白灰(无色系)

饱和度: 颜色的纯度(颜色中混水)

明度: 光亮变化

屏幕分辨率 72, 印刷品分辨率 300