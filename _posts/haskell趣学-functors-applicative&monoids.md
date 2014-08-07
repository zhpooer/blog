title: haskell趣学-Functors Applicative&Monoids
date: 2014-07-01 15:09:10
tags:
- haskell
---
Functors 是可以被 map over 的对象，
像是 lists，Maybe，trees 等等。
在 Haskell 中用 Functor 这个 typeclass 来描述他。
这个 typeclass 只有一个 method，叫做 fmap，他的型态是
`fmap :: (a -> b) -> f a -> f b`

把盒子里的东西取出来, 调用函数 `f`, 并放回去.
至于如何取出来(List 是一个一个取出来, Maybe 取出可以取出的),
是看 Functors 的具体实现类

~~~~~~
import Data.Char   
import Data.List   
   
main = do line <- fmap (intersperse '-' . reverse . map toUpper) getLine   
        putStrLn line

-- (->) r a 也可以是一个 Functor
-- (a -> b) -> (r -> a) -> (r -> b)
-- 跟一个从 r 到 a 的 function，并回传一个从 r 到 b 的 function
instance Functor ((->) r) where   
    fmap f g = (\x -> f (g x))

-- fmap = (.)
ghci> :t fmap (*3) (+100)   
fmap (*3) (+100) :: (Num a) => a -> a   
ghci> fmap (*3) (+100) 1   
303   
ghci> (*3) `fmap` (+100) $ 1   
303   
ghci> (*3) . (+100) $ 1   
303   
ghci> fmap (show . (*3)) (*100) 1   
"300"
~~~~~~

Haskell 的 function 实际上只接受一个参数。
一个型态是 `a -> b -> c` 的函数实际上是接受 a
然后回传 `b -> c`，而 `b -> c` 实际上接受一个 b 然后回传一个 c。
如果我们用比较少的参数调用一个函数，
他就会回传一个函数需要接受剩下的参数。
所以 `a -> b -> c` 可以写成 `a -> (b -> c)`

`fmap` 可以想象成 接受 function 并回传一个新的 function 的 function,
回传的 function 接受一个 functor 并回传一个 functor

他接受 `a -> b` 并回传 `f a -> f b`。这动作叫做 *lifting*, functor的柯里化

~~~~~~
-- fmap (*2) 接受一个 functor f，并回传一个基于数字的 functor
-- 这个 functor 可以是 Maybe, Either String
ghci> :t fmap (*2)   
fmap (*2) :: (Num a, Functor f) => f a -> f a
-- fmap (replicate 3) 可以接受一个基于任何型态的 functor，
-- 并回传一个基于 list 的 functor
ghci> :t fmap (replicate 3)   
fmap (replicate 3) :: (Functor f) => f a -> f [a]

-- 理解为柯里化

ghci> fmap (replicate 3) [1,2,3,4]   
[[1,1,1],[2,2,2],[3,3,3],[4,4,4]]   
ghci> fmap (replicate 3) (Just 4)   
Just [4,4,4]   
ghci> fmap (replicate 3) (Right "blah")   
Right ["blah","blah","blah"]   
ghci> fmap (replicate 3) Nothing   
Nothing   
ghci> fmap (replicate 3) (Left "foo")   
Left "foo"
~~~~~~


functor law 的第一条说明，如果我们对 functor 做 map id，
那得到的新的 functor 应该要跟原来的一样。
~~~~~~
-- id 只是 identity function，他只会把参数照原样丢出
ghci> fmap id (Just 3)   
Just 3   
ghci> id (Just 3)   
Just 3   
ghci> fmap id [1..5]   
[1,2,3,4,5]   
ghci> id [1..5]   
[1,2,3,4,5]   
ghci> fmap id []   
[]   
ghci> fmap id Nothing   
Nothing
~~~~~~

functors的结合律(第二定律):
`fmap (f . g) = fmap f . fmap g`, 
`fmap (f . g) F = fmap f (fmap g F)`

如果遵守了 functor laws，我们知道对他做 fmap 不会做多余的事情，
只是用一个函数做映射而已。
这让写出来的代码足够抽象也容易扩展。
因为我们可以用定律来推论类型的行为。
~~~~~~
-- 违反定律的案例
data CMaybe a = CNothing | CJust Int a deriving (Show)
instance Functor CMaybe where   
    fmap f CNothing = CNothing   
    fmap f (CJust counter x) = CJust (counter+1) (f x)

