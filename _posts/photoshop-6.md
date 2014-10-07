title: photoshop-6 钢笔工具
date: 2014-09-25 09:50:57
tags:
- photoshop
- 传智播客
---

# 钢笔工具(P) #

绘制选区方法
* 规则选区 `M`
* 不规则 `L`
* 颜色 `W`
* 实现复杂图像的抠图, 钢笔工具 `P`

贝塞尔曲线 转化为 选区: `Ctrl + 回车`

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

暂时切换到转换点工具, `Alt`
* 移动到转换点, 可调节转换点

用尽量少的锚点, 绘制尽量准确的曲线

`Esc`, 第一次隐藏锚点, 第二次隐藏路径

三种点的类型:(相互转换按住`alt`)
1. 尖角点, 没有杠杆
2. 平滑点, 两条杠杆保持在同一条直线上
3. 贝塞尔点, 两条杠杆, 不在同一条直线上

选项
* 形状, 有路径, 内容会直接填充颜色, 伪矢量
  * 填充
    1. 实色填充
    2. 渐变填充
    3. 图案填充
  * 描边(可以制作虚线)
* 路径, 由路径, 但是没有填充
* 像素, 只针对 `U` 工具组, 没有路径, 但是只有内部填充

如果选择路径布尔运算中相减时, 直接出现当前造型选择相反范围:
按一次`Esc`, 只显示路径, 不显示锚点, 再继续操作

绘制路径, 转化为选取时, 直接出现反选效果:
路径布尔运算当前选择的是`相减`, 可以选择其他选项如 `合并`

合并形状路径

非等比缩放圆角矩形, 使用 `A` 小白, 拖动圆角点

工作路径只能记录下最新绘制的路径.
可以拖拽工作路径到下方新建按钮, 转化为普通路径.

钢笔模拟压力, 必须在画笔面板中选择形状动态

# 消失点工具 #

消失点: 滤镜菜单>消失点 `Ctrl + Alt + V`
1. 基于画面透视关系绘制出正确的透视关系网络
2. 基于正确的透视网络, 完成画面修复

# Tips #

隐藏或显示选区 `Ctrl + H`

> 字体规范: 中文接英文或数字时, 两者之间应该有距离(为半角状态下一次空格)
副标题中破折号和后方的文字同上
