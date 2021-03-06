title: photoshop钢笔工具
date: 2014-06-18 14:17:51
tags:
- photoshop
---

# 画出虚线 #

* 矩形选框工具, 填充黑色, 不取消选框
* 编辑, 预设画笔
* 画笔控制栏, 调整间距, 形状动态, 控制选方向

# 钢笔工具(P) #
实现复杂图像的抠图

贝塞尔曲线 转化为 选区: `ctrl + 回车`

贝塞尔曲线组成:
1. 锚点
2. 连接锚点之间的曲线, 路径

删除锚点
* 方式一: 选择点, 安装 `del`, 删除之后要在最新锚点上单机(续上)
* 方式二: 历史回退

移动锚点
* 使用路径选择工具(A)(黑), 针对整体进行调节
* 使用路径选择工具(A)(白), 针对细节, 路径进行调节

杠杆, 调节路径的造型, 拖拽描点
* 杠杆的长度, 弧度的大小
* 杠杆的方向, 弧度的方向

alt : 暂时切换到转换点工具
* 移动到转换点, 可调节转换点

用尽量少的锚点, 绘制尽量准确的曲线

吸管工具(I)

三种点的类型:(相互转换按住`alt`)
1. 尖角点, 没有杠杆
2. 平滑点, 两条杠杆保持在同一条直线上
3. 贝塞尔点, 两条杠杆, 不在同一条直线上



# 四把锁头 #
1. 锁定透明像素, 像素分为: 普通像素和透明像素. 只能对普通像素进行操作
2. 锁定像素, 锁定普通像素和透明像素, 不能用画笔工具进行任何操作
3. 锁定位置, 不能使用 v 移动工具进行拖动, 但是在有选区情况下是可以拖动的
4. 锁定全部

`alt + shift + 退格` 填充前景色到有普通像素的区域

`ctrl + shift + 退格` 填充背景色到有普通像素的区域

# 图层蒙版 #
控制画面的显示和隐藏, 从而实现图像合成等操作

1. 选择图层
2. 点击`图像蒙版`按钮
3. 选择蒙版的白板
4. 使用画笔操作, 黑色代表隐藏, 白色代表显示

`shift+click`暂时关闭蒙版

`alt+click`在画面内显示蒙版上的黑白关系

应用蒙版

可以用蒙版的方式进行, 无损抠图, 保留了图片的所有像素

可以使用双蒙版, 使用钢笔工具进行操作


`ctrl + g` 选中的图层形成组

可以在组上添加蒙版进行控制

图像的分辨率(像素每英寸ppi):
* 72, 屏幕显示的分辨率
* 300, 印刷品的分辨率

图层样式斜面和浮雕
1. 外斜面, 自身不动, 外面动
2. 内斜面, 自身动, 外面不动
3. 斜面浮雕, 外斜面 + 内斜面
4. 枕状浮雕, 向下凹陷
5. 描边浮雕, 结合图层像是下的描边效果

复制图层样式, `alt + l + y + c` , `alt + l + y + p`

清除图层样式, `alt + l + y + a` 

# 通道 #

RGB颜色模式下: 4个通道层
* RGB,专色通道: 用来承载专色通道上的颜色信息, 用黑白显示相应的颜色信息
  * 黑色不显示, 白色显示
* 综合通道: 显示最终画面信息