ghci> fmap id (CJust 0 "haha")   
CJust 1 "haha"   
ghci> id (CJust 0 "haha")   
CJust 0 "haha"
~~~~~~

# Applicative functors #
它的核心是包装函数, 供盒子调用, 如何调用是盒子的具体实现
~~~~~~
class (Functor f) => Applicative f where
    -- 包装成`盒子`, 主要用作包装 函数
    pure :: a -> f a
    -- 装在盒子里的是 一个柯里化的函数
    (<*>) :: f (a -> b) -> f a -> f b

instance Applicative Maybe where   
    pure = Just   
    Nothing <*> _ = Nothing   
    (Just f) <*> something = fmap f something

ghci> Just (+3) <*> Just 9   
Just 12   
ghci> pure (+3) <*> Just 10   
Just 13   
ghci> pure (+3) <*> Just 9   
Just 12   
ghci> Just (++"hahah") <*> Nothing   
Nothing   
ghci> Nothing <*> Just "woot"   
Nothing

ghci> pure (+) <*> Just 3 <*> Just 5   
Just 8   
ghci> pure (+) <*> Just 3 <*> Nothing   
Nothing   
ghci> pure (+) <*> Nothing <*> Just 5   
Nothing
~~~~~~

`pure f <*> x` 等于 `fmap f x`, 中缀版的 `fmap`
~~~~~~
(<$>) :: (Functor f) => (a -> b) -> f a -> f b   
f <$> x = fmap f x

-- 如果三个 applicative functor, 可以写成 f <$> x <*> y <*> z。
-- 如果参数是普通值的话。我们则写成 f x y z

ghci> (++) <$> Just "johntra" <*> Just "volta"   
Just "johntravolta"
ghci> (++) "johntra" "volta"   
"johntravolta"
~~~~~~

~~~~~~
ghci> pure "Hey" :: [String]   
["Hey"]   
ghci> pure "Hey" :: Maybe String   
Just "Hey"

-- List 可以当做一个容器
-- List instance: (<*>) :: [a -> b] -> [a] -> [b]
ghci> [(*0),(+100),(^2)] <*> [1,2,3]   
[0,0,0,101,102,103,1,4,9]
ghci> [(+),(*)] <*> [1,2] <*> [3,4]   
[4,5,5,6,3,4,6,8]

ghci> [ x*y | x <- [2,5,10], y <- [8,10,11]]      
[16,20,22,40,50,55,80,100,110]
-- 比较这两个方法
ghci> (*) <$> [2,5,10] <*> [8,10,11]   
[16,20,22,40,50,55,80,100,110]

ghci> filter (>50) $ (*) <$> [2,5,10] <*> [8,10,11]   
[55,80,100,110]
~~~~~~

~~~~~~
instance Applicative IO where   
    pure = return   
    a <*> b = do   
        f <- a   
        x <- b   
        return (f x)

myAction :: IO String   
myAction = do   
    a <- getLine   
    b <- getLine   
    return $ a ++ b
-- 更加抽象了
myAction :: IO String   
myAction = (++) <$> getLine <*> getLine

main = do   
    a <- (++) <$> getLine <*> getLine   
    putStrLn $ "The two lines concatenated turn out to be: " ++ a
~~~~~~

~~~~~~
instance Applicative ((->) r) where
    -- 包装函数, 永远返回这个函数(或值)
    pure x = (\_ -> x)
    -- 这里有点不理解!!, 这里的 f 应该是 a -> a -> a
    f <*> g = \x -> f x (g x)

-- 当我们用 pure 将一个值包成 applicative functor 的时候，他产生的结果永远都会是那个值
ghci> (pure 3) "blah"
3

ghci> :t (+) <$> (+3) <*> (*100)   
(+) <$> (+3) <*> (*100) :: (Num a) => a -> a
-- 原理是左结合, 直觉可以理解为(5+3) + (5*100)
ghci> (+) <$> (+3) <*> (*100) $ 5   
508
ghci> (\x y z -> [x,y,z]) <$> (+3) <*> (*2) <*> (/2) $ 5   
[8.0,10.0,2.5]

