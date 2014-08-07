title: haskell趣学-高阶函数
date: 2014-06-29 09:40:54
tags:
- haskell
---

函数可以作为参数和回传值传来传去，这样的函数就被称作高阶函数.
在haskell中拒绝循环与状态的改变而通过定义问题"是什么"来解决问题

# 柯里化(Curried functions) #
柯里化（Currying）是把接受多个参数的函数变换成接受一个单一参数(最初函数的第一个参数)的函数，
并且返回接受余下的参数且返回结果的新函数的技术
~~~~~~
-- 两个函数是等价的
ghci> max 4 5 
5 
ghci> (max 4) 5 
5
~~~~~~

`max :: (Ord a) a -> a -> a` 也可以写作 `max :: (Ord a) a -> (a -> a)`

~~~~~~
multThree :: (Num a) => a -> a -> a -> a 
multThree x y z = x * y * z

ghci> let multTwoWithNine = multThree 9 
ghci> multTwoWithNine 2 3 
54 
ghci> let multWithEighteen = multTwoWithNine 2 
ghci> multWithEighteen 10 
180

-- 通过不全的调用创造新的函数
compareWithHundred :: (Num a，Ord a) => a -> Ordering 
compareWithHundred x = compare 100 x
-- 等价于, 不全调用
compareWithHundred :: (Num a, Ord a) => a -> Ordering
compareWithHundred = compare 100

-- 中缀, 和括号组合
divideByTen :: (Floating a) => a -> a 
divideByTen = (/10)
~~~~~~

# 高阶函数 #

~~~~~~
-- 函数中取另一个函数做参数, 函数调用两次
-- -> 符号是右结合
applyTwice :: (a -> a) -> a -> a
applyTwice f x = f (f x)

zipWith :: (a -> b -> c) -> [a] -> [b] -> [c]
zipWith _ [] _ = []
zipWith _ _ [] = []
zipWith f (x:xs) (y:ys) = f x y :: zipWith f xs ys

-- 传入一个函数 返回一个相似函数, 但是两个函数的参数位置调换
filp :: (a -> b -> c) -> (b -> a -> c)
flip f = g
     where g x y = f y x
~~~~~~


## map 和 filter ##

~~~~~~
map :: (a -> b) -> [a] -> [b]
map _ [] = []
map f (x:xs) = f x : map f xs

filter :: (a -> Bool) -> [a] -> [a]
filter _ [] = []
filter p (x:xs)
    | p x = x : filter p xs
    | otherwise = filter p xs


ghci> let listOfFuns = map (*) [0..]   
ghci> (listOfFuns !! 4) 5   
20
~~~~~~

# lambda #
lambda 就是匿名函数。
有些时候我们需要传给高阶函数一个函数，而这函数我们只会用这一次，
这就弄个特定功能的 lambda.
~~~~~~
numLongChains :: Int   
numLongChains = length (filter (\xs -> length xs > 15) (map chain [1..100]))

ghci> zipWith (\a b -> (a * 30 + 3) / b) [5,4,3,2,1] [1,2,3,4,5]

flip :: (a -> b -> c) -> b -> a -> c
flip = \x y -> f y x
~~~~~~

# fold #

~~~~~~
sum :: (Num a) => [a] -> a
sum xs = foldl (\acc x -> acc + x) 0 xs
-- 等价于
sum :: (Num a) => [a] -> a
sum = foldl (+) 0

reverse :: [a] -> [a]
reverse = foldl (\acc x -> x : acc) []
-- 等价于
reverse = foldl1 (\acc x -> x : acc)

head' :: [a] -> a   
head' = foldr1 (\x _ -> x)   
 
last' :: [a] -> a   
last' = foldl1 (\_ x -> x)
~~~~~~

`scanl` 和 `scanr` 与 `foldl` 和 `foldr` 相似，
只是它们会记录下累加值的所有状态到一个 List。也有 `scanl1` 和 `scanr1`

`scan` 可以用来跟踪 `fold` 函数的执行过程
~~~~~~
ghci> scanl (+) 0 [3, 5, 2 1]
[0,3,8,10,11]
ghci> scanr (+) 0 [3,5,2,1]
[11,8,3,1,0]   
ghci> scanl1 (\acc x -> if x > acc then x else acc) [3,4,5,3,7,9,2,1]   
[3,4,5,5,7,9,9,9]   
ghci> scanl (flip (:)) [] [3,2,1]   
[[],[3],[2,3],[1,2,3]]
~~~~~~

# `$` 函数调用 #
~~~~~~
($) :: (a -> b) -> a -> b
f $ x = f x
~~~~~~
普通的函数调用符有最高的优先级，而 $ 的优先级则最低。
用空格的函数调用符是左结合的，如 `f a b c` 与 `((f a) b) c` 等价，
而 `$` 则是右结合的。`f (g (z x))` 与 `f $ g $ z x` 等价
~~~~~~
> sqrt $ 3+4+9
> sqrt 3 + 4 + 9

> sum $ filter (> 10) $ map (*2) [2..10]
~~~~~~

# Function composition(函数组合) #

~~~~~~
(.) :: (b -> c) -> (a -> b) -> a -> c   
f . g = \x -> f (g x)
~~~~~~

~~~~~~
ghci> map (negate . abs) [5,-3,-6,7,-3,2,-19,24]

oddSquareSum :: Integer   
oddSquareSum = sum . takeWhile (<10000) . filter odd . map (^2) $ [1..]
~~~~~~
