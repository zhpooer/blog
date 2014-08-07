title: haskell趣学-构造Type和Typeclass
date: 2014-06-29 15:02:14
tags:
- haskell
---

使用`data`关键字定义自己的类型 `data Bool = False | True`

~~~~~~
data Shape = Circle Float Float Float | Retangle Float Float Float

-- 值构造子的本质是个函数，可以返回一个类型的值
-- class Circle(x:Float, y:Float, radius:Float), 没有变量名吗?
ghci> :t Circle 
Circle :: Float -> Float -> Float -> Shape 
ghci> :t Rectangle 
Rectangle :: Float -> Float -> Float -> Float -> Shape

surface :: Shape -> Float 
surface (Circle _ _ r) = pi * r ^ 2 
surface (Rectangle x1 y1 x2 y2) = (abs $ x2 - x1) * (abs $ y2 - y1)


-- Shape类型成为 Show 类型类的成员
data Shape = Circle Float Float Float | Retangle Float Float Float Float deriving (Show)

-- 获取一组不同半径的同心圆
map (Circle 10 20) [4,5,6,6]

-- 增加加一个表示二维空间中点的类型，可以让我们的 Shape 更加容易理解
data Point = Point Float Float deriving (Show)
data Shape = Circle Point Float | Retangle Point Point deriving (Show)

surface :: Shape -> Float
surface (Circle _ r) = pi * r^2
surface (Retangle (Point x1 y1) (Point x2 y2)) = (abs $ x2 - x1) * (abs $ y2 - y1)

~~~~~~

# 类型记录 #

~~~~~~
data Person = Person {
      firstName :: String
    , lastName :: String
    , age :: Int
    , height :: Float
    , phoneNumber :: String
    , flavor :: String
} deriving (Show)

ghci> :t flavor 
flavor :: Person -> String 
ghci> :t firstName 
firstName :: Person -> String
~~~~~~

# 类型参数 #

~~~~~~
-- a 相当于 泛型
data Maybe a = Nothing | Just a

ghci> Just "Haha" 
Just "Haha" 
ghci> Just 84 
Just 84 
ghci> :t Just "Haha" 
Just "Haha" :: Maybe [Char] 
ghci> :t Just 84 
Just 84 :: (Num t) => Maybe t 
ghci> :t Nothing 
Nothing :: Maybe a 
~~~~~~

## Car的案例 ##
~~~~~~
data Car = Car { company :: String 
                 , model :: String 
                 , year :: Int 
                 } deriving (Show)
-- 也可以改为这样, 但是可能没有用处
data Car a b c = Car { company :: a 
                       , model :: b 
                       , year :: c 
                        } deriving (Show)

tellCar :: Car -> String 
tellCar (Car {company = c, model = m, year = y}) = "This " ++ c ++ " " ++ m ++ " was made in " ++ show y
~~~~~~

## Vector(三维矢量) ##
~~~~~~
data Vecotr a = Vector a a a deriving (Show)
vplus :: (Num t) => Vector t -> Vector t -> Vector t
(Vector i j k) `vplus` (Vector l m n) = Vector (i+l) (j+m) (k+n)
~~~~~~

# Derived instances #

`TypeClass`, 在 Java, C++ 等语言中, 类就像是蓝图,
我们可以根据它来创造对象保存对象并执行操作.
`TypeClass` 更像是接口, 我们不是靠它构造数据，而是给既有的数据类型描述行为.

什么东西若可以判定相等性，我们就可以让它成为 Eq 类型类的 instance。什么东西若可以比较大小，
那就可以让它成为 Ord 类型类的 instance。

在一个类型 derive 为 Eq 的 instance 后，
就可以直接使用 == 或 /= 来判断它们的相等性了。
Haskell 会先看下这两个值的值构造子是否一致(这里只是单值构造子)，
再用 == 来检查其中的所有数据(必须都是 Eq 的成员)是否一致。
~~~~~~
data Person = Person { firstName :: String 
                     , lastName :: String 
                     , age :: Int 
                     } deriving (Eq, Show, Read)

ghci> read "Person {firstName =\"Michael\", lastName =\"Diamond\", age = 43}" :: Person
-- 如果我们 read 的结果会在后面用到参与计算，
-- Haskell 就可以推导出是一个 Person 的行为，不加注释也是可以的
ghci> read "Person {firstName =\"Michael\", lastName =\"Diamond\", age = 43}" == mikeD
True

