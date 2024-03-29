## Haskell

为了使用Yesod，你至少需要知道一些Haskell基础知识。另外，Yesod使用了一些在大部分入门教程里不会涉及的Haskell特性。虽然本书假设读者对Haskell基本熟悉，本章希望可以查疑补漏。

如果你对Haskell已经非常熟练，可以放心的跳过这一章。你可能更喜欢先体验一下Yesod，以后可以随时把本章作为参考。

如果你想打更全面的Haskell入门书，我推荐《真实世界的Haskell》和《Haskell趣学指南》。

### 术语

即便对于熟悉Haskell的人，有时仍然会弄混Haskell术语。让我们定义一些基本词汇，以便在全书中使用。

#### 数据类型
这是像Haskell这样的强类型语言的核心结构之一。有些数据类型，如(++Int++)，可以视为原始量(primitive values)，而其它数据类型将基于这些构建更复杂的值。比如，你可以这样来表示一个人：

``` haskell
data Person = Person Text Int
```

在这里，人名的类型是`Text`，年龄的类型是`Int`。因为这个例子足够简单，它将反复出现在这本书中。本质上你有三种方法创建一个新的数据类型：

* `type` 声明，如 `type GearCount = Int` 只是创建了一个已有类型的别名。类型系统不会阻止你在要用 `GearCount` 的地方使用 `Int`。使用 `type` 声明可以让你的代码更加文档化(self-documenting)。
* `newtype` 声明，如 `newtype Make = Make Text`。这种情况下，你不能在要用 `Make` 的地方使用 `Text` 替代；编译器不允许你这么做。newtype封装会在编译过程中自动消失，不会产生额外开销(overhead)。
* `data` 声明，如上文出现的 `Person`。你还可以创建代数数据类型(ADTs: Algebraic
Data Types)，比如 `data Vehicle = Bicycle GearCount | Car Make Model`。

数据构造函数:: 在上面的例子中，`Person`，`Make`，`Bicycle` 和 `Car` 是数据构造函数
。

类型构造函数:: 在上面的例子中，`Person`，`Make` 和 `Vehicle` 是类型构造函数。

类型变量:: 以 `data Maybe a = Just a | Nothing` 为例，`a` 就是类型变量。

### 工具

Haskell开发有两个主要工具。格拉斯哥Haskell编译器(GHC: Glasgow Haskell Compiler)是标准的Haskell编译器，也是Yesod唯一正式支持的编译器。你还需要Cabal，它是Haskell标准的构建(build)工具。我们不仅使用Cabal构建本地代码，也用它来自动从Hackage下载安装依赖包。Hackage是Haskell的软件包仓库。

如果你用的是Windows或Mac，强烈推荐你下载
 [Haskell Platform](http://hackage.haskell.org/platform/)。在Linux上，很多发行版在官方仓库里包含了Haskell Platform。比如在基于Debian的系统中，你可以通过运行++sudo apt-get install haskell-platform++来安装。如果你用的发行版没有包含Haskell Platform，你可以按照Haskell Platform网页上的介绍手动安装。

ps. 这个重要的包你需要手动更新。Haskell Platform包含的alex是版本2，而Yesod使用的Javascript最小化(minifier)工具hjsmin，需要版本3。一定要在Haskell Platform搭
建好后运行 `cabal install alex`，否则会有关于language-javascript包的报错信息。

[caption="注意"]
[NOTE]
====
有些人喜欢走在最前沿，总是在最新版本的GHC进入Haskell Platform之前就开始使用。我
们尽量让Yesod能工作在所有当前的GHC版本下，但我们正式支持的只有Haskell Platform
。如果你手动安装GHC，你需要注意以下几点：

* 有些包你需要额外安装，特别是__alex__和__happy__。

* 确保你安装了
  link:$$http://www.vex.net/%7Etrebla/haskell/haskell-platform.xhtml$$[所需的C
  语言库]。在Debian衍生系统中，你可以运行如下命令来安装：
++
----
sudo apt-get install libedit-dev libbsd-dev libgmp3-dev zlib1g-dev freeglut3-dev
----
====

不管你用哪种方式安装，你都需要把++cabal++的bin目录加到++PATH++环境变量里。在Mac和
Linux下，bin目录位于++$HOME/.cabal/bin++，在Windows下，bin目录位于
++%APPDATA%\cabal\bin++。

