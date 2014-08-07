title: haskell趣学-输入与输出
date: 2014-06-30 15:12:13
tags:
- haskell
---

# HelloWorld #

函数式语言, 一个函数不能改变状态(改变变量的内容),
当一个函数会改变状态,  我们说这函数是由副作用的.
如果我们用同样的参数调用两次同一个函数，它会回传相同的结果.

~~~~~~
main = putStrLn "hello, world"

> ghc --make helloworld
> ./helloworld

-- putStrLn 接受一个字符串并回传一个I/O action,
-- 这I/O action包含了() 形态
-- 一个 I/O action 是一个会造成副作用的动作，
-- 常是指读取输入或输出到屏幕，同时也代表会回传某些值
-- 一个 I/O action 会在我们把它绑定到 main
-- 这个名字并且执行程序的时候触发。
ghci> :t putStrLn 
putStrLn :: String -> IO () 
ghci> :t putStrLn "hello, world" 
putStrLn "hello, world" :: IO ()

-- 用 do 并且接着一连串指令, 每一步都是一个 I/O action
-- 所有的I/O action 用 do 绑定在一起变成了一个大的I/O action
-- main 的型态永远都是 main :: IO something
main = do 
    putStrLn "Hello, what's your name?" 
    name <- getLine 
    putStrLn ("Hey " ++ name ++ ", you rock!")

-- main函数中执行一个 I/O action getLine 并将它的结果绑定到 name 这个名字
ghci> :t getLine 
getLine :: IO String
~~~~~~

I/O action 就是一个盒子, 它会和真实世界交互(副作用),
一旦它带了某些数据给你，打开盒子的唯一办法就是用 `<-`.
而且如果我们要从 I/O action 拿出某些数据，
就一定同时要在另一个 I/O action 中

getLine 在这样的意义下是不纯粹的,
因为执行两次的时候它没办法保证会回传一样的值,
所以它需要在一个`IO`的形态构建子中,
任何一段程序一旦依赖着 I/O 数据的话，
那段程序也会被视为 I/O code

~~~~~~
-- 错误用法, getLine 是I/O Action
nameTag = "Hello, my name is " ++ getLine

-- 可以用 do 来串接 I/O actions，
-- 再用 do 来串接这些串接起来的 I/O actions。
-- 不过只有最外面的 I/O action 被指定给 main 才会触发执行。
main = do
    _ <- putStrLn "hello, what's your name?"
    name <- getLine
    putStrLn("Hey" ++ name ++ ", you rock")
~~~~~~

## let binding ##
~~~~~~
main = do 
    putStrLn "What's your first name?" 
    firstName <- getLine 
    putStrLn "What's your last name?" 
    lastName <- getLine 
    let bigFirstName = map toUpper firstName 
        bigLastName = map toUpper lastName 
    putStrLn $ "hey " ++ bigFirstName ++ " " ++ bigLastName ++ ", how are you?"
~~~~~~


~~~~~~
-- return "haha" 的型态是 IO String
main = do 
    line <- getLine 
    if null line 
        then return () 
        else do 
            putStrLn $ reverseWords line 
            main  -- 递归调用了main
            
-- reverseWords st = unwords (map reverse (words st)) 
reverseWords :: String -> String 
reverseWords = unwords . map reverse . words

-- return 与 <- 作用相反。return 把 value 装进盒子中，
-- 而 <- 将 value 从盒子拿出来，并绑定一个名称
main = do 
    a <- return "hell" 
    b <- return "yeah!" 
    putStrLn $ a ++ " " ++ b

-- 另一个版本
main = do 
    let a = "hell" 
        b = "yeah" 
    putStrLn $ a ++ " " ++ b

main = do print True
          putChar 'a'
          c <- getChar
~~~~~~

## io 语法糖 ##

~~~~~~
main = do 
    c <- getChar 
    if c /= ' ' 
        then do 
            putChar c 
            main 
        else return ()

-- when :: Monad m => Bool -> m () -> m ()
import Control.Monad 
main = do 
    c <- getChar 
    when (c /= ' ') $ do 
        putChar c 
        main
        
-- sequence :: Monad m => [m a] -> m [a]
main = do 
    rs <- sequence [getLine, getLine, getLine] 
    print rs