ghci> Nothing 
True 
ghci> Nothing > Just (-49999) 
False 
ghci> Just 100 > Just 50 
True

-- 使用 Enmu 和 Bounded 类型类, 做枚举
data Day = Monday | Tuesday | Wednesday | Thursday | Friday | Saturday | Sunday
           deriving (Eq, Ord, Show, Read, Bounded, Enum)

ghci> minBound :: Day 
Monday 
ghci> maxBound :: Day 
Sunday
~~~~~~

# 类型别名(Type synonyms) #
~~~~~~
type String = [Char]
toUpperString :: [Char] -> [Char]
-- 等同于
toUpperString :: String -> String

type PhoneNumber = String 
type Name = String 
type PhoneBook = [(Name,PhoneNumber)]
-- 可以这样使用, 更加易读
inPhoneBook :: Name -> PhoneNumber -> PhoneBook -> Bool 
inPhoneBook name pnumber pbook = (name,pnumber) `elem` pbook

-- 类型别名的参数, 相当于泛型(类型构造子)
type AssocList k v = [(k, v)]

AssocList [(1,2),(4,5),(7,9)]   -- 不能这么用
[(1,2),(4,5),(7,9)] :: AssocList Int Int  -- 正确使用方法

-- 他有两个值构造子 Left 和 Right, 类型构造子是 Either
data Either a b = Left a | Right b deriving (Eq, Ord, Read, Show)
~~~~~~
## Either 的使用 ##
学生向管理员索取储物箱, 保存物品, 每个储物箱都要密码.
如果学生想要用, 就要告诉管理员储物柜的号码.
如果已经被别人用了, 管理员就不能告诉它密码, 得换一个储物箱.

~~~~~~
import qualified Data.Map as Map

data LockerState = Taken | Free deriving (Show, Eq)
type Code = String
type LockerMap = Map.Map Int (LockerState, Code)

lockerLookup :: Int -> LockerMap -> Either String Code
lockerLookup lockerNumber map =
     case Map.lookup lockerNumber map of
         Nothing -> Left $ "Locker number" ++ show lockerNumber ++ "doesn't exist!"
         Just (state, code) -> if state /= Taken
                                then Right code
                                else Left $ "Locker" ++ show lockerNumber ++ " is already taken"

-- 我们完全可以用 Maybe a 来表示它的结果，但这样一来我们就对得不到密码的原因不得而知了。
-- 而在这里，我们的新类型可以告诉我们失败的原因。
~~~~~~

# 递归地定义数据结构 #

~~~~~~
-- cons 其实就是指 `:`
ghci> Empty 
Empty 
ghci> 5 `Cons` Empty 
Cons 5 Empty 
ghci> 4 `Cons` (5 `Cons` Empty) 
Cons 4 (Cons 5 Empty) 
ghci> 3 `Cons` (4 `Cons` (5 `Cons` Empty)) 
Cons 3 (Cons 4 (Cons 5 Empty))

-- 定义List
data List a = Empty | Cons a (List a) deriving (Show, Read, Eq, Ord)
~~~~~~

`infixr`, `infixl` 优先级设置(左结合, 右结合)
~~~~~~
-- 代表左结合
-- (4 * 3 * 2) 等于 ((4 * 3) * 2)。
-- * 拥有比 + 更高的优先级。所以 5 * 4 + 3 会是 (5 * 4) + 3。
infixl 7 *
infixl 6 +
~~~~~~

## 二元搜索树构建 ##
~~~~~~
data Tree a = EmptyTree | Node a (Tree a) (Tree a) deriving (Show, Read, Eq)

-- 定义单节点树
single :: a -> Tree a
singleton x = Node x EmptyTree EmptyTree
-- 插入
treeInsert :: (Ord a) => a -> Tree a -> Tree a
treeInsert x EmptyTree = singleton x
treeInsert x (Node x left right)
    | x == a = Node x left right
    | x < a = Node a (treeInsert x left) right
    | x > a = Node a left (treeInsert x right)

-- 查找
treeElem :: (Ord a) => a -> Tree a -> Bool 
treeElem x EmptyTree = False 
treeElem x (Node a left right) 
    | x == a = True 
    | x < a  = treeElem x left 
    | x > a  = treeElem x right
~~~~~~

# TypeClass #

ypeclass 就像是 interface。
一个 typeclass 定义了一些行为(像是比较相不相等，
比较大小顺序，能否穷举)
而我们会把希望满足这些性质的类型定义成这些 typeclass 的 instance。
typeclass 的行为是由定义的函数来描述。并写出对应的实作。
当我们把一个类型定义成某个 typeclass 的 instance，
就表示我们可以对那个类型使用 typeclass 中定义的函数。

