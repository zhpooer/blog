title: haskell趣学-更多Monad
date: 2014-07-05 21:54:04
tags:
- haskell
---

# Writer #
~~~~~~
-- 定义在 Control.Monad.Writer
newtype Writer w a = Writer { runWriter :: (a, w) }

instance (Monoid w) => Monad (Writer w) where   
    return x = Writer (x, mempty)   
    (Writer (x,v)) >>= f = let (Writer (y, v')) = f x in Writer (y, v `mappend` v')

ghci> runWriter (return 3 :: Writer String Int)   
(3,"")   
ghci> runWriter (return 3 :: Writer (Sum Int) Int)   
(3,Sum {getSum = 0})

-- 使用 do 表达式
import Control.Monad.Writer   
logNumber :: Int -> Writer [String] Int   
logNumber x = Writer (x, ["Got number: " ++ show x])   
   
multWithLog :: Writer [String] Int   
multWithLog = do   
    a <- logNumber 3   
    b <- logNumber 5   
    return (a*b)

ghci> runWriter multWithLog   
(15,["Got number: 3","Got number: 5"])

-- 使用 tell函数, 它的类型是 MonadWriter
multWithLog :: Writer [String] Int   
multWithLog = do   
    a <- logNumber 3   
    b <- logNumber 5   
    tell ["Gonna multiply these two"]   
    return (a*b)
~~~~~~

# 使用Writer记录日志 #
~~~~~~
gcd' :: Int -> Int -> Int   
gcd' a b    
    | b == 0    = a   
    | otherwise = gcd' b (a `mod` b)

-- 记录log
gcd' :: Int -> Int -> Writer [String] Int   
gcd' a b   
  | b == 0 = do   
      tell ["Finished with " ++ show a]   
      return a   
  | otherwise = do   
      tell [show a ++ " mod " ++ show b ++ " = " ++ show (a `mod` b)]   
      gcd' b (a `mod` b)

ghci> mapM_ putStrLn $ snd $ runWriter (gcd' 8 3)   
8 mod 3 = 2   
3 mod 2 = 1   
2 mod 1 = 0   
Finished with 1
~~~~~~

在上面的例子中使用`Writer`时, list append动作实际是这样的
~~~~~~
a ++ (b ++ (c ++ (d ++ (e ++ f))))
-- 如果我们不小心使用使他变成下面这样的话, 效率不太高
((((a ++ b) ++ c) ++ d) ++ e) ++ f

import Control.Monad.Writer   
gcdReverse :: Int -> Int -> Writer [String] Int   
gcdReverse a b   
    | b == 0 = do   
      tell ["Finished with " ++ show a]   
      return a   
    | otherwise = do   
      result <- gcdReverse b (a `mod` b)   
      tell [show a ++ " mod " ++ show b ++ " = " ++ show (a `mod` b)]   
      return result
~~~~~~

# Difference lists #
~~~~~~
newtype DiffList a = DiffList { getDiffList :: [a] -> [a] }
toDiffList :: [a] -> DiffList a   
toDiffList xs = DiffList (xs++)   
   
fromDiffList :: DiffList a -> [a]   
fromDiffList (DiffList f) = f []

instance Monoid (DiffList a) where   
    mempty = DiffList (\xs -> [] ++ xs)   
    (DiffList f) `mappend` (DiffList g) = DiffList (\xs -> f (g xs))

ghci> fromDiffList (toDiffList [1,2,3,4] `mappend` toDiffList [1,2,3])   
[1,2,3,4,1,2,3]

-- 用 difference list 来加速gcdReverse
import Control.Monad.Writer   
gcd' :: Int -> Int -> Writer (DiffList String) Int   
gcd' a b   
  | b == 0 = do   
      tell (toDiffList ["Finished with " ++ show a])   
      return a   
  | otherwise = do   
      result <- gcd' b (a `mod` b)   
      tell (toDiffList [show a ++ " mod " ++ show b ++ " = " ++ show (a `mod` b)])   
      return result
~~~~~~

# Reader Monad #

TODO

~~~~~~
-- Functors
ghci> let f = (*5)   
ghci> let g = (+3) 
ghci> (fmap f g) 8

-- Functors 是 applictive
ghci> let f = (+) <$> (*2) <*> (+10) 
ghci> f 3 
19

-- Functors 是 Monad
instance Monad ((->) r) where   
    return x = \_ -> x   
    h >>= f = \w -> f (h w) w -- ???????

import Control.Monad.Instances   
addStuff :: Int -> Int   
addStuff = do   
  a <- (*2)   
  b <- (+10)   
  return (a+b)
  
-- 这样更清楚
addStuff :: Int -> Int   
addStuff x = let   
    a = (*2) x   
    b = (+10) x   
    in a+b
~~~~~~

# State Monad #
~~~~~~
type Stack = [Int]   
   
pop :: Stack -> (Int,Stack)   
pop (x:xs) = (x,xs)   
 
push :: Int -> Stack -> ((),Stack)   
push a xs = ((),a:xs)

-- 实际操作
stackManip :: Stack -> (Int, Stack)   
stackManip stack = let   
    ((),newStack1) = push 3 stack   
    (a ,newStack2) = pop newStack1   
    in pop newStack2
~~~~~~
使用State Monad 来简化操作

~~~~~~
newtype State s a = State { runState :: s -> (a,s) }
instance Monad (State s) where   
    return x = State $ \s -> (x,s)   
    (State h) >>= f = State $ \s -> let (a, newState) = h s   
                                        (State g) = f a   
                                    in  g newState

-- 在这里 一个操作被抽象出 State
import Control.Monad.State   
pop :: State Stack Int   
pop = State $ \(x:xs) -> (x,xs)

push :: Int -> State Stack ()   
push a = State $ \xs -> ((),a:xs)

-- 最终可以这样使用   
stackManip :: State Stack Int   
stackManip = do   
  push 3   
  a <- pop   
  pop
~~~~~~

# Random State #
~~~~~~
random :: (RandomGen g, Random a) => g -> (a, g)

import System.Random   
import Control.Monad.State   
randomSt :: (RandomGen g, Random a) => State g a   
randomSt = State random

import System.Random   
import Control.Monad.State   
   
threeCoins :: State StdGen (Bool,Bool,Bool)   
threeCoins = do   
  a <- randomSt   
  b <- randomSt   
  c <- randomSt   
  return (a,b,c)

ghci> runState threeCoins (mkStdGen 33)   
((True,False,True),680029187 2103410263)
~~~~~~

# Error Monad #

~~~~~~
instance (Error e) => Monad (Either e) where   
    return x = Right x    
    Right x >>= f = f x   
    Left err >>= f = Left err   
    fail msg = Left (strMsg msg)

ghci> Left "boom" >>= \x -> return (x+1)   
Left "boom"   
ghci> Right 100 >>= \x -> Left "no way!"   
Left "no way!"
~~~~~~


# functor 和 monad #

fucntors 是可以 map over的事务

applicative functors: 把一般的值放到一个缺省的 context中 (pure + map)

~~~~~~
liftM :: (Monad m) => (a -> b) -> m a -> m b
-- liftM函数, 接受 monadic value 然后 map over
fmap :: (Functor f) => (a -> b) -> f a -> f b

ghci> liftM (*3) (Just 8)   
Just 24   
ghci> fmap (*3) (Just 8)   
Just 24   
ghci> runWriter $ liftM not $ Writer (True, "chickpeas")   
(False,"chickpeas")   
ghci> runWriter $ fmap not $ Writer (True, "chickpeas")   
(False,"chickpeas")   
ghci> runState (liftM (+100) pop) [1,2,3,4]   
(101,[2,3,4])   
ghci> runState (fmap (+100) pop) [1,2,3,4]   
(101,[2,3,4])

-- liftM 具体实现, monad 包含 functor 和 applicative 的特性
liftM :: (Monad m) => (a -> b) -> m a -> m b   
liftM f m = m >>= (\x -> return (f x))

liftM :: (Monad m) => (a -> b) -> m a -> m b   
liftM f m = do   
    x <- m   
    return (f x)
~~~~~~
monad 和 applicative
~~~~~~
-- ap 相当于 applicative 中的 <*>, 
ap :: (Monad m) => m (a -> b) -> m a -> m b   
ap mf m = do   
    f <- mf   
    x <- m   
    return (f x)

-- liftM2 也做了相同的事情
liftA2 :: (Applicative f) => (a -> b -> c) -> f a -> f b -> f c   
liftA2 f x y = f <$> x <*> y
~~~~~~

# join function #
~~~~~~
join :: (Monad m) => m (m a) -> m a

ghci> join (Just (Just 9))   
Just 9   
ghci> join (Just Nothing)   
Nothing   
ghci> join Nothing   
Nothing

ghci> join [[1,2,3],[4,5,6]]   
[1,2,3,4,5,6]
ghci> runWriter $ join (Writer (Writer (1,"aaa"),"bbb"))   
(1,"bbbaaa")
ghci> join (Right (Right 9)) :: Either String Int   
Right 9   
ghci> join (Right (Left "error")) :: Either String Int   
Left "error"   
ghci> join (Left "error") :: Either String Int   
Left "error"

-- 事实上 m >>= f 永远等价于 join (fmap f m)
-- (Writer (x,v)) >>= f = let (Writer (y, v')) = f x in Writer (y, v `mappend` v')
-- join 的具体实现
join :: (Monad m) => m (m a) -> m a   
join mm = do   
    m <- mm   
    m
~~~~~~
# filterM #
~~~~~~
filterM :: (Monad m) => (a -> m Bool) -> [a] -> m [a]

keepSmall :: Int -> Writer [String] Bool   
keepSmall x   
    | x < 4 = do   
        tell ["Keeping " ++ show x]   
        return True   
    | otherwise = do   
        tell [show x ++ " is too large, throwing it away"]   
        return False

ghci> fst $ runWriter $ filterM keepSmall [9,1,5,2,10,3]   
[1,2,3]

ghci> mapM_ putStrLn $ snd $ runWriter $ filterM keepSmall [9,1,5,2,10,3]   
9 is too large, throwing it away   
Keeping 1

-- 这个函数想不通!!
--  list 其实就是 non-deterministic value 如何理解?
powerset :: [a] -> [[a]]   
powerset xs = filterM (\x -> [True, False]) xs

ghci> powerset [1,2,3]   
[[1,2,3],[1,2],[1,3],[1],[2,3],[2],[3],[]]
~~~~~~

# foldM #
~~~~~~
foldM :: (Monad m) => (a -> b -> m a) -> a -> [b] -> m a
binSmalls :: Int -> Int -> Maybe Int   
binSmalls acc x   
    | x > 9     = Nothing   
    | otherwise = Just (acc + x)

ghci> foldM binSmalls 0 [2,8,3,1]   
Just 14   
ghci> foldM binSmalls 0 [2,11,3,1]   
Nothing
~~~~~~

# RPN计算机 #

~~~~~~
import Data.List   
   
readMaybe :: (Read a) => String -> Maybe a   
readMaybe st = case reads st of [(x,"")] -> Just x   
                                _ -> Nothing
                                
foldingFunction :: [Double] -> String -> Maybe [Double]   
foldingFunction (x:y:ys) "*" = return ((x * y):ys)   
foldingFunction (x:y:ys) "+" = return ((x + y):ys)   
foldingFunction (x:y:ys) "-" = return ((y - x):ys)   
foldingFunction xs numberString = liftM (:xs) (readMaybe numberString)

import Data.List   
solveRPN :: String -> Maybe Double   
solveRPN st = do   
  [result] <- foldM foldingFunction [] (words st)   
  return result

ghci> solveRPN "1 2 * 4 +"   
Just 6.0   
ghci> solveRPN "1 2 * 4 + 5 *"   
Just 30.0   
ghci> solveRPN "1 2 * 4"   
Nothing   
ghci> solveRPN "1 8 wharglbllargh"   
Nothing
~~~~~~

`>=>`
~~~~~~
ghci> let f = (+1) . (*100)
ghci> f 4
401
ghci> let g = (\x -> return (x+1)) <=< (\x -> return (x*100))   
ghci> Just 4 >>= g
Just 401

ghci> let f = foldr (.) id [(+1),(*100),(+1)]   
ghci> f 1   
jt
~~~~~~
tK  1201t
