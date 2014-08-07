title: haskell趣学-入门
date: 2014-06-25 14:58:38
tags:
- haskell
---

# 什么是Heskell #

* 纯函数式语言
  * 命令式编程, 变量在执行过后, 会发生变化(状态变化)
  * 函数式, 函数唯一做的事是引用计数结果, 不产生副作用(如改变全局变量)
  * 应用透明, 若以同样的参数调用同一个函数两次，得到的结果一定是相同.
  编译器理解程序的行为, 容易验证函数的正确性, 使简单组合成复杂
* 惰性(lazy), 若以同样的参数调用同一个函数两次, 得到的结果一定是相同值.
  * 结合引用透明, 可以把程序仅看作是数据的一系列变形
* 静态类型, 编译器检查错误, 自动类型推导

## 使用Haskell ##
编译器 GHC, `apt-get install Haskell-platform`
~~~~~~
ghci> :l myfunction.hs
~~~~~~

# 入门 #

    ghci> 2 + 15 
    17 
    ghci> 5 / 2 
    2.5
    ghci> False || True 
    True 
    ghci> not False
    True
    ghci> 1 == 0 
    False 
    ghci> 5 /= 5 
    False


    ghci> 5+"llama" -- 运算符要求两端都是数值
    ghci> 5==True  -- 报错, `==`对两个可比较的值可用, 橘子和苹果没法比较


中缀函数: `*`, `+`, 大多数命令式编程语言中的函数调用形式通常就是函数名,括号,
由逗号分隔的参数表.
在Haskell 中,函数调用的形式是函数名,空格,空格分隔的参数表
~~~~~~
succ 8  -- successor, 返回一个数的后继
min 8 9

-- 函数调用拥有最高的优先级
succ 9 + max 5 4 + 1      -- (succ 9) + (max 5 4) + 1
succ 9*10                 -- (succ 9)*10

-- 使用中缀是函数清晰
div 92 10
92 `div` 10
~~~~~~

## 定义函数 ##

~~~~~~
-- 声明一个函数, 功能是一个数字乘2
doubleMe x = x + x

doubleUs x y = x*2 + y*2     --  doubleUs x y = doubleMe x + doubleMe y

-- 首字母大写的函数是不允许
doubleSmallNumber x = (if x > 100 then x else x*2) + 1 
~~~~~~

## List ##

List 是一种单类型的数据结构,
可以用来存储多个类型相同的元素.
们可以在里面装一组数字或者一组字符, 但不能把字符和数字装在一起.

在 ghci 下，我们可以使用 `let` 关键字来定义一个常量.
在 ghci 下执行 `let a =1` 与在脚本中编写 `a=1` 是等价的.
~~~~~~
ghci> let lostNumbers = [4,8,15,16,23,48]   
ghci> lostNumbers   
[4,8,15,16,23,48]

-- 一个 List 由方括号括起，其中的元素用逗号分隔开来
ghci> [1,2,3,4] ++ [9,10,11,12]   
[1,2,3,4,9,10,11,12]

-- 字串实际上就是一组字符的 List，"Hello" 只是 ['h','e','l','l','o']
-- ++ 有效率问题
ghci> "hello" ++ " " ++ "world"   
"hello world"   
ghci> ['w','o'] ++ ['o','t']   
"woot"

-- 用 : 运算符往一个 List 前端插入元素
-- [1,2,3] 实际上是 1:2:3:[] 的语法糖
ghci> 5:[1,2,3,4,5]  
[5,1,2,3,4,5]

-- 索引
ghci> [9.4,33.2,96.2,11.2,23.25] !! 1
33.2

-- 常用函数
ghci> head [5,4,3,2,1]  
5
ghci> tail [5,4,3,2,1]   
[4,3,2,1]  
ghci> last [5,4,3,2,1]   
1
ghci> init [5,4,3,2,1] 
[5,4,3,2]
ghci> length [5,4,3,2,1]   
5