~~~~~~
-- Prelude 中 Eq 如何被定义
class Eq a where
   (==) :: a -> a -> Bool
   (/=) :: a -> a -> Bool
   x == y = not (x /= y)
   x /= y = not (x == y)


data TrafficLight = Red | Yellow | Green
-- 可以透过 derive 让他成为 Eq 的instance
-- 手动编写

instance Eq TrafficLight where
    Red == Red = True
    Green == Green = True
    Yellow == Yellow = True
    _ == _ = False

instance Show TrafficLight where 
    show Red = "Red light" 
    show Yellow = "Yellow light" 
    show Green = "Green light"

-- 类型约束, 定义一个类型为 Num 之前，必须先为他定义 Eq 的 instance
class (Eq a) => Num a where

-- instance Eq Maybe where, 错误用法
-- 是不是可以理解为类型 (Maybe a) 才是一个完整的类型
instance Eq (Maybe m) where 
    Just x == Just y = x == y 
    Nothing == Nothing = True 
    _ == _ = False

-- 最终极版本
instance (Eq m) => Eq (Maybe m) where
    Just x == Just y = x == y
    Nothing == Nothing = True
    _ == _ False
~~~~~~

`:info` 也可以查询类型跟类型构造子的信息

# yes-no typeclass #

~~~~~~
class YesNo a where
    yesno :: a -> Bool

instance YesNo Int where 
    yesno 0 = False 
    yesno _ = True

instance YesNo [a] where
    yesno [] = False
    yesno _ = True

-- 标准函式库中的一个函数，他接受一个参数并回传相同的东西
instance YesNo Bool where
    yesno = id

yesnoIf :: (YesNo y) => y -> a -> a -> a
yesnoIf yesnoVal yesResult noResult =
    if yesno yesnoVal then yesResult else noResult

ghci> yesnoIf [] "YEAH!" "NO!" 
"NO!" 
ghci> yesnoIf [2,3,4] "YEAH!" "NO!" 
"YEAH!" 
ghci> yesnoIf True "YEAH!" "NO!" 
"YEAH!" 
ghci> yesnoIf (Just 500) "YEAH!" "NO!" 
"YEAH!" 
ghci> yesnoIf Nothing "YEAH!" "NO!" 
"NO!"
~~~~~~


# Functor typeclass #
`Functor` 这个 typeclass，基本上就代表可以被 map over 的事物

~~~~~~
-- f 代表一个类型构造子 Maybe(不是 Maybe a), 不是具体类型
-- 可以当作盒子的类型可能就是一个 functor, 如 List, Maybe
-- 实例化后: (a -> b) -> Tree a -> Tree b
-- (a -> b) -> Maybe a -> Maybe b
class Functor f where
    fmap :: (a -> b) -> f a -> f b

-- map :: (a -> ) -> [a] -> [b]
-- 注意这里没有写 [a]
instance Functor [] where
    fmap = map
    
-- 注意 这边没有写 Maybe a
instance Functor Maybe where 
    fmap f (Just x) = Just (f x) 
    fmap f Nothing = Nothing
~~~~~~

# Kind #

类型构造子接受其他类型作为他的参数，来构造出一个具体类型.
我们可以在 ghci 中用 `:k` 来得知一个类型的 kind。
~~~~~~
-- 一个 * 代表这个类型是具体类型
ghci> :k Int 
Int :: *
-- * -> * 代表这个类型构造子接受一个具体类型并回传一个具体类型
ghci> :k Maybe 
Maybe :: * -> *
ghci> :k Maybe Int 
Maybe Int :: *

-- f 必须是 * -> * kind
class Functor f where 
    fmap :: (a -> b) -> f a -> f b
    
-- a 是 *
-- j 是 * -> * 
-- t 是 * -> (* -> *) -> *
class Tofu t where 
    tofu :: j a -> t a j

-- a 是 *
-- b 是 * -> *
data Frank a b  = Frank {frankField :: b a} deriving (Show)

-- Barry 是 (* -> *) -> * -> * -> *
data Barry t k p = Barry { yabba :: p, dabba :: t k }

instance Functor (Barry a b) where
    fmap :: (a -> b) -> Barry c d a -> Barry c d b

instance Functor (Barry a b) where
    fmap f (Barry {yabba = x, dabba =})
~~~~~~
