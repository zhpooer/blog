title: haskell趣学-Monad
date: 2014-07-02 08:43:39
tags:
- haskell
---

Functor 是因为我们观察到有许多态态都可以被 function 给 map over，
了解到这个目的，便抽象化了 `Functor` 这个 typeclass 出来

~~~~~~
-- >>= bind 操作, scala 中的 flatMap!!
(>>=) :: (Monad m) => m a -> (a -> m b) -> m b

-- 包装函数, 来调用实参
ghci> (\x -> Just (x+1)) 1   
Just 2   
ghci> (\x -> Just (x+1)) 100   
Just 110

-- Monad 是一个容器(盒子)
class Monad m where   
    return :: a -> m a   
    (>>=) :: m a -> (a -> m b) -> m b   
    (>>) :: m a -> m b -> m b   
    x >> y = x >>= \_ -> y
    fail :: String -> m a
    fail msg = error msg

instance Monad Maybe where
    return x = Just x
    Nothing >>= f = Nothing
    Just x >>= f = f x
    fail _ = Nothing

ghci> return "WHAT" :: Maybe String   
Just "WHAT"   
ghci> Just 9 >>= \x -> return (x*10)   
Just 90   
ghci> Nothing >>= \x -> return (x*10)   
Nothing

~~~~~~

# 走钢丝案例 #

有人走钢丝, 有小鸟会落在钢丝的左右两边,
如果两边的小鸟超过三只, 那么就会失败

~~~~~~
type Birds = Int
type Pole = (Birds, Birds)

landLeft :: Birds -> Pole -> Pole
landLeft n (left, right) = (left + n, right)

ghci> landLeft 2 (landRight 1 (landLeft 1 (0,0)))   
(3,1)

-- 定义一个操作符
x -: f = f x

-- 这样的模拟bug, 如果在中间步骤已经失败了, 那会如何呢
ghci> (0,0) -: landLeft 1 -: landRight 1 -: landLeft 2   
(3,1)

-- 使用 Maybe 重构
landLeft :: Birds -> Pole -> Maybe Pole
landLeft :: n (left, right)
    | abs ((left + n) - right) < 4 = Just (left + n, right)
    | otherwise = Nothing
landRight :: Birds -> Pole -> Maybe Pole   
landRight n (left,right)   
    | abs (left - (right + n)) < 4 = Just (left, right + n)   
    | otherwise                    = Nothing

ghci> landRight 1 (0,0) >>= landLeft 2   
Just (2,1)

ghci> return (0,0) >>= landRight 2 >>= landLeft 2 >>= landRight 2   
Just (2,4)

-- 如果不用 Monad 
routine :: Maybe Pole   
routine = case landLeft 1 (0,0) of   
    Nothing -> Nothing   
    Just pole1 -> case landRight 4 pole1 of    
            Nothing -> Nothing   
            Just pole2 -> case landLeft 2 pole2 of   
                    Nothing -> Nothing   
                    Just pole3 -> landLeft 1 pole3
~~~~~~

# do 用法 #
`do` 把 monadic value 串成一串

~~~~~~
ghci> Just 3 >>= (\x -> Just "!" >>= (\y -> Just (show x ++ y)))   
Just "3!"

foo :: Maybe String   
foo = Just 3   >>= (\x ->  
      Just "!" >>= (\y ->  
      Just (show x ++ y)))

-- 使用 do 表示法
foo :: Maybe String
foo = do
    x <- Just 3
    y <- Just "!"
    Just (show x ++ y)

-- 简化上一步的代码
-- 他们只是代表串行而已，每一步的值都倚赖前一步的结果
-- 并带着他们的 context 继续下去
foo :: Maybe String   
foo = do   
    x <- Just 3   
    y <- Just "!"   
    Just (show x ++ y)

-- 有用到 <- 来绑定值的话，其实实际上就是用了 >>，他会忽略掉计算的结果
routine :: Maybe Pole   
routine = do   
    start <- return (0,0)   
    first <- landLeft 2 start   
    Nothing   
    second <- landRight 2 first   
    landLeft 1 second
~~~~~~

都用于模式匹配

~~~~~~
justH :: Maybe Char   
justH = do   
    (x:xs) <- Just "hello"   
    return x

-- 考虑到失败情形
fail :: (Monad m) => String -> m a   
fail _ = Nothing

wopwop :: Maybe Char   
wopwop = do   
    (x:xs) <- Just ""   
    return x
~~~~~~

# List Monad #

~~~~~~
instance Monad [] where   
    return x = [x]   
    xs >>= f = concat (map f xs)   
    fail _ = []

ghci> [3,4,5] >>= \x -> [x,-x]   
[3,-3,4,-4,5,-5]
ghci> [] >>= \x -> ["bad","mad","rad"]   
[]   
ghci> [1,2,3] >>= \x -> []   
[]

ghci> [1,2] >>= \n -> ['a','b'] >>= \ch -> return (n,ch)   
[(1,'a'),(1,'b'),(2,'a'),(2,'b')]

-- 用do重构
listOfTuples :: [(Int,Char)]   
listOfTuples = do   
    n <- [1,2]   
    ch <- ['a','b']   
    return (n,ch)
-- 用 list comprehension 重构
ghci> [ (n,ch) | n <- [1,2], ch <- ['a','b'] ]   
[(1,'a'),(1,'b'),(2,'a'),(2,'b')]

-- 在 list comprehension 中使用 过滤条件
ghci> [ x | x <- [1..50], '7' `elem` show x ]   
[7,17,27,37,47]

-- 使用 MonadPlus 模拟 Monoid
class Monad m => MonadPlus m where
    mzero :: m a
    mplus :: m a -> m a -> m a
instance MonadPlus [] where   
    mzero = []   
    mplus = (++)
-- 定于 guard 方法
guard :: (MonadPlus m) => Bool -> m ()   
guard True = return ()   
guard False = mzero

-- guard 使用
ghci> [1..50] >>= (\x -> guard ('7' `elem` show x) >> return x)   
[7,17,27,37,47]
ghci> guard (5 > 2) >> return "cool" :: [String]   
["cool"]   
ghci> guard (1 > 2) >> return "cool" :: [String]   
[]

sevensOnly :: [Int]   
sevensOnly = do   
    x <- [1..50]   
    guard ('7' `elem` show x)   
    return x
-- 语法糖如下
ghci> [ x | x <- [1..50], '7' `elem` show x ]   
[7,17,27,37,47]
~~~~~~