-- ZipList用法
instance Applicative ZipList where   
        pure x = ZipList (repeat x)   
        ZipList fs <*> ZipList xs = ZipList (zipWith (\f x -> f x) fs xs)

-- 必须用 getZipList 函数来从 zip list 取出一个普通的 list
ghci> getZipList $ (+) <$> ZipList [1,2,3] <*> ZipList [100,100,100]   
[101,102,103]   
ghci> getZipList $ (+) <$> ZipList [1,2,3] <*> ZipList [100,100..]   
[101,102,103]   
ghci> getZipList $ max <$> ZipList [1,2,3,4,5,3] <*> ZipList [5,3,1,2]   
[5,3,3,4]   
ghci> getZipList $ (,,) <$> ZipList "dog" <*> ZipList "cat" <*> ZipList "rat"   
[('d','c','r'),('o','a','a'),('g','t','t')]
~~~~~~

除了 zipWith，标准函式库中也有 zipWith3, zipWith4 之类的函数，
最多支持到 7。
zipWith 接受一个接受两个参数的函数，
并把两个 list zip 起来。
zipWith3 则接受一个接受三个参数的函数，
然后把三个 list zip 起来。以此类推。

~~~~~~
-- exist in Control.Applicative
liftA2 :: (Applicative f) => (a -> b -> c) -> f a -> f b -> f c   
liftA2 f a b = f <$> a <*> b

ghci> liftA2 (:) (Just 3) (Just [4])   
Just [3,4]   
ghci> (:) <$> Just 3 <*> Just [4]   
Just [3,4]

sequenceA :: (Applicative f) => [f a] -> f [a]   
sequenceA [] = pure []   
sequenceA (x:xs) = (:) <$> x <*> sequenceA xs

ghci> sequenceA [Just 1, Just 2]
Just [1,2]

-- 也可以这样定义
sequenceA :: (Applicative f) => [f a] -> f [a]   
sequenceA = foldr (liftA2 (:)) (pure [])

ghci> sequenceA [Just 3, Just 2, Just 1]   
Just [3,2,1]   
ghci> sequenceA [Just 3, Nothing, Just 1]   
Nothing   
ghci> sequenceA [(+3),(+2),(+1)] 3   
[6,5,4]   
ghci> sequenceA [[1,2,3],[4,5,6]]   
[[1,4],[1,5],[1,6],[2,4],[2,5],[2,6],[3,4],[3,5],[3,6]]   
ghci> sequenceA [[1,2,3],[4,5,6],[3,4,4],[]]   
[]

ghci> sequenceA [(>4),(<10),odd] 7   
[True,True,True]   
ghci> and $ sequenceA [(>4),(<10),odd] 7   
True

ghci> sequenceA [[1,2,3],[4,5,6]]   
[[1,4],[1,5],[1,6],[2,4],[2,5],[2,6],[3,4],[3,5],[3,6]]   
ghci> [[x,y] | x <- [1,2,3], y <- [4,5,6]]   
[[1,4],[1,5],[1,6],[2,4],[2,5],[2,6],[3,4],[3,5],[3,6]]   
ghci> sequenceA [[1,2],[3,4]]   
[[1,3],[1,4],[2,3],[2,4]]   
ghci> [[x,y] | x <- [1,2], y <- [3,4]]   
[[1,3],[1,4],[2,3],[2,4]]   
ghci> sequenceA [[1,2],[3,4],[5,6]]   
[[1,3,5],[1,3,6],[1,4,5],[1,4,6],[2,3,5],[2,3,6],[2,4,5],[2,4,6]]   
ghci> [[x,y,z] | x <- [1,2], y <- [3,4], z <- [5,6]]   
[[1,3,5],[1,3,6],[1,4,5],[1,4,6],[2,3,5],[2,3,6],[2,4,5],[2,4,6]]

-- 当使用在 I/O action 上的时候，sequenceA 跟 sequence 是等价的
ghci> sequenceA [getLine, getLine, getLine]   
heyh   
ho   
woo   
["heyh","ho","woo"]
~~~~~~