ghci> sequence (map print [1,2,3,4,5])  sequence (map print [1,2,3,4,5])
-- 同上
ghci> mapM print [1,2,3]     
1 
2 
3 
[(),(),()]
-- 同上, 把结果丢掉
ghci> mapM_ print [1,2,3] 
1 
2 
3

-- forever :: Monad m => m a -> m b, 不断的要用户输入东西
import Control.Monad 
import Data.Char 
 
main = forever $ do 
    putStr "Give me some input: " 
    l <- getLine 
    putStrLn $ map toUpper l

-- forM
import Control.Monad 
main = do  
    colors <- forM [1,2,3,4] (\a -> do 
        putStrLn $ "Which color do you associate with the number " ++ show a ++ "?" 
        color <- getLine 
        return color) 
    putStrLn "The colors that you associate with 1, 2, 3 and 4 are: " 
    mapM putStrLn colors
~~~~~~

# 文件和字符流 #

~~~~~~
-- cat haiku.txt | ./capslocker
import Control.Monad
import Data.Char
main = forever $ do
    putStr "Give me some input: "
    l <- getLine
    putStrLn $ map toUpper l

-- 用 getContents 简单
-- getContents 也是惰性 I/O, 会一行一行读入并输出大写的版本
import Data.Char
main = do
    contents <- getContents
    putStr (map toUpper contents)
    
-- 运行效果
-- > $ ./capslocker 
-- > hey ho 
-- > HEY HO 
-- > lets go 
-- > LETS GO

main = do 
    contents <- getContents 
    putStr (shortLinesOnly contents) 
shortLinesOnly :: String -> String 
shortLinesOnly input =  
    let allLines = lines input 
        shortLines = filter (\line -> length line < 10) allLines 
        result = unlines shortLines 
    in result
~~~~~~

## interact ##

~~~~~~
-- interact :: (String -> String) -> IO ()
-- 从用户获取一行一行的输入，
-- 然后丢回根据那一行运算的结果，再拿取另一行
-- 这是基于 haskell 的惰性i/o
main = interact shortLinesOnly 
 
shortLinesOnly :: String -> String 
shortLinesOnly input =  
    let allLines = lines input 
        shortLines = filter (\line -> length line < 10) allLines 
        result = unlines shortLines 
    in result

-- 一行搞定
main = interact $ unlines . filter ((<10) . length) . lines
~~~~~~

## 文件读写 ##
~~~~~~
main = do    
    withFile "something.txt" ReadMode (\handle -> do   
        hSetBuffering handle $ BlockBuffering (Just 2048)   
        contents <- hGetContents handle   
        putStr contents)

import System.IO   
import System.Directory   
import Data.List   
   
main = do         
    handle <- openFile "todo.txt" ReadMode   
    (tempName, tempHandle) <- openTempFile "." "temp"   
    contents <- hGetContents handle   
    let todoTasks = lines contents      
    numberedTasks = zipWith (\n line -> show n ++ " - " ++ line) [0..] todoTasks      
    putStrLn "These are your TO-DO items:"   
    putStr $ unlines numberedTasks   
    putStrLn "Which one do you want to delete?"      
    numberString <- getLine      
    let number = read numberString      
    newTodoItems = delete (todoTasks !! number) todoTasks      
    hPutStr tempHandle $ unlines newTodoItems   
    hClose handle   
    hClose tempHandle   
    removeFile "todo.txt"   
    renameFile tempName "todo.txt"
~~~~~~

# 命令行引数 #

~~~~~~
import System.Environment    
import System.Directory   
import System.IO   
import Data.List   
   
dispatch :: [(String, [String] -> IO ())]   
dispatch =  [ ("add", add)   
            , ("view", view)   
            , ("remove", remove)   
            ]

main = do   
    (command:args) <- getArgs   
    let (Just action) = lookup command dispatch   
    action args V

add :: [String] -> IO ()   
add [fileName, todoItem] = appendFile fileName (todoItem ++ "\n")

view :: [String] -> IO ()   
view [fileName] = do   
    contents <- readFile fileName   
    let todoTasks = lines contents   
    numberedTasks = zipWith (\n line -> show n ++ " - " ++ line) [0..] todoTasks   
    putStr $ unlines numberedTasks

