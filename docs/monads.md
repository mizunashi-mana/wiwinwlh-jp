# モナド

## モナド菩提への八正道

モナドの神秘性とされているものについて陶酔した人々が、多くのインクを費やしてきました。私はその代わりに、悟りへの道筋を提示します。

1. モナドのチュートリアルを読むな。
2. いや、本当に、モナドのチュートリアルを読むな。
3. Haskell の型について学べ。
4. 型クラスとは何なのかを学べ。
5. [Typeclassopedia](http://www.haskell.org/haskellwiki/Typeclassopedia) を読め。
6. モナドの定義を読め。
7. モナドを実際のコードで使え。
8. モナドを比喩で説明するチュートリアルを書くな。

言い換えると、モナドを理解する唯一の道は、良質なソースを読み、GHC を立ち上げてコードを書くことです。類推や比喩は理解につながりません。

## モナドについての神話

以下のことは全て**誤り**です。

* モナドは純粋でない。
* モナドは副作用に関するものである。
* モナドは状態に関するものである。
* モナドは命令型の順序付けに関するものである。
* モナドは IO に関するものである。
* モナドは遅延性に依存している。
* モナドは副作用を実行するために言語に備え付けられた”裏口”である。
* モナドは Haskell に埋め込まれた命令型言語である。
* モナドを使うには抽象的な数学を知っていなくてはならない。

参照：

* [What a Monad Is Not](http://www.haskell.org/haskellwiki/What_a_Monad_is_not)

## 法則

モナドは複雑ではありません。モナドの実装は 2 つの関数 ``(>>=)``（"bind"［束縛］と発音される）と ``return``［返す］からなる型クラスです。"return" という単語に対してあなたが持っている先入観は全て捨てるべきです。ここでは全く違う意味で使われているのです。

```haskell
class Monad m where
  (>>=)  :: m a -> (a -> m b) -> m b
  return :: a -> m a
```

加えて、全てのモナドのインスタンスが満たすべき 3 つの法則があります。

**法則 1**

```haskell
return a >>= f ≡ f a
```

**法則 2**

```haskell
m >>= return ≡ m
```

**法則 3**

```haskell
(m >>= f) >>= g ≡ m >>= (\x -> f x >>= g)
```

束縛の操作については、引数を捨てる補助関数 ``(>>)`` が定義されています。

```haskell
(>>) :: Monad m => m a -> m b -> m b
m >> k = m >>= \_ -> k
```

参照：

* [Monad Laws](http://www.haskell.org/haskellwiki/Monad_laws)

## do記法

Haskell のモナド構文は、モナドの演算の単なる応用にちょうど相当する、糖衣化された形で書かれます。脱糖衣は以下の法則で再帰的に定義されます。

```haskell
do { a <- f ; m } ≡ f >>= \a -> do { m }
do { f ; m } ≡ f >> do { m }
do { m } ≡ m
```

例えば、以下のものは全て等価です。

```haskell
do
  a <- f
  b <- g
  c <- h
  return (a, b, c)

do {
  a <- f ;
  b <- g ;
  c <- h ;
  return (a, b, c)
}

f >>= \a ->
  g >>= \b ->
    h >>= \c ->
      return (a, b, c)
```

（Haskell の実際の流儀とは異なりますが）束縛演算子を脱カリー化された関数として書きたい場合、同じものを糖衣化すると以下のような、ラムダ式付きの束縛が入れ子になったものになるでしょう。

```haskell
bindMonad(f, lambda a:
  bindMonad(g, lambda b:
    bindMonad(h, lambda c:
      returnMonad (a,b,c))))
```

do 記法で上記のモナド則を書くと以下のようになります：

**法則 1**

```haskell
  do x <- m
     return x

= do m
```

**法則 2**

```haskell
  do y <- return x
     f y

= do f x
```

**法則 3**

```haskell
  do b <- do a <- m
             f a
     g b

= do a <- m
     b <- f a
     g b

= do a <- m
     do b <- f a
        g b
```

参照：

* [Haskell 2010: Do Expressions](http://www.haskell.org/onlinereport/haskell2010/haskellch3.html#x8-470003.14)

## Maybe

**Maybe** モナドはモナドのインスタンスの中で最も単純な例です。Maybe モナドは、計算の途中のどこでも値を生成しないようにできる計算をモデル化しています。

```haskell
data Maybe a = Just a | Nothing
```

```haskell
instance Monad Maybe where
  (Just x) >>= k = k x
  Nothing  >>= k = Nothing

  return = Just
```

```haskell
(Just 3) >>= (\x -> return (x + 1))
-- Just 4

Nothing >>= (\x -> return (x + 1))
-- Nothing

return 4 :: Maybe Int
-- Just 4
```

```haskell
example1 :: Maybe Int
example1 = do
  a <- Just 3
  b <- Just 4
  return $ a + b
-- Just 7

example2 :: Maybe Int
example2 = do
  a <- Just 3
  b <- Nothing
  return $ a + b
-- Nothing
```

## リスト

**リスト**モナドはモナドインスタンスの中で 2 番目に単純な例です。

```haskell
instance Monad [] where
  m >>= f   =  concat (map f m)
  return x  =  [x]
```

例えば以下について。

```haskell
m = [1,2,3,4]
f = \x -> [1,0]
```

評価は以下のように進みます。

```haskell
m >>= f
==> [1,2,3,4] >>= \x -> [1,0]
==> concat (map (\x -> [1,0]) [1,2,3,4])
==> concat ([[1,0],[1,0],[1,0],[1,0]])
==> [1,0,1,0,1,0,1,0]
```

Haskell のリスト内包記法はリストモナドを使っても実現できます。

```haskell
a = [f x y | x <- xs, y <- ys, x == y ]

-- `a` と同じ
b = do
  x <- xs
  y <- ys
  guard $ x == y
  return $ f x y
```

```haskell
example :: [(Int, Int, Int)]
example = do
  a <- [1,2]
  b <- [10,20]
  c <- [100,200]
  return (a,b,c)
-- [(1,10,100),(1,10,200),(1,20,100),(1,20,200),(2,10,100),(2,10,200),(2,20,100),(2,20,200)]
```

## IO

``IO a`` 型の値は、実行されると ``a`` 型の値を返す (``return``) 前に何らかの I/O を行う型です。IO モナドを脱糖衣化すると：

```haskell
main :: IO ()
main = do putStrLn "What is your name: "
          name <- getLine
          putStrLn name
```

```haskell
main :: IO ()
main = putStrLn "What is your name:" >>=
       \_    -> getLine >>=
       \name -> putStrLn name
```

```haskell
main :: IO ()
main = putStrLn "What is your name: " >> (getLine >>= (\name -> putStrLn name))
```

参照：

* [Haskell 2010: Basic/Input Output](http://www.haskell.org/onlinereport/haskell2010/haskellch7.html)

## 要は？

私たちには今、大きく異なるけれども基本的である、3 つのプログラミングの概念（**失敗**、**集まり**、**副作用**）について表現できる、統一されたインターフェースがある、という自明で無い事実について考えてみてください。

`sequence`（配列する）という、関数`mcons`を畳み込む新しい関数を書いてみましょう。この関数は、2 つのモナディック (monadic) な値（``p`` と ``q``）から束縛 (bind) を用いて 2 つのリストの構成要素を取り出すという点を除いて、リストのコンストラクタ（即ち ``(a: b)``）と似ていると見なせます。

```haskell
sequence :: Monad m => [m a] -> m [a]
sequence = foldr mcons (return [])

mcons :: Monad m => m t -> m [t] -> m [t]
mcons p q = do
  x <- p
  y <- q
  return (x:y)
```

ではこの関数は、これまで見てきたモナドのそれぞれの下で、どういう意味を持つのでしょうか？

### Maybeの場合

``Maybe``の値のリストを配列(``sequence``)することで、失敗する可能性のある計算列の結果をまとめて、それらの計算全てが成功した場合に限り、まとめた値を生み出す、ということができます。

```haskell
sequence :: [Maybe a] -> Maybe [a]
```

```haskell
sequence [Just 3, Just 4]
-- Just [3,4]
sequence [Just 3, Just 4, Nothing]
-- Nothing
```

### リストの場合

リストモナドの束縛の演算では 、2つの引数から一組一組要素を取りだしたリストを作るので、``sequence`` でリストのリスト全体を束縛で畳み込むことは、任意個のリストに対する一般化されたカルテシアン積を実現していることになります。

```haskell
sequence :: [[a]] -> [[a]]
```

```haskell
sequence [[1,2,3],[10,20,30]]
-- [[1,10],[1,20],[1,30],[2,10],[2,20],[2,30],[3,10],[3,20],[3,30]]
```

### IOの場合

``sequence``は IO アクションのリストを受け取り、それらを順次実行し、得られる結果の値のリストを、配列された順番のままで返します (``return``)。

```haskell
sequence :: [IO a] -> IO [a]
```

```haskell
sequence [getLine, getLine]
-- a
-- b
-- ["a","b"]
```

こうして見ていくと、通常それぞれ独立して定義される 3 つの基本的な計算の概念が、実はみな、抽象化し再利用して、現在や将来の実装に役立つ高度に抽象的な仕組みを作ることができる、共通の構造を持つということが分かります。モナドを理解したいと思うためのきっかけが欲しいならば、これがまさにそれです！僕が思い返した時、モナドについて僕なら知りたいことの本質はこれなのです。

参照：

* [Control.Monad](http://hackage.haskell.org/package/base-4.6.0.1/docs/Control-Monad.html#g:4)

## Readerモナド

Readerモナドはモナディックな環境 (context) にある共有された不変の状態にアクセスさせてくれます。

```haskell
ask :: Reader r a
asks :: (r -> a) -> Reader r a
local :: (r -> b) -> Reader b a -> Reader r a
runReader :: Reader r a -> r -> a
```

```haskell
import Control.Monad.Reader

data MyContext = MyContext
  { foo :: String
  , bar :: Int
  } deriving (Show)

computation :: Reader MyContext (Maybe String)
computation = do
  n <- asks bar
  x <- asks foo
  if n > 0
    then return (Just x)
    else return Nothing

ex1 :: Maybe String
ex1 = runReader computation $ MyContext "hello" 1

ex2 :: Maybe String
ex2 = runReader computation $ MyContext "haskell" 0
```

Readerモナドの単純な実装：

```haskell
newtype Reader r a = Reader { runReader :: r -> a }

instance Monad (Reader r) where
  return a = Reader $ \_ -> a
  m >>= k  = Reader $ \r -> runReader (k (runReader m r)) r

ask :: Reader a a
ask = Reader id

asks :: (r -> a) -> Reader r a
asks f = Reader f

local :: (r -> b) -> Reader b a -> Reader r a
local f m = Reader $ runReader m . f
```

## Writerモナド

Writerモナドを使えば、モナディックな環境から値の遅延ストリームを流すことができます。

```haskell
tell :: w -> Writer w ()
execWriter :: Writer w a -> w
runWriter :: Writer w a -> (a, w)
```

```haskell
import Control.Monad.Writer

type MyWriter = Writer [Int] String

example :: MyWriter
example  = do
  tell [1..5]
  tell [5..10]
  return "foo"

output :: (String, [Int])
output = runWriter example
```

書き留めモナドの単純な実装：

```haskell
import Data.Monoid

newtype Writer w a = Writer { runWriter :: (a, w) }

instance Monoid w => Monad (Writer w) where
  return a = Writer (a, mempty)
  m >>= k  = Writer $ let
      (a, w)  = runWriter m
      (b, w') = runWriter (k a)
      in (b, w `mappend` w')

execWriter :: Writer w a -> w
execWriter m = snd (runWriter m)

tell :: w -> Writer w ()
tell w = Writer ((), w)
```

この実装は怠惰 (lazy) なので、実際にはサンクのストリームを単に生成できるように注意せねばなりません。``runWriter`` により怠惰に取りだされたサンクのストリームを必要とするような計算を生み出すことが望ましいことが多いですが、``runWriter`` の呼び出しの時点で強制評価される値の有限ストリームを生み出すことが必要なものであるということも多いです。書き留めモナドによる望まれない遅延性 (laziness) はしばしば悲しみを引き起こしていますが、非常に解決しやすい問題でもあります。

## 状態モナド

状態モナドを使えば、状態のあるモナディックな環境にある関数が、共有の状態にアクセスし修正することができます。

```haskell
runState  :: State s a -> s -> (a, s)
evalState :: State s a -> s -> a
execState :: State s a -> s -> s
```

```haskell
import Control.Monad.State

test :: State Int Int
test = do
  put 3
  modify (+1)
  get

main :: IO ()
main = print $ execState test 0
```

状態モナドは非純粋なものだと誤って説明されることが多いですが、実際には全く持って純粋であり、明示的に状態を渡すことで同じ作用が得られうるのです。状態モナドの単純な実装は、ほんの僅かな行数で書けます。

```haskell
newtype State s a = State { runState :: s -> (a,s) }

instance Monad (State s) where
  return a = State $ \s -> (a, s)

  State act >>= k = State $ \s ->
    let (a, s') = act s
    in runState (k a) s'

get :: State s s
get = State $ \s -> (s, s)

put :: s -> State s ()
put s = State $ \_ -> ((), s)

modify :: (s -> s) -> State s ()
modify f = get >>= \x -> put (f x)

evalState :: State s a -> s -> a
evalState act = fst . runState act

execState :: State s a -> s -> s
execState act = snd . runState act
```

## モナド・チュートリアル

多くのモナド・チュートリアルが書かれたということは、疑問を生みます。Haskell を最初に学ぶ時にモナドの何がそんなに難しいのか、ということです。その理由には 3 つの側面があると考えています。

１．**脱糖衣はいくつかのレベルで回りくどい**

私たちが書く Haskell の多くは、裏で大幅に書き直され変換されて、完全に新しい形になります。

ほとんどのモナド・チュートリアルは手動では do の糖衣を展開しません。このせいで、初心者はモナドはコードの中の疑似命令型言語に入り込んでいく方法であると考え、さらには、IO のような特定のインスタンスが最も一般性のあるモナドである、という勘違いまで助長してしまうのです。

```haskell
main = do
  x <- getLine
  putStrLn x
  return ()
```

脱糖衣を手動で出来ることが、理解のうえで重要なのです。

```haskell
main =
  getLine >>= \x ->
    putStrLn x >>= \_ ->
      return ()
```

２．**高階関数に対する非対称な中置演算子は他の言語ではあまり見られない**

```haskell
(>>=) :: Monad m => m a -> (a -> m b) -> m b
```

この演算子の左辺には ``m a`` があり、右辺には ``a -> m b`` があります。それ自体が高階関数である中置演算子がある言語はありますが、それでも頻繁に見かけるものではありません。

ですから、関数が脱糖衣された状態だと、``(>>=)`` が実は関数を合成することでずっと大きい関数を作っているため、ややこしく感じられることがあるのです。

```haskell
main =
  getLine >>= \x ->
    putStrLn >>= \_ ->
      return ()
```

前置記法で書くと、もう少し納得しやすくなります。

```haskell
main =
  (>>=) getLine (\x ->
    (>>=) putStrLn (\_ ->
          return ()
    )
  )
```

他の言語から来た人には、演算子も全て取り除いてしまったほうが直感的に分かりやすいかもしれません。

```haskell
main = bind getLine (\x -> bind putStrLn (\_ -> return ()))
  where
    bind x y = x >>= y
```

３．**アドホック多相は他の言語ではあまり見かけない**

Haskell のオーバーローディングの実装は、型推論に馴染みが無ければ非直感的に感じるかもしれません。ユーザーからは隠蔽されていますが、``(>>=)`` 即ち ``bind`` 関数は実は 3 引数関数なのです。型クラスの辞書 (``$dMonad``) が引数として余分に、暗黙的に引っ張られて付いて回っているのです。

```haskell
main $dMonad = bind $dMonad getLine (\x -> bind $dMonad putStrLn (\_ -> return $dMonad ()))
```

ただし、ここではモナドクラスの引数が（型推論で）一つの具体的なクラスのインスタンスへと単一化されている場合を除いています。この場合は、インスタンスの辞書（``$dMonadIO``）が代わりに付いて回っています。

```haskell
main :: IO ()
main = bind $dMonadIO getLine (\x -> bind $dMonadIO putStrLn (\_ -> return $dMonadIO ()))
```

これらの変換は全て、いったん理解してしまえば自明なものなので、普通は論じられないのです。私の考えでは、モナド・チュートリアルが根本的に勘違いしていることは、モナドに対する直感は伝えづらいものでは無く（メタファーを使わなければ伝わらないものでも無く）、初心者は (1)・(2)・(3) のポイントをきちんと理解していないままモナドに接し、モナドは 3 つの全てが合わさっている Haskell の概念として最初に触れる例である、という単純な事実でつまずいてしまう、ということです。

参照：

* [Monad Tutorial Fallacy](http://byorgey.wordpress.com/2009/01/12/abstraction-intuition-and-the-monad-tutorial-fallacy/)