applicative functor 应该遵守如下定律

    pure id <*> v = v
    pure (.) <*> u <*> v <*> w = u <*> (v <*> w)
    pure f <*> pure x = pure (f x)
    u <*> pure y = pure ($ y) <*> u


# 关键字 newtype #

~~~~~~
data ZipList a = ZipList [a]
data ZipList a = ZipList { getZipList :: [a] }
-- 为了可以调用 getZipList $ ZipList [(+1),(*100),(*5)] <*> ZipList [1,2,3]  

newtype ZipList a = ZipList { getZipList :: [a] }
-- newtype 比较快速, 用data解开来需要成本.
-- 用 newtype 的话，Haskell 会知道只是要将一个现有的类型包成一个新的类型
-- 内部运作完全一样但只是要一个全新的类型而已。
~~~~~~
newtype 来制作一个新的类型时，
只能定义单一一个值构造子，
而且那个构造子只能有一个字段

使用 data 的话，可以让那个类型有好几个值构造子，
并且每个构造子可以有零个或多个字段
~~~~~~
data Profession = Fighter | Archer | Accountant   
   
data Race = Human | Elf | Orc | Goblin

newtype CharList = CharList { getCharList :: [Char] } deriving (Eq, Show)
~~~~~~

newtype 的 lazy 性
~~~~~~
data CoolBool = CoolBool { getCoolBool :: Bool }
helloMe :: CoolBool -> String   
helloMe (CoolBool _) = "hello"

-- 不能运行
ghci> helloMe undefined   
"*** Exception: Prelude.undefined  "

-- 使用 newtype 时，Haskell 内部可以将新的类型用旧的类型来表示
-- Haskell 会知道 newtype 定义的类型一定只会有一个构造子，
-- 不必计算喂给函数的值就能确定他是 (CoolBool _) 的形式，
-- 因为 newtype 只有一个可能的值跟单一字段
newtype CoolBool = CoolBool { getCoolBool :: Bool }
ghci> helloMe undefined   
"hello"
~~~~~~

对 newtype 做模式匹配并不是像从盒子中取出东西，
他比较像是将一个类型转换成另一个类型。

## type vs newtype vs data ##

`type` 关键字是让我们定义 type synonyms。
他代表我们只是要给一个现有的类型另一个名字
~~~~~~
type IntList = [Int]

ghci> ([1,2,3] :: IntList) ++ ([1,2,3] :: [Int])   
[1,2,3,1,2,3]
~~~~~~

`newtype` 关键字将现有的类型包成一个新的类型，
大部分是为了要让他们可以是特定 typeclass 的 instance 而这样做.
使用 `newtype` 来包裹一个现有的类型时，这个类型跟原有的类型是分开的
~~~~~~
newtype CharList = CharList { getCharList :: [Char] }
~~~~~~
可以将 `newtype` 想成是只能定义一个构造子跟一个字段的 data 宣告

使用 `data` 关键字是为了定义自己的类型。

如果你只是希望你的 type signature 看起来比较干净，
你可以只需要 `type synonym`。
如果你想要将现有的类型包起来并定义成一个 `type class` 的 instance，
你可以尝试使用 `newtype`。
如果你想要定义完全新的类型，那你应该使用 data 关键字

# Monoids #

一个 monoid 是你有一个遵守结合律的二元函数还有
一个可以相对于那个函数作为 identity 的值。

monoid 可以是一个链式的抽取规则, 可以看 Ordering monoid
~~~~~~
-- import Data.Monoid
class Monoid m where   
    mempty :: m   
    mappend :: m -> m -> m   
    mconcat :: [m] -> m   
    mconcat = foldr mappend mempty
~~~~~~

monoid 必须遵守结合律和交换律

    mempty `mappend` x = x
    x `mappend` mempty = x
    (x `mappend` y) `mappend` z = x `mappend` (y `mappend` z)

## List Monoid ##
~~~~~~
instance Monoid [a] where   
    mempty = []   
    mappend = (++)