++cabal++有很多选项可以用，不过暂时先试试这两个命令：

* ++cabal update++会从Hackage下载最新的软件包列表。

* ++cabal install yesod++会安装Yesod和它所依赖的包。

[caption="注意"]
[NOTE]
====
你可能更愿意在沙盒中安装你的Yesod项目，用++cabal sandbox init++命令来开启一个
cabal沙盒。

====

[[I_sect12_d1e628]]

=== 语言编译指示(Language Pragmas)

GHC默认以接近Haskell98的模式运行。同时它自带了大量的语言扩展，允许更强大的型类
，语法变化等等。有多种方法让GHC开启这些扩展。在本书的大部分代码片断中，你都会看
到像这样的语言编译指示：

[source, haskell]
{-# LANGUAGE MyLanguageExtension #-}

这些指示需要放置在源代码文件的最上方。另外，还有两种常见方法：

* 在GHC命令行，传入额外的参数++-XMyLanguageExtension++。

* 在你的++cabal++文件中，加入++extensions++选项块.

我个人从来不用GHC命令行参数这种方法。这是个人喜好问题，我喜欢把我的设置清楚的在
文件里说明。通常建议不要把语言扩展放在++cabal++文件里；然而，在Yeosd脚手架生成的
网站里，我们采用了这种方法，以避免在每个源代码文件里都指明相同的语言编译指示。

我们在本书中用了很多语言扩展。（脚手架项目使用了11个）。我们不会涉及每一个扩展
的意思。如果需要，请查阅
link:$$http://www.haskell.org/ghc/docs/latest/html/users_guide/ghc-language-features.html$$[GHC文档]。

[[I_sect12_d1e671]]

=== 重载字符串(Overloaded Strings)

++"hello"++的类型是什么？传统上，它是++String++类型。++String++的定义是++type String =
[Char]++。不幸的是，这样有很多局限性：

* 用字符串表示文本数据是一种非常低效的方法。我们需要为每个字符分配额外的内存以
  表示字符连接操作，并且每个字符本身也要占用一个字长。

* 有时候我们有类似字符串的数据，但实际上不是文本，比如++ByteString++和HTML。

为解决这些局限，GHC有一个叫做重载字符串(++OverloadedStrings++)的语言扩展。当启
用了这个扩展，字符串字面量不再只有单一的++String++类型，它们的类型变成++IsString
a => a++，++IsString++的定义是：


[source, haskell]
class IsString a where
    fromString :: String -> a

Haskell中有很多类型都是++IsString++的实例(instances)，比如++Text++(一种高效打包的
++String++类型)，++ByteString++和++Html++。基本上本书每一个例子都假设已经启用了这个扩
展。

不幸的是，这个扩展有个缺陷：它有时候会让GHC的类型检查器困惑。假设我们有这样的
代码：


[source, haskell]
----
{-# LANGUAGE OverloadedStrings, TypeSynonymInstances, FlexibleInstances #-}
import Data.Text (Text)

class DoSomething a where
    something :: a -> IO ()

instance DoSomething String where
    something _ = putStrLn "String"

instance DoSomething Text where
    something _ = putStrLn "Text"

myFunc :: IO ()
myFunc = something "hello"
----

程序会打印出来++String++还是++Text++呢？不清楚。在这种情况下，你需要显式的用类型标
注指明++"hello"++应该被当作++String++还是++Text++处理。

[[I_sect12_d1e753]]

=== 类型族(Type Families)

类型族的基本思想是表达两种不同类型间的关联。假设我们要写一个函数，它能安全的得
到一个列表(list)的第一个元素。但是我们不希望它只能工作在列表上；我们希望它能将
++ByteString++视为一列++Word8++。要做到这一点，我们需要引入一些关联类型(associated
type)来指明对于一个特定的类型，列表内容是什么类型。


[source, haskell]
----
{-# LANGUAGE TypeFamilies, OverloadedStrings #-}
import Data.Word (Word8)
import qualified Data.ByteString as S
import Data.ByteString.Char8 () -- get an orphan IsString instance

class SafeHead a where
    type Content a
    safeHead :: a -> Maybe (Content a)

instance SafeHead [a] where
    type Content [a] = a
    safeHead [] = Nothing
    safeHead (x:_) = Just x

instance SafeHead S.ByteString where
    type Content S.ByteString = Word8
    safeHead bs
        | S.null bs = Nothing
        | otherwise = Just $ S.head bs

main :: IO ()
main = do
    print $ safeHead ("" :: String)
    print $ safeHead ("hello" :: String)

    print $ safeHead ("" :: S.ByteString)
    print $ safeHead ("hello" :: S.ByteString)
----

这里的新语法是可以在++class++和++instance++的定义中，定义++type++。我们也可以用++data++
定义，这样就能创建新的数据类型，而不是已有类型的引用。

[caption="注意"]
NOTE: 类型族也有在型类以外的用法。但是在Yesod中，所有关联类型都是型类的一部分
。更多关于类型族的信息，参阅
link:$$http://www.haskell.org/haskellwiki/GHC/Type_families$$[Haskell维基页]。

[[I_sect12_d1e789]]

=== Haskell模板(Template Haskell)

Haskell模板(TH)是一种__代码生成(code generation)__方法。Yesod在很多地方使用
Haskell模板来减少样板代码(boilerplate)，并且保证生成的代码是正确的。Haskell模板
本质上是Haskell，它会生成了一棵Haskell抽象语法树(AST: Abstract Syntax Tree)。


[caption="注意"]
NOTE: 实际上Haskell模板有更多功能，比如可以检查代码。但在Yesod中没有用到这些功
能。

写TH代码需要一些技巧，而且不幸的是这其中没有多少类型安全可言。你写的TH代码很容
易就会生成无法编译的代码。不过这只是Yesod开发者的问题，与用户无关。在开发过程中
，我们使用了大量单元测试来保证生成代码的正确性。作为用户，你所需要的就是调用这
些已有函数。比如，要引入一个外部定义的Hamlet模板，你可以这样写：

[source, haskell]
$(hamletFile "myfile.hamlet")

(Hamlet会在莎氏模板一章中介绍。)美元符号后紧跟括号，会告诉GHC接下来是一个
Haskell模板函数。括号中的代码于是在编译器中运行，生成一棵Haskell抽象语法树，然
后再编译。是的，它甚至可以
link:http://www.yesodweb.com/blog/2010/09/yo-dawg-template-haskell[抽象到这种
程度]。

TH代码的一个好处是可以执行任意的++IO++操作，因此我们可以在外部文件里放一些输入，
然后在编译时解析。一种示例用法是编译时检查的HTML、CSS和Javascript模板。

如果你的Haskell模板是用来生成声明，并且被放置在源文件的顶层，我们可以省去美元
符号和括号。也就是说：


[source, haskell]
----
{-# LANGUAGE TemplateHaskell #-}

-- 普通的函数定义，没什么特别的
myFunction = ...

-- 引入TH代码
$(myThCode)

-- 同样是引入TH代码
myThCode
----

有时候看看Haskell模板生成的代码会很有帮助。通过使用GHC的++-ddump-splices++选项可
以输出所生成的代码。

[caption="注意"]
NOTE: Haskell模板很多其它的特性这里没有涉及。更多信息，参阅
link:http://www.haskell.org/haskellwiki/Template_Haskell[Haskell维基页].

[[I_sect12_d1e833]]

=== 准引用(QuasiQuotes)

准引用(QQ: QuasiQuotes)是Haskell模板的一个小扩展，它允许我们在Haskell源文件中嵌
入任何内容。比如我们之前提到++hamletFile++这个TH函数，它需要从外部文件中读取模板
内容。我们也可以用++hamlet++这个准引用，来内联模板内容：


[source, haskell]
----
{-# LANGUAGE QuasiQuotes #-}

[hamlet|<p>This is quasi-quoted Hamlet.|]
----

这里的语法要点是方括号([])和管道符号(|)。准引用的名字在左括号和第一个竖线间，准
引用的内容在两个竖线之间。

本书中，我们通常采用QQ方法，而不是TH外部文件的方法，因为前者更容易复制粘贴。然
而，在实际项目中，除了极为简短的模板输入可以用准引用，其它情况都建议使用外部文
件，因为外部文件的方法很好的将非Haskell语法与你的Haskell代码分离。

[[I_sect12_d1e851]]

=== 小结

使用Yesod不需要你是一个Haskell专家，对Haskell基本的熟悉就够了。希望本章提供给你
足够的信息以更加轻松的跟上后面的章节。
