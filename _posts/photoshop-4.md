title: photoshop-4 图像修复
date: 2014-09-22 09:17:50
tags:
- 传智播客
- photoshop
---

# 仿制图章(S) #

恢复到画面初始打开状态 `F12`
  
画笔类工具, 包含调节大小、硬度
* 笔尖大小, 变小 `[`, 变大 `]`
* 笔尖硬度, 变软 `shift + [`, 变硬 `shift + ]`


使用流程
1. 切换到仿制图章工具
2. 取样, `Alt + click`, 松开 `alt`
3. 调整笔尖大小, 硬度
4. 修复, 点击或拖拽

> 使用技巧: 就近取样, 多取样

选项
* 对齐, 在连续点击时, 保证当前取样的连续性
* 不透明度, 工具不透明度, 减弱复制粘贴效果, 更柔和

## 延续纹理修复 ##

方法一: 使用仿制图章工具(S)
* 分两次取样, 不松开 `alt`
* 可以用羽化来消除重复纹理

# 修复画笔工具组 (J) #

* 修复画笔工具:
与仿制图章工具相似, 但是会和下方的图像之间实现融合效果.
* 污点修复画笔具: 是傻瓜版的修复画笔工具
* 修补工具, 操作类似于套索工具(也可以进行布尔运算)
   * 选项`源`, 绘制出来的范围被覆盖(从外边拿回来)
   * 选项`目标`, 将选择的像素复制出去
   * 选项`透明`, 配合选项`目标`使用
* 红眼修复工具

<!-- 7 10 12 15 17 20 22-25 -->

# 羽化(Shift + F6) #

针对选区做出边缘虚化的效果

在选区下, 使用(`Shift + F6`)

# 渐变(G) #

通过渐变编辑器(渐变编辑条), 设置渐变颜色
  * 色标(矩形小块), 下面的每个色标代表一种颜色, 上面的色标代表透明度
    * 复制, 选中要复制的色标, 点击渐变编辑条
    * 更改色标, 双击
    * 删除色标, 拖拽
    * 添加色标, 单击渐变编辑条
  * 颜色中心点(菱形图标)

渐变的类型(5种), 在工具属性栏设置
* 线性渐变, 以一条线的方式排列颜色, 按 `shift` 约束平滑,
* 中心向外渐变(其他4种, 径向 + 角度  + 对称 + 菱形)

选项
* 反向, 落点颜色 和 结束点颜色 交换
* 仿色, 色彩更加逼真
* 透明区域, 实现透明效果


# 彩虹光盘制作 #

* 新建 200*200 画布
* 定位中心点
* 添加灰色圆环和彩虹圆环
* 双击图层代开图层样式窗口, 添加投影 `alt + l + n`


# Tips #

带选区位移复制, 不会创建新图层

转换成组, `Ctrl + g`

v 状态下, 按住 ctrl, 实现快速选择(点选, 框选),
移动工具属性栏中, 自动选择不要勾选, 选择的对象给到图层

移动工具状态下, 在要选择的物件上, 右键, 出现光标下方所有图层列表,
选择目标图层, 配合图层命名使用

脸部, 使用亮色调突出, 使用暗色表现凹下

