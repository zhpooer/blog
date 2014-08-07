title: haskell趣学-TypeClass
date: 2014-06-28 10:15:03
tags:
- haskell
---

# Type #
Haskell 支持类型推导
~~~~~~
ghci> :t 'a'   
'a' :: Char   
ghci> :t True   
True :: Bool   
ghci> :t "HELLO!"   
"HELLO!" :: [Char] 
~~~~~~

函数也有类型
~~~~~~
removeNonUppercase :: [Char] -> [Char]
removeNonUppercase st = [ c | c <- st, c `elem` ['A'..'Z']]

-- Integer 也是整数，但它是无界的, Int 是有界的
factorial :: Integer -> Integer   
factorial n = product [1..n]

circumference :: Float -> Float   
circumference r = 2 * pi * r

circumference' :: Double -> Double   
circumference' r = 2 * pi * r

-- Bool 表示布林值，它只有两种值：True 和 False
-- Char 表示一个字符。一个字符由单引号括起，一组字符的 List 即为字串。
-- Tuple 的类型取决于它的长度及其中项的类型。注意，空 Tuple 同样也是个类型，它只有一种值：()
~~~~~~

# Type variables(类型变量?泛型?) #
类型变量, `head :: [a] -> a`, 意味着 a 可以是任意的类型,
使用到类型变量的函数被称作"多态函数 "，
`head` 函数的类型声明里标明了它可以取任意类型的
`List` 并回传其中的第一个元素

~~~~~~
-- a 和 b 是不同的类型变量
fst :: (a, b) -> a
~~~~~~

# Typeclasses #

类型定义行为的接口，如果一个类型属于某 Typeclass，
那它必实现了该 Typeclass 所描述的行为, 可以看做是 java 的 interface

~~~~~~
-- +-*/之类的运算符也是同样。在缺省条件下，它们多为中缀函数。
-- 若要检查它的类型，就必须得用括号括起使之作为另一个函数，或者说以首码函数的形式调用它。
-- => 符号, 它左边的部分叫做类型约束
-- "相等函数取两个相同类型的值作为参数并回传一个布林值，
-- 而这两个参数的类型同在 Eq 类之中(即类型约束)"
ghci> :t (==)
(==) :: (Eq a) => a -> a -> Bool

-- Num 相当于泛型?
addVectors :: (Num a) => (a, a) -> (a, a) -> (a, a)   
addVectors a b = (fst a + fst b, snd a + snd b)
~~~~~~

## 常用的 TypeClass ##

`Eq` 包含可判断相等性的类型, 提供实现的函数是 == 和 /=

`Ord` 包含可比较大小的类型. 包含了<, >, <=, >= 之类用于比较大小的函数..

`Show` 的成员为可用字串表示的类型, 可以取任一Show的成员类型并将其转为字串
~~~~~~
ghci> show 3   
"3"   
ghci> show 5.334   
"5.334"   
ghci> show True   
"True"
~~~~~~

`Read` 函数可以将一个字串转为 Read 的某成员类型
~~~~~~
ghci> read "True" || False   
True   
ghci> read "8.2" + 3.8   
12.0   
ghci> read "5" - 2   
3   
ghci> read "[1,2,3,4]" ++ [3]   
[1,2,3,4,3]

-- 搞不清楚究竟该是 Int 还是 Float 了
ghci> read "5" :: Int   
5   
ghci> read "5" :: Float   
5.0   
ghci> (read "5" :: Float) * 4   
20.0   
ghci> read "[1,2,3,4]" :: [Int]   
[1,2,3,4]   
ghci> read "(3, 'a')" :: (Int, Char)   
(3, 'a')
~~~~~~

`Bounded` 的成员都有一个上限和下限
~~~~~~
ghci> minBound :: Int   
-2147483648   
ghci> maxBound :: Char   
'\1114111'   
ghci> maxBound :: Bool   
True   
ghci> minBound :: Bool   
False
~~~~~~

`Enum` 的成员都是连续的类型,
个值都有后继子 (successer) 和前置子 (predecesor)，
分别可以通过 succ 函数和 pred 函数得到
~~~~~~
ghci> ['a'..'e']   
"abcde"   
ghci> [LT .. GT]   
[LT,EQ,GT]   
ghci> [3 .. 5]   
[3,4,5]   
ghci> succ 'B'   
'C'
~~~~~~

`Num` 是表示数字的 Typeclass，它的成员类型都具有数字的特征.
类型只有亲近 `Show` 和 `Eq`，才可以加入 Num

`Integral` 同样是表示数字的 Typeclass。
Num 包含所有的数字：实数和整数。
而 Intgral 仅包含整数，其中的成员类型有 Int 和 Integer

`Floating` 仅包含浮点类型：Float 和 Double
