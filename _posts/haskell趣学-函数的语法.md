title: haskell趣学-常用的函数
date: 2014-06-28 19:19:17
tags:
- haskell
---

# 模式匹配 #
模式匹配决定参数的组成形式, Guard 进行参数的细化管理(过滤)
~~~~~~
sayMe :: (Integral a) => a -> String   
sayMe 1 = "One!"   
sayMe 2 = "Two!"   
sayMe 3 = "Three!"   
sayMe 4 = "Four!"   
sayMe 5 = "Five!"   
sayMe x = "Not between 1 and 5"
-- 将不会执行到, 模式从上到下执行
sayMe 6 = "six"


factorial :: (Integral a) => a -> a   
factorial 0 = 1   
factorial n = n * factorial (n - 1)  
~~~~~~

在定义模式时，一定要留一个万能匹配的模式，
这样我们的进程就不会为了不可预料的输入而崩溃了。
~~~~~~
charName :: Char -> String   
charName 'a' = "Albert"   
charName 'b' = "Broseph"   
charName 'c' = "Cecil"
-- when called
charName 'h' -- throws Exception

addVectors :: (Num a) => (a, a) -> (a, a) -> (a, a)   
addVectors a b = (fst a + fst b, snd a + snd b)
-- 或者可以这样
addVectors :: (Num a) => (a, a) -> (a, a) -> (a, a)   
addVectors (x1, y1) (x2, y2) = (x1 + x2, y1 + y2)

first :: (a, b, c) -> a   
first (x, _, _) = x   
 
second :: (a, b, c) -> b   
second (_, y, _) = y   
  
third :: (a, b, c) -> c   
third (_, _, z) = z

head' :: [a] -> a   
head' [] = error "Can't call head on an empty list, dummy!"   
head' (x:_) = x

tell :: (Show a) => [a] -> String   
tell [] = "The list is empty"   
tell (x:[]) = "The list has one element: " ++ show x   
tell (x:y:[]) = "The list has two elements: " ++ show x ++ " and " ++ show y   
tell (x:y:_) = "This list is long. The first two elements are: " ++ show x ++ " and " ++ show y

capital :: String -> String   
capital "" = "Empty string, whoops!"   
capital all@(x:xs) = "The first letter of " ++ all ++ " is " ++ [x]  
~~~~~~

## Guards ##

~~~~~~
bmiTell :: (RealFloat a) => a -> String   
bmiTell bmi   
    | bmi <= 18.5 = "You're underweight, you emo, you!"   
    | bmi <= 25.0 = "You're supposedly normal. Pffft, I bet you're ugly!"   
    | bmi <= 30.0 = "You're fat! Lose some weight, fatty!"   
    | otherwise   = "You're a whale, congratulations!"

-- 通过反单引号，我们不仅可以以中缀形式调用函数，
-- 也可以在定义函数的时候使用它。有时这样会更易读。
myCompare :: (Ord a) => a -> a -> Ordering   
a `myCompare` b   
    | a > b     = GT   
    | a == b    = EQ   
    | otherwise = LT 
~~~~~~

## where 关键字 ##

~~~~~~
bmiTell :: (RealFloat a) => a -> a -> String   
bmiTell weight height   
    | bmi <= skinny = "You're underweight, you emo, you!"   
    | bmi <= normal = "You're supposedly normal. Pffft, I bet you're ugly!"   
    | bmi <= fat    = "You're fat! Lose some weight, fatty!"   
    | otherwise     = "You're a whale, congratulations!"   
    where bmi = weight / height ^ 2   
          skinny = 18.5   
          normal = 25.0   
          fat = 30.0
-- 也可以这样
     where bmi = weight / height ^ 2   
           (skinny, normal, fat) = (18.5, 25.0, 30.0)

-- 在where里面定义函数
calcBmis :: (RealFloat a) => [(a, a)] -> [a]   
calcBmis xs = [bmi w h | (w, h) <- xs]  
    where bmi weight height = weight / height ^ 2 
~~~~~~

## Let 关键字 ##
let 绑定则是个表达式，允许你在任何位置定义局部变量，
而对不同的 guard 不可见。
~~~~~~
cylinder :: (RealFloat a) => a -> a -> a   
cylinder r h =  
    let sideArea = 2 * pi * r * h   
        topArea = pi * r ^2   
    in  sideArea + 2 * topArea

[let square x = x * x in (square 5, square 3, square 2)]
(let (a,b,c) = (1,2,3) in a+b+c) * 100

-- 它做的不是过滤，而是绑定名字
calcBmis :: (RealFloat a) => [(a, a)] -> [a]   
calcBmis xs = [bmi | (w, h) <- xs, let bmi = w / h ^ 2]
~~~~~~

## Case 表达 ##
模式匹配本质上不过就是 case 语句的语法糖而已。
这两段代码就是完全等价的：
~~~~~~
head' :: [a] -> a   
head' [] = error "No head for empty lists!"   
head' (x:_) = x

head' :: [a] -> a   
head' xs = case xs of [] -> error "No head for empty lists!"   
                      (x:_) -> x  
~~~~~~

case表达式的语法
~~~~~~
case expression of pattern -> result   
                   pattern -> result   
                   pattern -> result   
~~~~~~

~~~~~~
describeList :: [a] -> String   
describeList xs = "The list is " ++ what xs   
    where what [] = "empty."   
          what [x] = "a singleton list."   
          what xs = "a longer list."
~~~~~~


# 递归 #
递归实际上是定义函数以调用自身的方式,
在递归定义中声明的一两个非递归的值, 称作边界条件.

## Max函数 ##
`maximum` 函数取一组可排序的 List 做参数，并回传其中的最大值.

命令式:
~~~~~~
def maximum(list:List):Int = {
    var maxNum = 0
    for(i <- list) maxNum = maximum(i, maxNum)
    return maxNum
}
~~~~~~
函数式(递归):
~~~~~~
maximum :: (Ord a) => [a] -> a
maximum [] = error "empty list"
maximum [x] = x
maximum (x:xs) = max x (maximum xs)
~~~~~~

## 其他案例 ##
~~~~~~
take :: (Num i, Ord i) => i -> [a] -> [a]
take n _
    | n <= 0 = []
take _ []    = []
take n (x:xs) = x : take (n-1) xs


reverse :: [a] -> a
reverse [] = []
reverse [x:xs] = reverse xs ++ [x]


repeat :: a -> [a]
repeat x = x : repeat x

repeat 3  -- 会永远的执行下去
take 5 repeat 3  -- 得到3个


zip :: [a] -> [b] -> [(a, b)]
zip [] _ = []
zip _ [] = []
zip (x:xs) (y:ys) = (x, y) : zip xs ys
~~~~~~


## 快速排序 ##

~~~~~~
quicksort :: (Ord a) => [a] -> a
quicksort [] = []
quicksort (x:xs) = smallerSorted ++ [x] + biggerSorted
    where smallerSorted = quicksort [a | a <- xs, a <= x]
          biggerSorted = quicksort [a | a <- xs, a > x]
~~~~~~