remove :: [String] -> IO ()   
remove [fileName, numberString] = do   
    handle <- openFile fileName ReadMode   
    (tempName, tempHandle) <- openTempFile "." "temp"   
    contents <- hGetContents handle   
    let number = read numberString   
        todoTasks = lines contents   
        newTodoItems = delete (todoTasks !! number) todoTasks   
    hPutStr tempHandle $ unlines newTodoItems   
    hClose handle   
    hClose tempHandle   
    removeFile fileName   
    renameFile tempName fileName
~~~~~~

# 乱数 #

Haskell 是一个纯粹的函数式语言, 是引用透明的.
那代表你喂给一个函数相同的参数，不管怎么调用都是回传相同的结果.

所以说在 Haskell 中，假如我们能作一个函数，
他会接受一个具随机性的参数，然后根据那些信息还传一个数值。

~~~~~~
import System.Random
-- RandomGen typeclass 是指那些可以当作乱源的型态。
-- 而Random typeclass 则是可以装乱数的型态
random :: (RandomGen g, Random a) => g -> (a, g)
~~~~~~
在 System.Random 中有一个很酷的型态，叫做 StdGen，
他是 RandomGen 的一个 instance。
我们可以自己手动作一个 StdGen 也可以告诉系统给我们一个现成的。

~~~~~~
ghci> random (mkStdGen 949488) :: (Float, StdGen)   
(0.8938442,1597344447 1655838864)   
ghci> random (mkStdGen 949488) :: (Bool, StdGen)   
(False,1485632275 40692)   
ghci> random (mkStdGen 949488) :: (Integer, StdGen)   
(1691547873,1597344447 1655838864)

