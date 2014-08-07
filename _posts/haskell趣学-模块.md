title: haskell趣学-模块
date: 2014-06-29 14:16:24
tags:
- haskell
---

Haskell 中的模块是含有一组相关的函数，类型和类型类的组合.

Haskell 进程的本质便是从主模块中
引用其它模块并调用其中的函数来执行操作

`Prelude` 模块包含一些基本函数,类型以及类型类，它缺省自动装载

~~~~~~
import Data.List   
-- nub 筛掉重复元素
numUniques :: (Eq a) => [a] -> Int   
numUniques = length . nub
~~~~~~

在 ghci 中装载模块
~~~~~~
ghci> :m Data.List Data.Map Data.Set
~~~~~~
import 各种语法: 
~~~~~~
-- 仅装载 Data.List 模块 nub 和 sort
import Data.List (nub, sort)

-- 装载 Data.List 除了 nub
import Data.List hiding (nub)

-- 使用: Data.Map.filter
import qualified Data.Map
-- 更加简短: M.filter
import qualified Data.Map as M
~~~~~~

**翻阅标准库中的模块和函数是提升个人 Haskell 水平的重要途径**
[Hoogle](http://www.haskell.org/hoogle/)

`Data.List`, `Data.Char`, `Data.Map`, `Data.Set`使用详解: 
http://learnyouahaskell-zh-tw.csie.org/zh-cn/modules.html


# 建立自己的模块 #
构造一个由计算机几何图形体积和编辑组成的模块 `Geometry.hs`
~~~~~~
module Geometry
( sphereVolumn
, sphereArea
, cubeVolume
, cubeArea
, cuboidArea
, cuboidVolume
) where
shpereVolume :: Float -> Float
sphereVolume = (4.0 / 3.0) * pi * (raius^3)

sphereArea :: Float -> Float   
sphereArea radius = 4 * pi * (radius ^ 2)   
 
cubeVolume :: Float -> Float   
cubeVolume side = cuboidVolume side side side   
 
cubeArea :: Float -> Float   
cubeArea side = cuboidArea side side side   
 
cuboidVolume :: Float -> Float -> Float -> Float   
cuboidVolume a b c = rectangleArea a b * c   
 
cuboidArea :: Float -> Float -> Float -> Float   
cuboidArea a b c = rectangleArea a b * 2 + rectangleArea a c * 2 + rectangleArea c b * 2   
 
rectangleArea :: Float -> Float -> Float   
rectangleArea a b = a * b
~~~~~~

可以把 `Geometry` 分成三个子模块
`sphere.hs`
~~~~~~
module Geometry.Sphere   
( volume   
，area   
) where   
 
volume :: Float -> Float   
volume radius = (4.0 / 3.0) * pi * (radius ^ 3)   
 
area :: Float -> Float   
area radius = 4 * pi * (radius ^ 2)
~~~~~~
`cuboid.hs`
~~~~~~
module Geometry.Cuboid   
( volume   
，area   
) where   
 
volume :: Float -> Float -> Float -> Float   
volume a b c = rectangleArea a b * c
~~~~~~
cube.hs
~~~~~~
module Geometry.Cube   
( volume   
，area   
) where   
 
import qualified Geometry.Cuboid as Cuboid   
 
volume :: Float -> Float   
volume side = Cuboid.volume side side side   
 
area :: Float -> Float   
area side = Cuboid.area side side side
~~~~~~