ghci> null [1,2,3]   
False   
ghci> null []   
True

ghci> reverse [5,4,3,2,1]   
[1,2,3,4,5]
ghci> take 3 [5,4,3,2,1]   
[5,4,3]   
ghci> take 1 [3,9,3]   
[3]
ghci> drop 3 [8,4,2,1,5,6]   
[1,5,6]

ghci> minimum [8,4,2,1,5,6]   
1   
ghci> maximum [1,9,2,3,4]   
9

ghci> sum [5,2,1,6,3,2,5,7]   
31   
ghci> product [6,2,1,2]   -- 返回元素的积
24

-- 判断一个元素是否在包含于一个 List，通常以中缀函数的形式调用它
ghci> 4 `elem` [3,4,5,6]   
True   
ghci> 10 `elem` [3,4,5,6]   
False 
~~~~~~

## 使用 Range ##

~~~~~~
ghci> [1..20] 
[1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20] 
ghci> ['a'..'z'] 
"abcdefghijklmnopqrstuvwxyz"

-- 跨步
ghci> [2,4..20] 
[2,4,6,8,10,12,14,16,18,20]

-- 由于是惰性的, 可以得到无限长度 List
ghci> take 24 [13,26..]
ghci> take 10 (cycle [1,2,3])
ghci> take 10 (repeat 5) 
~~~~~~

## List Comprehension ##
~~~~~~
ghci> [x*2 | x <- [1..10]] 
[2,4,6,8,10,12,14,16,18,20]

ghci> [x*2 | x <- [1..10], x*2 >= 12] 
[12,14,16,18,20]

ghci> boomBangs xs = [ if x < 10 then "BOOM!" else "BANG!" | x <- xs, odd x]

-- 过滤若条件为 false, 则不被包含
ghci> [ x | x <- [10..20], x /= 13, x /= 15, x /= 19] 
[10,11,12,14,16,17,18,20]

ghci> [ x*y | x <-[2,5,10], y <- [8,10,11], x*y > 50] 
[55,80,100,110]

-- 自己定义length
-- _ 表示我们并不关心从 List 中取什么值，与其弄个永远不用的变量，不如直接一个 _
ghci> length' xs = sum [1 | _ <- xs]

-- 去除所有的奇数
ghci> let xxs = [[1,3,5,2,3,1,2,4,5],[1,2,3,4,5,6,7,8,9],[1,2,4,2,1,6,3,1,3,2,3,6]] 
ghci> [ [ x | x <- xs, even x ] | xs <- xxs] 
[[2,2,4],[2,4,6,8],[2,4,2,6,2,6]]
~~~~~~

## 元组(Tuple) ##
Tuple (元组)很像 List, 都是将多个值存入一个个体的容器.
Tuple 则要求你对需要组合的数据的数目非常的明确，
它的类型取决于其中项的数目与其各自的类型.(可以装载不同类型的数据)

`[(1,2),(8,11),(4,5)]` 相较 `[[1,2],[8,11,5],[4,5]]`

`[(1,2),(8,11,5),(4,5)]`，`[(1,2),("one",2)]` 都会报错

~~~~~~
ghci> fst (8,11) 
8 
ghci> fst ("Wow", False) 
"Wow"

ghci> snd (8,11) 
11 
ghci> snd ("Wow", False) 
False
-- 这两个函数仅对序对有效, 不能应用于三元组，四元组和五元组之上

ghci> zip [1..] ["apple", "orange", "cherry", "mango"] 
[(1,"apple"),(2,"orange"),(3,"cherry"),(4,"mango")]

-- 告诉它只要周长为 24 的直角三角形
-- 同时也考虑上 b 边要短于斜边，a 边要短于 b 边情况
ghci> let rightTriangles' = [ (a,b,c) | c <- [1..10], b <- [1..c], a <- [1..b], a^2 + b^2 == c^2, a+b+c == 24] 
ghci> rightTriangles' 
[(6,8,10)]

~~~~~~