ghci> [1,2,3] `mappend` [4,5,6]   
[1,2,3,4,5,6]   
ghci> ("one" `mappend` "two") `mappend` "tree"   
"onetwotree"
ghci> mconcat [[1,2],[3,6],[9]]   
[1,2,3,6,9]   
ghci> mempty :: [a]   
[]

-- 乘法的Monoid实现
newtype Product a =  Product { getProduct :: a }   
    deriving (Eq, Ord, Read, Show, Bounded)

instance Num a => Monoid (Product a) where   
    mempty = Product 1   
    Product x `mappend` Product y = Product (x * y)

-- 或的Monoid实现
newtype Any = Any { getAny :: Bool }   
    deriving (Eq, Ord, Read, Show, Bounded)

instance Monoid Any where   
    mempty = Any False   
    Any x `mappend` Any y = Any (x || y)

-- 与的Monoid实现
newtype All = All { getAll :: Bool }   
        deriving (Eq, Ord, Read, Show, Bounded)
        
instance Monoid All where   
        mempty = All True   
        All x `mappend` All y = All (x && y)


-- Ordering monoid
instance Monoid Ordering where   
    mempty = EQ   
    LT `mappend` _ = LT   
    EQ `mappend` y = y   
    GT `mappend` _ = GT

-- 使用
lengthCompare :: String -> String -> Ordering   
lengthCompare x y = let a = length x `compare` length y    
                        b = x `compare` y   
                    in  if a == EQ then b else a

import Data.Monoid
lengthCompare :: String -> String -> Ordering
lengthCompare x y = (length x `compare` length y) `mappend`
                    (x `compare` y)
~~~~~~

## Maybe monoid ##

~~~~~~
instance Monoid a => Monoid (Maybe a) where   
    mempty = Nothing   
    Nothing `mappend` m = m   
    m `mappend` Nothing = m   
    Just m1 `mappend` Just m2 = Just (m1 `mappend` m2)

ghci> Nothing `mappend` Just "andy"   
Just "andy"   
ghci> Just LT `mappend` Nothing   
Just LT   
ghci> Just (Sum 3) `mappend` Just (Sum 4)   
Just (Sum {getSum = 7})


instance Monoid (First a) where   
    mempty = First Nothing   
    First (Just x) `mappend` _ = First (Just x)   
    First Nothing `mappend` x = x

ghci> getFirst $ First (Just 'a') `mappend` First (Just 'b')   
Just 'a'   
ghci> getFirst $ First Nothing `mappend` First (Just 'b')   
Just 'b'   
ghci> getFirst $ First (Just 'a') `mappend` First Nothing   
Just 'a'
~~~~~~

## Using monoids to fold data structures ##
`import qualified Foldable as F`

`foldr` 接受一个 list 并将他 fold 起来，
`Data.Foldable` 中的 foldr 接受任何可以 fold 的类型。并不只是 list。
~~~~~~
-- 结果相同
ghci> foldr (*) 1 [1,2,3]   
6   
ghci> F.foldr (*) 1 [1,2,3]   
6

ghci> F.foldl (+) 2 (Just 9)   
11   
ghci> F.foldr (||) False (Just True)   
True
~~~~~~

简化整棵树到只有单一一个 monoid
~~~~~~

-- 定义 foldMap :: (Monoid m, Foldable t) => (a -> m) -> t a -> m
data Tree a = Empty | Node a (Tree a) (Tree a) deriving (Show, Read, Eq)
instance F.Foldable Tree where   
    foldMap f Empty = mempty   
    foldMap f (Node x l r) = F.foldMap f l `mappend`   
                                f x           `mappend`   
                                F.foldMap f r

testTree = Node 5   
            (Node 3   
             (Node 1 Empty Empty)   
             (Node 6 Empty Empty)   
            )   
            (Node 9   
             (Node 8 Empty Empty)   
             (Node 10 Empty Empty)   
            )

ghci> F.foldl (+) 0 testTree   
42   
ghci> F.foldl (*) 1 testTree   
64800

-- foldMap 对简化结构至单一 monoid 值有用
-- 树中有没有 3
ghci> getAny $ F.foldMap (\x -> Any $ x == 3) testTree   
True
-- 树中有没有大于15的数
ghci> getAny $ F.foldMap (\x -> Any $ x > 15) testTree   
False
~~~~~~