threeCoins :: StdGen -> (Bool, Bool, Bool)   
threeCoins gen =    
    let (firstCoin, newGen) = random gen   
    (secondCoin, newGen') = random newGen   
    (thirdCoin, newGen') = random newGen'   
    in (firstCoin, secondCoin, thirdCoin))

-- 返回n个随机数
ghci> take 5 $ randoms (mkStdGen 11) :: [Int]   
[-1807975507,545074951,-1015194702,-1622477312,-502893664]   
ghci> take 5 $ randoms (mkStdGen 11) :: [Bool]   
[True,True,True,True,False]   
ghci> take 5 $ randoms (mkStdGen 11) :: [Float]   
[7.904789e-2,0.62691015,0.26363158,0.12223756,0.38291094]

-- 返回一定范围内的乱数
ghci> randomR (1,6) (mkStdGen 359353)   
(6,1494289578 40692)   
ghci> randomR (1,6) (mkStdGen 35935335)   
(3,1250031057 40692)

-- 他会产生一连串在给定范围内的乱数
ghci> take 10 $ randomRs ('a','z') (mkStdGen 3) :: [Char]
~~~~~~
以上, 程序永远都会回传同样的乱数, `getStdGen` 会生成一个真实的随机数
~~~~~~
import System.Random
-- getStdGen 的 类型是 IO StdGen
main = do   
    gen <- getStdGen   
    putStr $ take 20 (randomRs ('a','z') gen)

-- 连续两次调用, 实际上会回传同样的 global generator, 可以使用 newStdGen
main = do   
    gen <- getStdGen   
    putStrLn $ take 20 (randomRs ('a','z') gen)   
    gen2 <- getStdGen   
    putStr $ take 20 (randomRs ('a','z') gen2)

-- newStdGen 用法
main = do      
    gen <- getStdGen      
    putStrLn $ take 20 (randomRs ('a','z') gen)      
    gen' <- newStdGen   
    putStr $ take 20 (randomRs ('a','z') gen')
~~~~~~

# Bytstrings #

List 是惰性的, 在读取大文件的时候会出现性能问题.
所以就有了 `bytestrings`, 他的每一个元素都是一个 byte(8 bits),
而且分为不同程度的惰性: strict 和 lazy.

strict: 放在 `Data.ByteString`, 完全没有惰性, 一个
strict bytesstring 代表一连串 bytes.

lazy: 放在 `Data.ByteString.Lazy`, 就有惰性, 但是没有 List 极端.
他是一个 chunk(64k) 一个 chunk 地去读取, 不会使用大量的记忆体.

lazy 和 strict 名字一样, 所以可以这么使用
~~~~~~
-- lazy
import qualified Data.ByteString.Lazy as B
-- strict
import qualified Data.ByteString as S


-- pack :: [Word8] -> ByteString
ghci> B.pack [99,97,110]   
Chunk "can" Empty   
ghci> B.pack [98..120]   
Chunk "bcdefghijklmnopqrstuvwx" Empty

-- unpack 是 pack 的相反，他把一个 bytestring 变成一个 byte list。
-- fromChunks 接受一串 strict 的 bytestrings 并把他变成一串 lazy bytestring。
-- toChunks 接受一个 lazy bytestrings 并将他变成一串 strict bytestrings。
ghci> B.fromChunks [S.pack [40,41,42], S.pack [43,44,45], S.pack [46,47,48]]   
Chunk "()*" (Chunk "+,-" (Chunk "./0" Empty))

-- cons 是 lazy 的操作，即使 bytestring 的第一个 chunk 不是满的，他也会新增一个 chunk
-- 当你要插入很多 bytes 可以使用 strict 版本的 cons，也就是 cons'
ghci> B.cons 85 $ B.pack [80,81,82,84]   
Chunk "U" (Chunk "PQRT" Empty)   
ghci> B.cons' 85 $ B.pack [80,81,82,84]   
Chunk "UPQRT" Empty   
ghci> foldr B.cons B.empty [50..60]   
Chunk "2" (Chunk "3" (Chunk "4" (Chunk "5" (Chunk "6" (Chunk "7" (Chunk "8" (Chunk "9" (Chunk ":" (Chunk ";" (Chunk "<"   
Empty))))))))))   
ghci> foldr B.cons' B.empty [50..60]   
Chunk "23456789:;<" Empty
~~~~~~

## ReadFile ##
如果你用了 strict bytestring 来读取一个文件，
他会把文件内容都读进记忆体中。
而使用 lazy bytestring，他则会读取 chunks。
~~~~~~
import System.Environment   
import qualified Data.ByteString.Lazy as B   
   
main = do   
    (fileName1:fileName2:_) <- getArgs   
    copyFile fileName1 fileName2   
 
copyFile :: FilePath -> FilePath -> IO ()   
copyFile source dest = do   
    contents <- B.readFile source   
    B.writeFile dest contents
~~~~~~

# 异常 #

~~~~~~
ghci> 4 `div` 0   
*** Exception: divide by zero   
ghci> head []   
*** Exception: Prelude.head: empty list
~~~~~~
pure code 能丢出 Exception，但 Exception 只能在 I/O section 中被接到
（也就是在 main 的 do block 中）
这是因为在 pure code 中你不知道什么东西什么时候会被 evaluate。
因为 lazy 特性的缘故，程序没有一个特定的执行顺序，但 I/O code 有。

~~~~~~
import System.Environment   
import System.IO   
import System.IO.Error   
   
main = toTry `catch` handler   
 
toTry :: IO ()   
toTry = do (fileName:_) <- getArgs   
            contents <- readFile fileName   
            putStrLn $ "The file has " ++ show (length (lines contents)) ++ " lines!"
-- 抛出用户错误            
-- ioError $ userError "remote computer unplugged!"
handler :: IOError -> IO ()   
handler e   
    | isDoesNotExistError e =
        case ioeGetFileName e of Just path -> putStrLn $ "Whoops! File does not exist at: " ++ path   
                                 Nothing -> putStrLn "Whoops! File does not exist at unknown location!" 
    | isFullError e = freeSomeSpace   
    | isIllegalOperation e = notifyCops   
    | otherwise = ioError e
~~~~~~

程序里面有好几个运作在 IOError 上的 I/O action，当其中一个没有被 evaluate 成 True 时，就会掉到下一个 guard。
这些 predicate 分别为：

    isAlreadyExistsError

    isDoesNotExistError

    isFullError

    isEOFError

    isIllegalOperation

    isPermissionError

    isUserError


~~~~~~
main = do toTry `catch` handler1   
          thenTryThis `catch` handler2   
          launchRockets  -- 其他处理函数
~~~~~~
