# Swift Style Guide
### 源文件基础
#### 文件名
所有文件的后缀名都是 .swift

一般情况下，文件名精准描述了文件内容包含的实体。主要描述某一个类型的文件，文件名就是该类型名。扩展某个类型，满足了某个协议要求的文件，文件名建议为类型名+协议名。更复杂的情况自行判断。举例：

* 包含了一个简单类型的 MyType 文件，命名为 MyType.swift
* 包含了一个类型 MyType，以及该类型的顶级辅助函数(top-level helper function) 的，还是命名为 MyType.swift
* 仅为类型 MyType 扩展了某个协议 MyProtocol 的文件，命名为 MyType+MyProtocol.swift
* 为类型 MyType 增加了多个扩展、内嵌类型或者其他可以被更通用地命名的，应当以 MyType+ 为前缀；比如 MyType+Additions.swift
* 包含的相关声明无法用某个类型或者命名空间来囊括的(例如全局的一堆数学函数)，可以描述性地命名；比如 Math.swift

#### 文件编码(UTF-8)
#### 空白字符
除了断行符外，空格(U+0020)是唯一可用的空白字符。这意味着

* 字符串和字符字面量里出现的所有其他空白字符必须以各自对应的转义序列来表示
* 不许用制表符(Tab)来做代码对齐

#### 特殊转义序列
相比 Unicode 的转义序列(比如 \u{000a}), 最好使用特殊转移序列形式，如 (\t, \n, \r, \", \', \\,  和 \0)
#### 不可见字符与不可见修饰符
不可见字符，也就是不影响字符串的图像输出的0宽度字符和其他控制字符，总是写成 Unicode 转移序列的形式。

控制字符，组合字符，以及能影响图像输出的变体选择器(variation selector, FE00-FE0F)，当它们修饰字符串或者字符时，不能用转义。如果这样一个 Unicode scalar 隔离出现或者不去修饰同一个字符串里的其他字符时，应该写作 Unicode 转移序列形式。

以下字符串格式正确，因为变音符(¨)和变体选择器(dark skin tone, 第六种肤色)修饰了临近的字符

```swift
let size  = "Übergröße"                   ✅
let shrug = "🤷🏿‍️"
```

下面的例子里，变音符和变体选择器是它们自身，所以被转义

```swift
let diaeresis = "\u{0308}"                ✅
let skinToneType6 = "\u{1F3FF}"
```

否则如果用字面量，变音符会作用于临近的双引号影响可读性；而即便多数系统将肤色选择符表示为色块，下面的例子也是不接受的，因为它是修饰符但找不到被修饰的字符。

```swift
let diaeresis = "̈"                        ⛔️
let skinToneType6 = "🏿"
```

#### 字符串字面量
Unicode 转义序列(\u{????}) 只能和 7-bit 的ASCII 一起使用，不能和其他的 Unicode 字面字符(literal code points, 比如 Ü) 在同一个字符串里同时出现。
具体而言，字符串字面量只能是如下情况之一

* 仅由纯字面字符和/或单字符转义(\t, 不能是\u{????})组成
* 仅由7-bit ASCII外加任意数量的Unicode转义序列和/或其他的转义序列组成

例子如下，前两行是正确的，因为 \n 两种情况都可以出现；但第二种不推荐，因为辨识难度大。第三行是不允许的因为第一个字符是非 7-bit ASCII 的字面量。

```swift
//correct
let size = "Übergröße\n"
let size = "\u{00DC}bergr\u{00F6}\u{00DF}e\n"
//wrong
let size = "Übergr\u{00F6}\u{00DF}e\n"         //⛔️
```

> 不要因为担心有些程序无法处理非 ASCII 编码而降低代码可读性(主动全部写成 Unicode 转义序列？)。如果这种情况发生了，有问题的是那些出错的程序，它们应该被修复。  

 
### 源文件格式
#### 文件注释
描述文件内容的注释是可选的。如果文件只包含一个抽象(abstraction, 比如只有一个类声明)，那么建议不要文件注释。因为抽象自身的注释说明已经足够了，文件注释如果不能提供更多信息的话，不建议添加。文件注释主要用于整体性描述多个抽象。

#### import 声明
一个源文件只包含它依赖的顶层模块；不多也不少。如果一个文件同时依赖 Foundation 和 UIKit，显示地把两者都导入(import)进来。不要依赖某些实现细节，比如某些 Apple Framework 在实现中传递性地导入了另外的 Framework。(译注：FrameworkA 实现时导入了 FrameworkB，偷懒的话可以只导入前者，但不推荐这么干)

推荐导入整个 module，而非个体声明(individual declarations)或者 submodules
> 有多个理由避免导入个体成员(individual members):  
> 1. 没有自动化工具解析/处理导入过程  
> 2. 现有工具(比如 Xcode 的 migrator) 将这种情况看作是边界情况，可能工作不正常  
> 3. Swift 社区的主流意见(基于官方示例和社区代码)是导入整个 module   

当整体导入某个 module 会造成顶层定义污染全局命名空间时(比如 C 接口)，允许导入个体声明。自行处理这类状况。

某些情况下允许 import submodule。比如UIKit.UIGestureRecognizerSubclass 在子类化 UIGestureRecognizer 时是必须的，然而只 import UIKit 是没用的，必须显式 import UIKit.UIGestureRecognizerSubclass。

import 声明不许折行

import 声明是源文件里首次出现非注释的符号(token)。它们分成以下几个组，组内按照字典顺序排序，组间空行隔开：

1. 非测试的 module/submodule
2. 个体声明(class, enum, func, struct, etc)
3. @testable 修饰的 module

```swift
import CoreLocation
import MyThirdPartyModule
import SpriteKit
import UIKit

import func Darwin.C.isatty

@testable import MyModuleUnderTest
```

#### 类型，变量和函数声明
一般来说，一个文件里只包含一个顶级类型(top-level type), 特别是这个类型的声明很多。一个文件定义多个相关类型的例外也存在，比如

* 类和它的委托协议(delegate protocol) 可以放在同一个文件里
* 一个类型和它的较小的辅助类型可以放在同一个文件里。用 fileprivate 就可以做到恰当的代码隔离了。

文件内部的类型、变量和函数的顺序，以及这些类型的成员的顺序，非常影响可读性。可是，对此没有简单的一纸处方，不同的文件，不同的类型，可能会采用不同的顺序。

重要的是，当维护者被问到时，可以描述出一些逻辑顺序(logic order)。举例而言，新的方法不应该只是习惯性地添加到该类型的最后面，因为这只是按照日期的顺序，而这不包含什么逻辑。

当成员顺序的逻辑确定后，这将帮助阅读代码的人和未来的维护者(包括你自己)用 “// MARK: Comments” 来提供分组描述。Xcode 会把这些标记解析为书签(比如，// MARK: - 会被解析成一条分割线)。

```swift
class MovieRatingViewController: UITableViewController {

  // MARK: - View controller lifecycle methods

  override func viewDidLoad() {
    // ...
  }

  override func viewWillAppear(_ animated: Bool) {
    // ...
  }

  // MARK: - Movie rating manipulation methods

  @objc private func ratingStarWasTapped(_ sender: UIButton?){
    // ...
  }

  @objc private func criticReviewWasTapped(_ sender: UIButton?){
    // ...
  }
}
```


#### 重载(Overload)声明
如果一个类型有多个初始化函数或者下标函数(subscripts), 或者一个文件/类型内有多个名称相同参数类型不同的函数，并且这些重载发生在同一个类型内或者 exension 内，那么它们应该顺序出现并且之间不夹杂其他代码。

####  扩展(Extension)
扩展可用于跨越多个单元(units)给类型组织功能。类似于成员顺序，有组织的结构/分组极大地影响可读性；当 reviewer 询问时，你应当能阐述里面遵从的逻辑结构。

### 通用格式
#### 列限制
正常每行100字符，除非有下面的例外说明。超过的字符必须[折行](#Line-Wrapping)

**例外：**

1. 一旦遵守字数限制就会破坏有意义的较重要的内容(比如评论里的一个长链接)
2. import 语句
3. 其他工具生成的代码

#### 大括号(Braces)
一般情况下，大括号遵守非空代码块的 Kernighan and Ritchie(K&R) 风格，同时包含以下 Swift 例外情况

1. 左括号(opening brace, 即 “{”) 不作为行首出现，除非[折行](#Line-Wrapping)里的规则要求
2. 左括号作为行尾出现，除非
 
	* 在闭包里，闭包的签名放在和左括号同一行内，此时断行断在关键字 in 后面
	* 当它可能被省略(见[一句一行](#OneStatementPerLine))
	* 空代码块可以写成 {}

3. 右括号(closing brace,  即 “}”) 作为行首出现，除非它可以被省略(见[一句一行](#OneStatementPerLine)), 或者它是空代码块的结尾
4. 右括号作为行尾出现，当且仅当它是一个语句或者一个声明的结尾。比如，else 代码块可以写成  "} else {", 括号可以放在同一行

#### 分号
不使用分号，无论是作为行尾还是分割语句。换句话说，分号出现的地方，只能是字符串字面量或者注释里。

正确的例子

```swift
func printSum(_ a: Int, _ b: Int) {
  let sum = a + b
  print(sum)
}
```

错误的示范

```swift
func printSum(_ a: Int, _ b: Int) {
  let sum = a + b;    //⛔️
  print(sum);         //⛔️
}
```


#### 一句一行 <a name="OneStatementPerLine">
一行最多一条语句，每条语句后面跟一个断行符。除非该行以一个 block 结尾，而这个 block 内包含0条或者1条语句。

```swift
guard let value = value else { return 0 }

defer { file.close() }

switch someEnum {
case .first: return 5
case .second: return 10
case .third: return 20
}

let squares = numbers.map { $0 * $0 }

var someProperty: Int {
  get { return otherObject.property }
  set { otherObject.property = newValue }
}

var someProperty: Int { return otherObject.somethingElse() }

required init?(coder aDecoder: NSCoder) { fatalError("no coder") }
```

单语句块把内容单独成行总是允许的。条件判断语句和内容(body)是否放在同一行，请在实践中自行定夺。比如说，快速返回(early-return)和基本清理任务是比较合适写成同一行的；然而如果内容里含有重要逻辑的函数调用时就不一定了。如果有疑虑，请写成多行形式。

####  折行 <a name="Line-Wrapping">
> 术语：折行(Line-wrapping) 是指将合法的一行代码拆为多行的活动  

为了达到 Google Swift style 的目的，许多声明(比如类型和函数声明)以及其他表达式(比如函数调用)可以划分成一些可分割的单元(breakable units)，它们是用不可分割的标记序列(unbreakable delimiting token sequences)来隔开的。

以下面的函数声明为例，由于过长，需要折行：

```swift
public func index<Elements: Collection, Element>(of element: Element, in collection: Elements) -> Elements.Index? where Elements.Element == Element, Element: Equatable {
  // ...
}
```

它可以被分成以下几个部分(kurk注: 原文用背景色和右上角数字标记区分，鄙人搞不定格式，这里主动断行区分)， 1，3，5，7 是不可分割标记，2，4，6，8 可以分割。

```swift
public func index< //1
Elements: Collection, Element //2
>( //3
of element: Element, in collection: Elements //4
) -> //5
Elements.Index? //6
where //7
Elements.Element == Element, Element: Equatable //8
{
  // ...
}
```

1. *不可分割*标记序列从最左边一直到泛型参数列表左边的左尖括号
2. *可分割*的泛型参数列表
3. *不可分割*标记序列(>() 分开了泛型参数和函数正式参数
4. *可分割*的逗号隔开的正式参数列表
5. *不可分割*标记序列从右圆括号(")")到箭头(->), 后面紧跟返回类型
6. *可分割*的返回类型
7. *不可分割*的 where 关键字，后面是泛型限制列表
8. *可分割*的逗号隔开的泛型限制列表

利用上述概念，Google Swift style 里对折行的主要规则是

1. 如果完整的声明，语句或者表达式适合一行展示，那么就不折行
2. 逗号分隔的列表要么水平要么竖直放置。也就是说，要么所有的元素都在同一行内，要么每个元素占一行。水平放置的列表不包含断行符，即使在外面的左右两端。例外是条件分支语句，竖直放置的里列表在第一个元素前面以及后续每一个元素的前面都要断行。
3. 以不可分割标记序列开始的中间行的缩进，和原始行保持一致
4. 竖直放置的以逗号分隔的中间行，相对于原始行缩进2个空格
5. 当左大括号 ({) 紧跟在一个折行的声明或者表达式后面，它和最后的连续行在同一行，除非该行在原始行基础上缩进了2个空格。这种情况下，大括号放在单独一行上，避免连续行在视觉上与后续代码块混在一块。

好的例子

```swift
public func index<Elements: Collection, Element>(
  of element: Element,
  in collection: Elements
) -> Elements.Index?
where
  Elements.Element == Element,
  Element: Equatable
{  // GOOD.
  for current in elements {
    // ...
  }
}
```

不好的例子

```swift
public func index<Elements: Collection, Element>(
  of element: Element,
  in collection: Elements
) -> Elements.Index?
where
  Elements.Element == Element,
  Element: Equatable {  // AVOID.
  for current in elements {
    // ...
  }
}
```

针对用 where 从句包含泛型限制的声明，添加如下规则：

1. 如果作为返回类型的泛型限制列表放同一行会超过字符数限制，那么关键字 where 前面插入换行符，同时保持和原始行一样的缩进级别。
2. 如果插入换行符后还是超过字符数限制，那么在 where 后面加入换行符，同时将限制列表竖直放置，最后一个限制条件后也加入换行符。

具体的例子会在后面相关章节里提供。

这种折行风格能保证声明的不同部分可以通过缩进和断行快速而方便地识别出来，同时也为文件中的那些部分保留相同的缩进级别。具体而言，它避免了参数基于左圆括号(opening parenthseses)缩进而导致的锯齿效应，这在其他语言里很常见：

```
public func index<Elements: Collection, Element>(of element: Element,  // AVOID.
                                                 in collection: Elements) -> Elements.Index?
    where Elements.Element == Element, Element: Equatable {
  doSomething()
}
```

##### 函数声明

<table><tr>  
<td bgcolor=#FFE0B2 ><i>modifiers func name </i>(</td>
<td></td>
<td bgcolor=#BBDEFB> <i>formal arguments </i></td>
<td></td>
<td bgcolor=#FFE0B2> )</td>
<td>{</td>
</tr></table>   
<table><tr>
<td bgcolor=#FFE0B2 ><i>modifiers func name </i>(</td>
<td></td>
<td bgcolor=#BBDEFB> <i>formal arguments </i></td>
<td></td>
<td bgcolor=#FFE0B2> ) -></td>
<td></td>
<td bgcolor=#BBDEFB> <i>result </i></td>
<td>{</td>
</tr></table>   
<table><tr>
<td bgcolor=#FFE0B2 ><i>modifiers func name </i><</td>
<td></td>
<td bgcolor=#BBDEFB> <i>generic arguments </i></td>
<td></td>
<td bgcolor=#FFE0B2>  >(</td>
<td></td>
<td bgcolor=#BBDEFB> <i>formal arguments </i></td>
<td></td>
<td bgcolor=#FFE0B2> ) throws -> </td>
<td></td>
<td bgcolor=#BBDEFB> <i>result </i></td>
<td>{</td>
<table><tr>
<td bgcolor=#FFE0B2 ><i>modifiers func name </i><</td>
<td></td>
<td bgcolor=#BBDEFB> <i>generic arguments </i></td>
<td></td>
<td bgcolor=#FFE0B2>  >(</td>
<td></td>
<td bgcolor=#BBDEFB> <i>formal arguments </i></td>
<td></td>
<td bgcolor=#FFE0B2> ) throws -></td>
<td></td>
<td bgcolor=#BBDEFB><i>result </i></td>
<td></td>
<td bgcolor=#FFE0B2>where</td>
<td></td>
<td bgcolor=#BBDEFB><i>generic constraints </i></td>
<td>{</td>
</tr></table>

从左往右应用上面的规则，我们得到

```swift
public func index<Elements: Collection, Element>(
  of element: Element,
  in collection: Elements
) -> Elements.Index? where Elements.Element == Element, Element: Equatable {
  for current in elements {
    // ...
  }
}
```

以右圆括号(")")结尾的协议里的函数声明，可以把结尾的括号放在最后一个参数的同一行上，也可以单独放一行。

```swift
public protocol ContrivedExampleDelegate {
  func contrivedExample(
    _ contrivedExample: ContrivedExample,
    willDoSomethingTo someValue: SomeValue)
}

public protocol ContrivedExampleDelegate {
  func contrivedExample(
    _ contrivedExample: ContrivedExample,
    willDoSomethingTo someValue: SomeValue
  )
}
```

如果类型过于复杂并且/或者深度嵌套，参数列表和限制列表以及返回类型里的单独元素也可以折行。在一些极端的案例里，适用于声明的折行规则也同样适用于这些部分。

```swift
public func performanceTrackingIndex<Elements: Collection, Element>(
  of element: Element,
  in collection: Elements
) -> (
  Element.Index?,
  PerformanceTrackingIndexStatistics.Timings,
  PerformanceTrackingIndexStatistics.SpaceUsed
) {
  // ...
}
```

不过，在可能的情况下，typealias 或者其他一些方法是简化复杂声明的更好手段。

##### 类型和扩展声明(Type and Extension Declarations)
下面的例子同样地适用于 class, struct, enum, extension 和 protocol (明显的例外是仅 class 在继承列表里有超类，其他没有。除此以外它们的结构是相似的)

<table><tr>  
<td bgcolor=#FFE0B2 ><i>modifiers class name </i></td>
<td></td>
<td>{</td>
</tr></table>
<table><tr>
<td bgcolor=#FFE0B2 ><i>modifiers class name: </i></td>
<td></td>
<td bgcolor=#BBDEFB> <i>superclass and protocols </i></td>
<td></td>
<td>{</td>
</tr></table>
<table><tr>
<td bgcolor=#FFE0B2 ><i>modifiers class name <</i></td>
<td></td>
<td bgcolor=#BBDEFB> <i>generic arguments </i></td>
<td></td>
<td bgcolor=#FFE0B2>  >:</td>
<td></td>
<td bgcolor=#BBDEFB> <i>superclass and protocols </i></td>
<td></td>
<td>{</td>
<table><tr>
<td bgcolor=#FFE0B2 ><i>modifiers class name <</i></td>
<td></td>
<td bgcolor=#BBDEFB> <i>generic arguments </i></td>
<td></td>
<td bgcolor=#FFE0B2>  >:</td>
<td></td>
<td bgcolor=#BBDEFB> <i>superclass and protocols </i></td>
<td></td>
<td bgcolor=#FFE0B2>where</td>
<td></td>
<td bgcolor=#BBDEFB><i>generic constraints </i></td>
<td>{</td>
</tr></table>


```swift
class MyClass:
  MySuperclass,
  MyProtocol,
  SomeoneElsesProtocol,
  SomeFrameworkProtocol
{
  // ...
}

class MyContainer<Element>:
  MyContainerSuperclass,
  MyContainerProtocol,
  SomeoneElsesContainerProtocol,
  SomeFrameworkContainerProtocol
{
  // ...
}

class MyContainer<BaseCollection>:
  MyContainerSuperclass,
  MyContainerProtocol,
  SomeoneElsesContainerProtocol,
  SomeFrameworkContainerProtocol
where BaseCollection: Collection {
  // ...
}

class MyContainer<BaseCollection>:
  MyContainerSuperclass,
  MyContainerProtocol,
  SomeoneElsesContainerProtocol,
  SomeFrameworkContainerProtocol
where
  BaseCollection: Collection,
  BaseCollection.Element: Equatable,
  BaseCollection.Element: SomeOtherProtocolOnlyUsedToForceLineWrapping
{
  // ...
}
```

##### 函数调用(Function Calls)
函数调用如果断行了，那么每个参数单独占一行，相对原始行缩进2个空格。

和函数声明一样，如果函数调用以右圆括号(")")结尾(也就是说最后一个参数不是闭包, 非 trailing closure)，那么这个圆括号可以和最后一个参数放在同一行，也可以单独成行。

```swift
let index = index(
  of: veryLongElementVariableName,
  in: aCollectionOfElementsThatAlsoHappensToHaveALongName)

let index = index(
  of: veryLongElementVariableName,
  in: aCollectionOfElementsThatAlsoHappensToHaveALongName
)
```

如果函数是以尾闭包结尾(trailing closure)并且闭包的签名必须折行，那么签名应该另起一行，(有必要的话)括号内的参数列表也折行，把它跟接下来闭包的实现区分开来。

```swift
someAsynchronousAction.execute(withDelay: howManySeconds, context: actionContext) {
  (context, completion) in
  doSomething(withContext: context)
  completion()
}
```


##### 控制流语句(Control Flow Statements)
当一个控制流语句(if, while, guard, for)折行时, 第一个衔接的行缩进到跟紧跟着控制关键字后的代码(token)位置。后续行如果语法上是同一级的，则对齐第二行；否则按照语法层级每层往后缩进2个空格。

在控制流语句主体(body)之前的左大括号("{")，可以跟在前面的最后一个连续行尾部，也可以另起一行，缩进跟语句的最开始一致。guard 语句里的 “else {” 必须在同一行，不能从中断行。

```swift
if aBooleanValueReturnedByAVeryLongOptionalThing() &&
   aDifferentBooleanValueReturnedByAVeryLongOptionalThing() &&
   yetAnotherBooleanValueThatContributesToTheWrapping() {
  doSomething()
}

if aBooleanValueReturnedByAVeryLongOptionalThing() &&
   aDifferentBooleanValueReturnedByAVeryLongOptionalThing() &&
   yetAnotherBooleanValueThatContributesToTheWrapping()
{
  doSomething()
}

if let value = aValueReturnedByAVeryLongOptionalThing(),
   let value2 = aDifferentValueReturnedByAVeryLongOptionalThing() {
  doSomething()
}

if let value = aValueReturnedByAVeryLongOptionalThing(),
   let value2 = aDifferentValueReturnedByAVeryLongOptionalThingThatForcesTheBraceToBeWrapped()
{
  doSomething()
}

guard let value = aValueReturnedByAVeryLongOptionalThing(),
      let value2 = aDifferentValueReturnedByAVeryLongOptionalThing() else {
  doSomething()
}

guard let value = aValueReturnedByAVeryLongOptionalThing(),
      let value2 = aDifferentValueReturnedByAVeryLongOptionalThing()
else {
  doSomething()
}

for element in collection
    where element.happensToHaveAVeryLongPropertyNameThatYouNeedToCheck {
  doSomething()
}
```

##### 其他表达式
当折行发生的表达式不是函数调用语句(规则见前述)时，第二行(紧跟第一个断行符的那行)相对原始行缩进2个空格。

如果超过一个折行，缩进可以不一致，每次增加2个空格。一般来说，两个后续行缩进一致，当且仅当它们语法上层级一致(begin with syntactically parallel elements)。但是，如果长长的表达式折行成大量连续行，建议引入中间临时变量将其简化断开。

```swift
                                          ✅
let result = anExpression + thatIsMadeUpOf * aLargeNumber +
  ofTerms / andTherefore % mustBeWrapped + (
    andWeWill - keepMakingItLonger * soThatWeHave / aContrivedExample)
```

```swift
                                          ⛔️
let result = anExpression + thatIsMadeUpOf * aLargeNumber +
    ofTerms / andTherefore % mustBeWrapped + (
        andWeWill - keepMakingItLonger * soThatWeHave / aContrivedExample)
```

#### 水平空白字符(Horizontal Whitespace)
> 术语: 本节水平空白字符专指*内部*的空格。这些规则和行首增加或者减少空格无关。    

除了语言或其他风格规则的要求，文字和评论外，单个空格仅在以下情况出现：

1. 如果条件分支关键字(if, guard, while, switch)后面紧跟着的是左圆括号("("), 用空格隔开它们

	```swift
	if (x == 0 && y == 0) || z == 0 {          //✅
	  // ...
	}
	```
	```swift
	if(x == 0 && y == 0) || z == 0 {           //⛔️
	  // ...
	}
	```

2. 不是行首的右大括号("}")前面、左大括号("{")的前面、以及如果右边有代码时，左大括号的后边都需要加空格

	```swift
	let nonNegativeCubes = numbers.map { $0 * $0 * $0 }.filter { $0 >= 0 } //✅
	```
	```swift
	let nonNegativeCubes = numbers.map { $0 * $0 * $0 } .filter { $0 >= 0 } //⛔️
	let nonNegativeCubes = numbers.map{$0 * $0 * $0}.filter{$0 >= 0}
	```

3. 在二元或者三元运算符(包括下面描述的类似运算符的符号)的两侧需要加空格。例外情况在最后面有说明。

	1. 赋值运算符(=), 用于赋值，变量和属性的初始化，以及函数参数的默认值

		```swift
		var x = 5                                                //✅
		func sum(_ numbers: [Int], initialValue: Int = 0) {
		  // ...
		}
		```
		```swift
		var x=5                                                //⛔️
		func sum(_ numbers: [Int], initialValue: Int=0) {
		  // ...
		}
		```

	2. “&” 在用于 protocol 组合时

		```swift
		                                               //✅
		func sayHappyBirthday(to person: NameProviding & AgeProviding) {
		  // ...
		}
		```
		```swift
		                                                //⛔️
		func sayHappyBirthday(to person: NameProviding&AgeProviding) {
		  // ...
		}
		```

	3. 声明和实现一个运算符时

		```swift
		static func == (lhs: MyType, rhs: MyType) -> Bool {   //✅
		  // ...
		}
		```
		```swift
		static func ==(lhs: MyType, rhs: MyType) -> Bool {  //⛔️
		  // ...
		}
		```

	4. 在函数返回类型前面的箭头 (->)
    
		```swift
		func sum(_ numbers: [Int]) -> Int {         //✅
		  // ...
		}
		```
		```swift
		func sum(_ numbers: [Int])->Int {           //⛔️
		  // ...
		}
		```

	5. **例外**：点操作符(.)用于引用值或者类型成员时两边不需要空格
    
		```swift                                               
		let width = view.bounds.width                         //✅
		```
		```swift                                                
		let width = view . bounds . width                     //⛔️
		```

	6. **例外**：范围运算符(range operator, "..<" 和 "...")两侧
    
		```swift
		for number in 1...5 {                                 //✅
		  // ...
		}
		
		let substring = string[index..<string.endIndex]
		```
		```swift
		for number in 1 ... 5 {                              //⛔️
		  // ...
		}
		
		let substring = string[index ..< string.endIndex]
		```

4. 逗号(,)的前面不需要，但是后面需要加空格，特别是 tuple/array/dictionary 的字面量(literals)
    
	```swift
	let numbers = [1, 2, 3]                               //✅
	```
	```swift
	let numbers = [1,2,3]                                 //⛔️
	let numbers = [1 ,2 ,3]
	let numbers = [1 , 2 , 3]
	```

5. 分号(:)的前面不需要，但是后面需要加空格：
	1. 超类(superclass) 和 协议一致性列表(protocol conformance lists) 以及泛型限制(generic constraints)
    
		```swift
		struct HashTable: Collection {                        //✅
		  // ...
		}
		
		struct AnyEquatable<Wrapped: Equatable>: Equatable {
		  // ...
		}
		```
		```swift
		struct HashTable : Collection {                       //⛔️
		  // ...
		}
		
		struct AnyEquatable<Wrapped : Equatable> : Equatable {
		  // ...
		}
		```

	2. 函数参数标签和 tuple 的元素标签
    
		```swift
		let tuple: (x: Int, y: Int)                         //✅
		
		func sum(_ numbers: [Int]) {
		  // ...
		}
		```
		```swift
		let tuple: (x:Int, y:Int)                           //⛔️
		let tuple: (x : Int, y : Int)
		
		func sum(_ numbers:[Int]) {
		  // ...
		}
		
		func sum(_ numbers : [Int]) {
		  // ...
		}
		```

	3. 变量和属性的显示类型声明
    
		```swift
		let number: Int = 5                    //✅
		```
		```swift
		let number:Int = 5                    //⛔️
		let number : Int = 5
		```

	4. dictionary 类型的简写
    
		```swift
		var nameAgeMap: [String: Int] = []                   //✅
		```
		```swift
		var nameAgeMap: [String:Int] = []                    //⛔️
		var nameAgeMap: [String : Int] = []
		```

	5. dictionary 的字面量
    
		```swift
		let nameAgeMap = ["Ed": 40, "Timmy": 9]            //✅
		```
		```swift
		let nameAgeMap = ["Ed":40, "Timmy":9]              //⛔️
		let nameAgeMap = ["Ed" : 40, "Timmy" : 9]
		```

6. 行尾的单行注释(//), 前面需要至少两个空格，后面紧跟一个空格
    
	```swift
	let initialFactor = 2  // Warm up the modulator.   //✅
	```
	```swift
	let initialFactor = 2 //    Warm up the modulator. //⛔️
	```

7. array 和 dictionary 的字面量的方括号("[]")外面, tuple 字面量的圆括号外面。里面不需要。
    
	```swift
	let numbers = [1, 2, 3]                             //✅
	```
	```swift
	let numbers = [ 1, 2, 3 ]                           //⛔️
	```


#### 水平对齐(Horizontal Alignment)
> 术语: 水平对齐是指在实践中添加额外的空白字符使得某些特定标记(tokens)出现在上一行的一些特定标记的正下方。  

除了编写明显的表格数据时，忽略对齐会损害可读性以外，水平对齐是禁止的。其他情况下(比如，排列 class 或者 struct 的存储属性(stored property) 的声明的类型时)，水平对齐是维护问题的邀请函：引入一个新的成员时可能需要将其他所有成员重新对齐。

```swift
struct DataPoint {                             //✅
  var value: Int
  var primaryColor: UIColor
}
```
```swift
struct DataPoint {                            //⛔️
  var value:        Int
  var primaryColor: UIColor
}
```

#### 竖直空行(Vertical Whitespace)
单个空行出现在以下情况：

1. 一个类型的连续成员之间，包括属性(properties), 初始化函数(initializers), 方法(methods), 枚举实例(enum cases) 和内嵌类型(enum types)，不包含：
	1. 两个存储属性或者两个枚举实例之间的空行是可选的, 只要它们的声明适合一行写下。此时空行用于创建逻辑分组(logic groupings)。
	2. 两个关系及其紧密的属性之间的空行是可选的，否则不符合上面的标准。比如，一个私有的 store property 和相关的公开 computed property
2. 语句(statements)之间仅用于将代码隔成逻辑小块(logic subsections)。
3. 在类型的第一个成员之前，以及最后一个成员之后是可选的(不推荐也不鼓励)。
4. 本文档其他章节显式要求的。

多行空行是允许的，但不是必须的(也不鼓励)。如果你使用了连续空行，注意在代码库里保持一致。

#### 圆括号(Parentheses)
关键字 if, guard, switch, while 后紧跟的表达式的最外面不需要圆括号。

```swift
if x == 0 {                             //✅
  print("x is zero")
}

if (x == 0 || y == 1) && z == 2 {
  print("...")
}
```
```swift
if (x == 0) {                            //⛔️
  print("x is zero")
}

if ((x == 0 || y == 1) && z == 2) {
  print("...")
}
```

当作者和审查人(reviewer)都同意，删掉括号后不会有错误理解，添加括号也无助于增加可读性时，可选的分组括号才可以省略掉。要求所有人记住完整的 Swift 运算符优先级表是不合理的。

### 格式化特定结构(Formatting Specific Constructs)
#### 非文档注释
非文档注释使用C++风格单行注释符(//), 不使用C风格多行注释符(/* ...  */)

#### 属性(Properties)
本地变量尽量靠近首次使用处前声明(合理范围内)，以缩小作用域(scope)范围。

多元组解构(tuple destructuring)例外，其他情况每个 let/var 都只能声明一个常/变量(无论是属性还是局部变量)

```swift
var a = 5                              //✅
var b = 10

let (quotient, remainder) = divide(100, 9)
```
```swift
var a = 5, b = 10                        //⛔️
```

#### Switch 语句
case 语句的缩进和 switch 语句保持一致; case 内部的代码块在此基础上缩进2个空格。

```swift
switch order {                             //✅
case .ascending:
  print("Ascending")
case .descending:
  print("Descending")
case .same:
  print("Same")
}
```
```swift
switch order {                             //⛔️
  case .ascending:
    print("Ascending")
  case .descending:
    print("Descending")
  case .same:
    print("Same")
}
```
```swift
switch order {                            //⛔️
case .ascending:
print("Ascending")
case .descending:
print("Descending")
case .same:
print("Same")
}
```

#### 枚举(Enum Cases)
一般来说，enum 里每行只有一个 case。当所有 case 都没有关联值或者原始值时，可以使用逗号分隔的形式，一起放在同一行，不需要进一步的文档，因为名称里就表达完善了。

```swift
public enum Token {                                 //✅
  case comma
  case semicolon
  case identifier
}

public enum Token {
  case comma, semicolon, identifier
}

public enum Token {
  case comma
  case semicolon
  case identifier(String)
}
```
```swift
public enum Token {                       //⛔️
  case comma, semicolon, identifier(String)
}
```

当所有 case 都是 indirect 时，可以将 indirect 关键字往前提升到 enum 声明处，单独的 case 里省略。

```swift
public indirect enum DependencyGraphNode {           //✅
  case userDefined(dependencies: [DependencyGraphNode])
  case synthesized(dependencies: [DependencyGraphNode])
}
```
```swift
public enum DependencyGraphNode {                    //⛔️
  indirect case userDefined(dependencies: [DependencyGraphNode])
  indirect case synthesized(dependencies: [DependencyGraphNode])
}
```

当某个 case 里没有关联值时，不要添加空的圆括号

```swift
public enum BinaryTree<Element> {                //✅
  indirect case node(element: Element, left: BinaryTree, right: BinaryTree)
  case empty  // GOOD.
}
```
```swift
public enum BinaryTree<Element> {                //⛔️
  indirect case node(element: Element, left: BinaryTree, right: BinaryTree)
  case empty()  // AVOID.
}
```

当被问到时，作者应当能说明 enum 里各个 case 的排列遵守的逻辑顺序。如果没有，建议按照名称的字典顺序。

下面的例子里，case 按照 HTTP 状态码的数字的顺序并用空行做了分组。

```swift
public enum HTTPStatus: Int {                //✅
  case ok = 200

  case badRequest = 400
  case notAuthorized = 401
  case paymentRequired = 402
  case forbidden = 403
  case notFound = 404

  case internalServerError = 500
}
```

下面的例子可读性就差了一些。尽管已经按照字典顺序排列，与值相关的有意义的分组信息已经丢失。

```swift
public enum HTTPStatus: Int {                //⛔️
  case badRequest = 400
  case forbidden = 403
  case internalServerError = 500
  case notAuthorized = 401
  case notFound = 404
  case ok = 200
  case paymentRequired = 402
}
```


#### 尾闭包(Trailing Closures)
不推荐这样的重载：两个函数唯一的差别是尾闭包参数的标签(label)。这种情况下不推荐尾闭包语法——省略掉标签后无法区分是对哪个函数的调用。

请看下面的例子，阻止了用尾闭包语法调用 greet 函数：

```swift
//⛔️
func greet(enthusiastically nameProvider: () -> String) {
  print("Hello, \(nameProvider())! It's a pleasure to see you!")
}

func greet(apathetically nameProvider: () -> String) {
  print("Oh, look. It's \(nameProvider()).")
}

greet { "John" }  // error: ambiguous use of 'greet'
```

这个例子的修复，是通过修改闭包参数之外的函数签名的部分来区分两个函数——此处是修改函数名称(base name)：

```swift
//✅
func greetEnthusiastically(_ nameProvider: () -> String) {
  print("Hello, \(nameProvider())! It's a pleasure to see you!")
}

func greetApathetically(_ nameProvider: () -> String) {
  print("Oh, look. It's \(nameProvider()).")
}

greetEnthusiastically { "John" }
greetApathetically { "not John" }
```

如果一个函数调用包含多个闭包参数，那么它们*都不用*尾闭包语法；*均使用*标签并且嵌套在参数列表的圆括号内。

```swift
UIView.animate(                //✅
  withDuration: 0.5,
  animations: {
    // ...
  },
  completion: { finished in
    // ...
  })
```
```swift
UIView.animate(                //⛔️
  withDuration: 0.5,
  animations: {
    // ...
  }) { finished in
    // ...
  }
```

如果一个函数只有一个闭包参数，而且是最后一个参数，那么它*总是*在调用时用尾闭包语法，除非是以下避免歧义或者解析错误的情况：

1. 前面描述过的，除了参数的标签外，函数签名完全一致；此时必须写出参数标签避免歧义。
2. 当使用尾闭包的格式时，闭包的主体(body)可以被解析为控制流语句的主体(body)。这种情况下，必须还原为标签式的参数形式。

```swift
//✅
Timer.scheduledTimer(timeInterval: 30, repeats: false) { timer in
  print("Timer done!")
}

if let firstActive = list.first(where: { $0.isActive }) {
  process(firstActive)
}
```
```swift
//⛔️
Timer.scheduledTimer(timeInterval: 30, repeats: false, block: { timer in
  print("Timer done!")
})

// This example fails to compile.
if let firstActive = list.first { $0.isActive } {
  process(firstActive)
}
```

当函数调用使用的尾闭包语法不包含其他参数时，函数名后的空圆括号对("()")省略掉。

```swift
let squares = [1, 2, 3].map { $0 * $0 }         //✅
```
```swift
let squares = [1, 2, 3].map({ $0 * $0 })        //⛔️
let squares = [1, 2, 3].map() { $0 * $0 }       //⛔️
```

#### 结尾逗号(Trailing Commas)
array 和 dictionary 的字面量里，当每个元素占一行时，最后一个元素后面的逗号(trailing commas)是必须的。这样做，方便后续更清晰地在尾部添加新元素。

```swift
let configurationKeys = [                      //✅
  "bufferSize",
  "compression",
  "encoding",                                    // GOOD.
]
```
```swift
let configurationKeys = [                      //⛔️
  "bufferSize",
  "compression",
  "encoding"                                     // AVOID.
]
```

#### 数字字面量(Numeric Literals)
推荐(但不强制要求)在较长的数字字面量 (十进制，十六进制，八进制，二进制)是数值含义或者存在特定域分组含义时，使用下划线(_)来分割数字，增强可读性。

推荐的分组是十进制三个数字一组(译注：英文习惯，thousands，国内可以考虑万，4个)，十六进制4个一组，二进制则4个或者8个一组，或者特定值的字段分界(如果它们存在，例如 unix 文件权限是3个8进制数来描述的)

如果字面量作为不透明标识符不具有数值上的含义，则不要将数字分组。

#### 属性(Attributes)
带参数的属性（例如 @availability(...) 或 @objc(...) ）写在它们应用的声明之前，每个单独占一行，按字典顺序排列，缩进和声明保持一致。

```swift
@available(iOS 9.0, *)                                   //✅
public func coolNewFeature() {    
  // ...
}
```
```swift
@available(iOS 9.0, *) public func coolNewFeature() {    //⛔️
  // ...
}
```

不带参数的属性 (比如单独的 @objc, @IBOutlet, @NSManaged) 按照字典顺序排列并且*可以*和声明放在同一行(当且仅当一行放得下，不触发断行)。如果声明所在的那行新添加一个属性会触发之前不需要断行的声明的断行，那么这个属性可以单独占一行。

```swift
public class MyViewController: UIViewController {     //✅
  @IBOutlet private var tableView: UITableView!
}
```

### 命名(Naming)
#### Apple’s API Style Guidelines
[苹果官方的 Swift 命名和 API 设计指南](https://swift.org/documentation/api-design-guidelines/) 被认为是本风格指南的一部分。里面的内容需要遵循，就像我们在这里重复(要求)了一遍。

#### 命名约定不是访问控制
受限访问控制(internal, private, fileprivate) 是为了对客户隐藏信息，不是为了命名约定。

命名约定(比如用下划线当前缀)是仅在少数场合下，当为了克服语言限制，一个声明需要给出更高级的可见度(visibility)时使用。比如，一个类型的某个函数，被设计为让穿越了 module 边界的库的其他部分调用，那么就必须被声明为 public

#### 标识符(Identifiers)
一般情况下，标识符仅由7-bit ASCII 字符组成。如果在代码库的问题领域，Unicode 标识符具有清晰而合理的含义(比如希腊字母可用于表达数学概念)，同时作为代码库拥有者的团队对此完全了解，那么也是可用的。

```swift
let smile = "😊"                               //✅
let deltaX = newX - previousX
let Δx = newX - previousX
```
```swift
let 😊 = "😊"                                  //⛔️
```

#### 初始化函数(Initializers)
明确起见，初始化函数里直接对应于存储属性(stored property)的参数名称，和属性名称保持一致。赋值时显式添加 “self.” 以避免歧义。

```swift
public struct Person {                               //✅
  public let name: String
  public let phoneNumber: String

  // GOOD.
  public init(name: String, phoneNumber: String) {
    self.name = name
    self.phoneNumber = phoneNumber
  }
}
```
```swift
public struct Person {                                  //⛔️
  public let name: String
  public let phoneNumber: String

  // AVOID.
  public init(name otherName: String, phoneNumber otherPhoneNumber: String) {
    name = otherName
    phoneNumber = otherPhoneNumber
  }
}
```

#### Static and Class Properties
静态属性和类属性如果是返回一个所属类型的实例的，名称上不需要以该类型名称为后缀。

```swift
public class UIColor {                               //✅
  public class var red: UIColor {                // GOOD.
    // ...
  }
}

public class URLSession {
  public class var shared: URLSession {          // GOOD.
    // ...
  }
}
```
```swift
public class UIColor {                                  //⛔️
  public class var redColor: UIColor {           // AVOID.
    // ...
  }
}

public class URLSession {
  public class var sharedSession: URLSession {   // AVOID.
    // ...
  }
}
```

如果静态属性或者雷属性用于实现单件模式(singleton), 返回所属类型的一个实例，那么名称推荐使用 “shared” 或者 “default”。本风格指南并不对此作出强制要求，作者需要自行选择最适合该类型的名称。

#### 全局常量(Global Constants)
类似其他变量，全局常量也使用 lowerCamelCase, 首字母小写的驼峰格式。不使用 g 或者 k 开头的匈牙利记法。

```swift
let secondsPerMinute = 60                        //✅
```
```swift
let SecondsPerMinute = 60                        //⛔️
let kSecondsPerMinute = 60
let gSecondsPerMinute = 60
let SECONDS_PER_MINUTE = 60
```

#### 委托方法(Delegate Methods)
受到 Cocoa 协议的启发，委托协议(delegate protocol)和类似委托协议(比如 data sources, 数据源)里的方法，使用下面描述的语言语法来命名。

> 术语“委托的源对象(delegate's source object)”指的是调用委托内方法的对象。比如说，UITableView 就是调用 UITableViewDelegate (设置为 UITableView 的 delegate 属性)里的方法的源对象。  

所有的方法把源对象作为第一个参数。

如果方法仅有源对象一个参数：

* 如果方法没有返回值 (比如有些方法只是用于通知委托对象某个事件发生了)，那么方法的名称(base name) 由委托的源对象的类型后面紧跟描述事件的指示性动词短语组成。参数不需要标签(unlabeled).

        func scrollViewDidBeginScrolling(_ scrollView: UIScrollView)    //✅

* 如果方法返回 Bool (比如对委托的源对象做断言的代码)，那么方法名称由委托的源对象的类型后面紧跟描述断言的指示性或者条件性动词短语组成。参数不需要标签。

        func scrollViewShouldScrollToTop(_ scrollView: UIScrollView) -> Bool    //✅

* 如果方法返回其他值 (比如查询委托的源对象的属性信息的), 那么方法的名字是一个描述被查询的*名词性短语*。方法参数的标签可以是一个介词，或者是一个介词后置的短语(这个短语恰当地组合了名词短语和委托的源对象) 

        func numberOfSections(in scrollView: UIScrollView) -> Int    //✅

如果方法还需要额外的参数，那么方法名就是委托的源对象*自己*的类型，同时第一个参数不需要标签：

* 如果方法没有返回值，那么方法的第二个参数的标签(label)是一个描述事件的指示性动词短语构成；参数对象构成了这个事件的直接对象或者介词对象。其他的参数(如果存在)提供了进一步的上下文。

                func tableView(               //✅
          _ tableView: UITableView,
          willDisplayCell cell: UITableViewCell,
          forRowAt indexPath: IndexPath)

* 如果方法返回 Bool ，那么第二个参数的标签由一个指示性或者条件性动词短语构成，该短语用参数来描述返回值。其他的参数(如果存在)提供了进一步的上下文。

        func tableView(                  //✅
          _ tableView: UITableView,
          shouldSpringLoadRowAt indexPath: IndexPath,
          with context: UISpringLoadedInteractionContext
        ) -> Bool

* 如果方法返回其他值 , 那么第二个参数的标签是名词性短语并且以介词结尾，它用参数来描述返回值。其他的参数(如果存在)提供了进一步的上下文。

        func tableView(               //✅
          _ tableView: UITableView,
          heightForRowAt indexPath: IndexPath
        ) -> CGFloat

关于 [委托和数据源](https://developer.apple.com/library/content/documentation/General/Conceptual/CocoaEncyclopedia/DelegatesandDataSources/DelegatesandDataSources.html)  的苹果文档也包含了一些不错的关于这些命名的通用指导意见。
 
### 编程实践
本节规则中的共同主题是：避免冗余，避免歧义，并且隐式优于显式，除非显式提高了可读性和/或减少了歧义。

#### 编译器警告
可能的话应该干掉所有警告。容易解决的警告必须解决。

一个合理的例外是弃用警告(deprecation warnings), 当无法立即迁移到替代 API, 或者一个 API 针对外部用户已弃用但在弃用期间必须在库内得到支持。

#### 初始化函数(Initializers)
对于 struct, Swift 合成了一个非公开的，成员初始化函数 init, 它为声明的 var 属性以及没有默认值的 let 属性提供了参数。当这个初始化函数是合适的(也就是说不需要 public 的)时，它被使用并且不写入显式的初始化函数。

由特殊的 ExpressibleBy*Literal 编译器协议声明的初始化函数不能被直接调用。

```swift
struct Kilometers: ExpressibleByIntegerLiteral {      //✅
  init(integerLiteral value: Int) {
    // ...
  }
}

let k1: Kilometers = 10                          // GOOD.
let k2 = 10 as Kilometers                        // ALSO GOOD.
```
```swift
struct Kilometers: ExpressibleByIntegerLiteral {        //⛔️
  init(integerLiteral value: Int) {
    // ...
  }
}

let k = Kilometers(integerLiteral: 10)           // AVOID.
```

仅当调用的接收者(receiver)是一个元类型(metatype)时，才允许显式调用 .init(...) 。使用字面类型名间接调用初始化函数时，.init 被省略。(使用 MyType.init 语法直接引用初始化函数将其转换成闭包是允许的。)

```swift
let x = MyType(arguments)                 //✅

let type = lookupType(context)
let x = type.init(arguments)

let x = makeValue(factory: MyType.init)
```
```swift
let x = MyType.init(arguments)        //⛔️
```

#### 属性(Properties)
只读的计算属性(computed property) 里的 get 区块省略了，它的主体(body)直接内嵌到属性声明里。

```swift
var totalCost: Int {                 //✅
  return items.sum { $0.cost }
}
```
```swift
var totalCost: Int {                 //⛔️
  get {
    return items.sum { $0.cost }
  }
}
```

#### Types with Shorthand Names
array, dictionary 和 optional 总是写成简写形式; 即 [Element], [Key: Value], Wrapped?。长格式 Array<Element>, Dictionary<Key, Value>, Optional<Wrapped> 只有在编译器	要求时才需要。比如，Swift 解析器要求 Array<Element>.Index, 不接受 [Element].Index。

```swift
func enumeratedDictionary<Element>(       //✅
  from values: [Element],
  start: Array<Element>.Index? = nil
) -> [Int: Element] {
  // ...
}
```
```swift
func enumeratedDictionary<Element>(        //⛔️
  from values: Array<Element>,
  start: Optional<Array<Element>.Index> = nil
) -> Dictionary<Int, Element> {
  // ...
}
```

Void 是空白元组 () 的 typealias, 从实现的角度来看，它们是一回事。在函数类型声明(闭包，持有函数引用的变量)时，返回类型总是写作 Void，不用 () 。用 func 来声明的函数里，作为返回类型的 Void 完全省略了。

```swift
func doSomething() {                   //✅
  // ...
}

let callback: () -> Void
```
```swift
func doSomething() -> Void {            //⛔️
  // ...
}

func doSomething2() -> () {
  // ...
}

let callback: () -> ()
```

#### Optional Types
设计算法时，警戒值(Sentinel Values)是需要避免的(比如当没有找到结果时，返回索引值 -1)。由于类型系统无法区分警戒值和有效值，警戒值很容易出错并通过其他逻辑层传播。

Optional 用于传递无错误结果，结果要么是一个值，要么表示值不存在。比如，搜索结果为空是一个**有效和预期**的结果，而不是错误。

```swift
func index(of thing: Thing, in things: [Thing]) -> Int? {//✅
  // ...
}

if let index = index(of: thing, in: lotsOfThings) {
  // Found it.
} else {
  // Didn't find it.
}
```
```swift
func index(of thing: Thing, in things: [Thing]) -> Int {//⛔️
  // ...
}

let index = index(of: thing, in: lotsOfThings)
if index != -1 {
  // Found it.
} else {
  // Didn't find it.
}
```

Optional 也用于单一明显的错误的出错场景，比如说某个操作可能会导致一个客户端可一理解的特定领域的原因导致的错误。(特定领域的限制意味着排除了多数情况下用户无法处理的严重错误，比如内存溢出错误)

举个例子，如果字符串不能表示适合位宽的有效数字，将它转化为数字会出错。

```swift
struct Int17 {                                 //✅
  init?(_ string: String) {
    // ...
  }
}
```

测试 Optional 是否为空但并不访问包裹的值(wrapped value)的条件语句写成是和 nil 的比较。下面的例子清楚地表明了程序员的意图：

```swift
struct Int17 {                                 //✅
  init?(_ string: String) {
    // ...
  }
}
```

Conditional statements that test that an Optional is non-nil but do not access the wrapped value are written as comparisons to nil. The following example is clear about the programmer’s intent:

```swift
if value != nil {                             //✅
  print("value was not nil")
}
```

下面的例子，采用了 Swift 的模式匹配和绑定语法，用解包该值然后立刻丢弃的方式，混淆了意图。

```swift
if let _ = value {                             //⛔️
  print("value was not nil")
}
```

#### 出错类型
当有多个可能的错误状态时使用出错类型。(译注: 本节内容在 Java 里对应的是“异常处理”)

抛出错误而不是在返回类型里包含它们，在 API 中是完全不同的考虑。有效输入和有效状态在结果域里生成有效的输出，并采用标准顺序控制流来处理。无效输入和无效状态被看作是错误，使用的是相关语法结构(do-catch 和 try)。比如：

```swift
struct Document {                             //✅
  enum ReadError: Error {
    case notFound
    case permissionDenied
    case malformedHeader
  }

  init(path: String) throws {
    // ...
  }
}

do {
  let document = try Document(path: "important.data")
} catch Document.ReadError.notFound {
  // ...
} catch Document.ReadError.permissionDenied {
  // ...
} catch {
  // ...
}
```

这种设计迫使调用者有意识地处理(acknowledge)出错案例：

+ 把调用代码塞进 do-catch 块里，然后处理出错案例到一个合适的程度
+ 将调用代码缩在的函数声明成 throws , 把错误传递出去, 或者
+ 如果出错原因不重要，只需要知道调用是否成功，那么使用 try?

一般来说，除了下面提到的例外，强制的 try! 是被禁止的; 它等同于 try 后面紧跟 fataError 而且没有有意义的信息。如果某个错误的出现意味着程序进入一个无法复原的状态，立即中止是最合理的选择，那么最好使用 do-catch 或者 try? , 然后在出错信息里提供更多的上下文帮助调试。

> **例外:** 强制的 try! 可以用于单元测试和测试代码。如果能清楚地表明，错误只可能是由于程序员出错而抛出时，可以用于非测试代码; 我们特别将其定义为可以在 Swift REPL 没有上下文的情况下可以求值单个表达式。比如，考虑用字符串字面量来初始化正则表达式：  
>   
>     let regex = try! NSRegularExpression(pattern: "a*b+c?")      //✅  
>   
> 如果正则表达式格式错误，那么 NSRegularExpression 的初始化函数会抛出错误，但它是字符串字面量，出错只可能是程序员敲错了。这里写上出错处理逻辑没有任何好处。  
>    
> 如果上面输入的模式不是字面量而是动态的或者由用户的输入生成，那么不应该使用 try! , 错误应当得到妥善处理。  

#### 强制解包和强制转换(Force Unwrapping and Force Casts)
两者都是代码的坏味道，强烈不推荐使用。除非周围代码非常清晰地表明了为什么这样的代码是安全的，否则需要一段描述了保证代码安全的不变量的注释。

```swift
let value = getSomeInteger()                      //✅

// ...intervening code...

// This force-unwrap is safe because `value` is guaranteed to fall within the
// valid enum cases because it came from some data source that only permits
// those raw values.
return SomeEnum(rawValue: value)!
```

> **例外:** 强制解包可以在单元测试和测试代码中使用，无需额外说明。这使得代码去掉了不必要的控制流。如果 nil 被解包或者类型不兼容导致转换失败，测试失败是预期的结果。  

#### (隐式解析可选类型)Implicitly Unwrapped Optionals
隐式解析可选类型(译注: 下面用 IUO 代替)本质上是不安全的，应当尽量避免使用，选择非 optional 声明或者普通的 optional 类型来代替。例外情形在下面详细描述。

使用期基于 UI 的生命周期，而不是严格基于持有对象的生命期的用户接口对象可以使用 IUO。比如带 @IBOutlet 声明并且连接到 XIB 文件或者 storyboard 的属性，在外部初始化(比如调用的 view controller 的 prepareForSegue 的实现里) 的属性，以及在 class 的生命周期里别的位置初始化的属性(比如 view controller 的 viewDidLoad 里的 views)。将这些属性实现为正常的 optional 将会为后续的解包带来过重的负担，因为它们一旦准备完毕，后续将永远为非空值。

```swift
class SomeViewController: UIViewController {     //✅
  @IBOutlet var button: UIButton!

  override func viewDidLoad() {
    populateLabel(for: button)
  }

  private func populateLabel(for button: UIButton) {
    // ...
  }
}
```

IUO 也可以在 Swift 调用没有合适的 nullability 属性修饰的 Objective-C API 时出现。可能的话，联系作者添加对应的注解(annotations)以便相关 API 更清晰地导入(import)到 Swift 。如果无法联系到代码作者，保证 IUO 的足迹(footprint)尽可能地小；也就是说，不要通过自己的抽象层来传播它们。

IUO 也允许在单元测试里使用。这和前面提到的 UI 对象场景类似，测试装置的生命周期往往不是从初始化函数开始，而是在测试的 setUp() 里，这么处理
便于在每个测试执行之前可以重置。

#### 访问权限(Access Levels)
声明里可以省略掉访问权限。最顶层的声明的默认访问权限是 internal 。内嵌的声明，默认的访问权限是 internal 和 包裹住它的声明的访问权限之间取更严格的那个。

禁止在文件级别的扩展(extension)上显式标记访问权限。如果扩展的每个成员的访问权限和默认不一样，请一一标明。

```swift
extension String {                                //✅
  public var isUppercase: Bool {
    // ...
  }

  public var isLowercase: Bool {
    // ...
  }
}
```
```swift
public extension String {                          //⛔️
  var isUppercase: Bool {
    // ...
  }

  var isLowercase: Bool {
    // ...
  }
}
```

#### 嵌套(Nesting)和命名空间(Namespacing)
Swift 允许 enum, struct, class 嵌套，所以可能的话，在类型之间表达范围和层级关系时，嵌套是更好的选择(而非命名约定)。比如，与特定类型相关的标记枚举和出错类型嵌套在该类型里面。

```swift
class Parser {                                     //✅
  enum Error: Swift.Error {
    case invalidToken(String)
    case unexpectedEOF
  }

  func parse(text: String) throws {
    // ...
  }
}
```
```swift
class Parser {                                     //⛔️
  func parse(text: String) throws {
    // ...
  }
}

enum ParseError: Error {
  case invalidToken(String)
  case unexpectedEOF
}
```

Swift 目前不允许协议在其他类型里嵌套，反之亦然。所以以上规则不适用于牵涉到协议的场景，比如控制器类和它的委托协议。

声明一个没有 case 的 enum，是定义一组相关声明(比如常量或者帮助函数)的“命名空间”的标准做法。这个 enum 自动没有实例，也不需要阻止实例化的额外样板代码。

```
enum Dimensions {                                    //✅
  static let tileMargin: CGFloat = 8
  static let tilePadding: CGFloat = 4
  static let tileContentSize: CGSize(width: 80, height: 64)
}
```
```swift
struct Dimensions {                                 //⛔️
  private init() {}

  static let tileMargin: CGFloat = 8
  static let tilePadding: CGFloat = 4
  static let tileContentSize: CGSize(width: 80, height: 64)
}
```

#### 提前退出(Early Exit)的 guard
和相反条件的 if 语句相比，guard 语句提供了视觉上的强调，被测试的条件是导致从封闭范围内提前退出的特殊情况。

此外，guard 语句通过消除额外的嵌套层级("pyramid of doom", 即过深层级带来的缩进形成的金字塔)提高了可读性；出错条件和触发它们的条件互相临近，主逻辑在其范围内保持齐平。

这可以从下面的例子里看出来。首先有个清晰的过程来检查无效状态以便退出，然后在成功的情况下执行主逻辑。在没有使用 guard 的第二个例子里，主逻辑被埋在一个随意的嵌套等级上，抛出错误的语句和它们对应的条件距离很远。

```
func discombobulate(_ values: [Int]) throws -> Int { //✅
  guard let first = values.first else {
    throw DiscombobulationError.arrayWasEmpty
  }
  guard first >= 0 else {
    throw DiscombobulationError.negativeEnergy
  }

  var result = 0
  for value in values {
    result += invertedCombobulatoryFactory(of: value)
  }
  return result
}
```
```swift
func discombobulate(_ values: [Int]) throws -> Int { //⛔️
  if let first = values.first {
    if first >= 0 {
      var result = 0
      for value in values {
        result += invertedCombobulatoryFactor(of: value)
      }
      return result
    } else {
      throw DiscombobulationError.negativeEnergy
    }
  } else {
    throw DiscombobulationError.arrayWasEmpty
  }
}
```

guard-continue 语句在循环里也十分有用，当整个循环体只需要在部分条件下执行时, 它可以避免增加缩进(但也请看下面的 for-where 讨论)。

#### for-where 循环
当整个 for 循环体都被一个 if 条件包裹住，而条件是测试每一个元素的，那么这个测试会被放到 for 语句之后的 where 从句里。

```
for item in collection where item.hasProperty { //✅
  // ...
}
```
```swift
for item in collection {                          //⛔️
  if item.hasProperty {
    // ...
  }
}
```

#### switch 里的 fallthrough
当 switch 有多个 case 是执行相同代码时，case 模式被合成一个范围或者逗号分隔的列表。多个 case 分支什么也不干仅仅跌落(fallthrough)到下面一条 case 分支是不允许的。

```
switch value {                                   //✅
case 1: print("one")
case 2...4: print("two to four")
case 5, 7: print("five or seven")
default: break
}
```
```swift
switch value {                                 //⛔️
case 1: print("one")
case 2: fallthrough
case 3: fallthrough
case 4: print("two to four")
case 5: fallthrough
case 7: print("five or seven")
default: break
}
```

换句话说，不允许某个 case 分支仅包含 fallthrough 语句。case 分支包含额外语句然后 fallthrough 到接下来的 case 分支是允许的。

#### 模式匹配(Pattern Matching)
关键字 let 和 var 分别放置在每一个要被匹配的元素前面。在整个模式之前和中间分布的 let/var 的简写形式是禁止的，因为如果模式里被匹配的值本身也是个变量的话，会导致非预期的行为。

```
enum DataPoint {                                 //✅
  case unlabeled(Int)
  case labeled(String, Int)
}

let label = "goodbye"

// 下面的 `label` 前面没有 `let`, 它被看作是一个值，所以
// 下面的模式仅匹配标签为 “goodbye” 的 DataPoint
switch DataPoint.labeled("hello", 100) {
case .labeled(label, let value):
  // ...
}

// 在每个单独的绑定前面写上 `let` 能清楚地表达意图是
// 引入一个新的绑定(这个例子里覆盖掉(shadowing)局部变量)
// 而不是匹配局部变量的值. 因此下面的模式匹配
// 带任意标签的 DataPoint
switch DataPoint.labeled("hello", 100) {
case .labeled(let label, let value):
  // ...
}
```

下面的例子里，如果作者的意图是使用上面 label 变量的值来做匹配，那么已经丢失了，因为 let 分布在整个模式中，用匹配任意字符串值的绑定的含义，覆盖了取变量值的含义。(译注: 总有歧义，要避免，不清晰)

```swift
switch DataPoint.labeled("hello", 100) {      //⛔️
case let .labeled(label, value):
  // ...
}
```

多元组(tuple)参数的标签和枚举(enum)关联值在绑定时，如果绑定值的变量名和标签同名，那么标签和关联值可以省略。

```
enum BinaryTree<Element> {                   //✅
  indirect case subtree(left: BinaryTree<Element>, right: BinaryTree<Element>)
  case leaf(element: Element)
}

switch treeNode {
case .subtree(let left, let right):
  // ...
case .leaf(let element):
  // ...
}
```

此时保留标签是增加冗余且无用的噪音。

```swift
switch treeNode {                              //⛔️
case .subtree(left: let left, right: let right):
  // ...
case .leaf(element: let element):
  // ...
}
```

#### 多元组模式(Tuple Patterns)
只有当赋值运算符左边没有标记(unlabeled)时，才允许使用多元组模式(有时候称作 tuple shuffle)来给变量赋值。

```
let (a, b) = (y: 4, x: 5.0)                     //✅
```
```swift
let (x: a, y: b) = (y: 4, x: 5.0)               //⛔️
```

左侧的标签和类型标注非常相似，容易导致误解。

```swift
                                                //⛔️
// 这里声明了两个变量，名为`Int`, 实际上是值为 5.0 的`Double`类型；
// 以及名为`Double`, 实际上是值为 4 的`Int`类型.
// `x` 和 `y` 不是变量
let (x: Int, y: Double) = (y: 4, x: 5.0)
```

#### 数字和字符串字面量
Swift 的整数和字符串没有固有类型。比如说，5 本身不是 Int 类型；它是一个特殊的字面值，可以被转换成满足 ExpressibleByIntegerLiteral 的任意类型，如果类型推导(type inference)没有进一步将它映射到更具体的类型时它才是 Int 。类似的，“x” 不是 String 也不是 Character或者 UnicodeScalar，但是它可以根据上下文转化为三者之一，默认返回 String。

因此，如果用字面量来初始化非默认值的类型的值，而且类型无法通过上下文来推断，那么在声明里显式指定类型或者用 as 表达式强制指定。

```swift
                                             //✅
// 没有更显式的指定类型, x1 将被推断为 Int 类型
let x1 = 50

// 显式指定为 Int32 类型
let x2: Int32 = 50
let x3 = 50 as Int32

// 没有更显式的指定类型, y1 将被默认推断为 String 类型
let y1 = "a"

// 显式指定为 Character 类型
let y2: Character = "a"
let y3 = "a" as Character

// 显式指定为 UnicodeScalar.
let y4: UnicodeScalar = "a"
let y5 = "a" as UnicodeScalar

func writeByte(_ byte: UInt8) {
  // ...
}
// 函数参数也会做类型推断，没有显式强制指定，50 就是 UInt8 类型
writeByte(50)
```

比如，一个数字无法转为整型或者一个多字节的字符串无法强制转为一个字符，编译器会为这种无效字面量强制转换给出合适的出错信息。所以下面的示例里出错了，是“好”的行为，因为错误是在编译期间以正确的原因捕捉到的(kurk注: 编译器没中文版的，下面出错信息不翻了)。

```swift
                                             //✅
// error: integer literal '9223372036854775808' overflows when stored into 'Int64'
let a = 0x8000_0000_0000_0000 as Int64

// error: cannot convert value of type 'String' to type 'Character' in coercion
let b = "ab" as Character
```

使用初始化语法来强制指定这些类型可能导致误导的编译器错误，或者更糟，导致难以调试的运行时错误。

```swift
                                                //⛔️
// 下面先会从字面量转为带符号的 `Int` 类型，然后转为 `UInt64`
// 尽管这个字面量可以直接转为 `UInt64`, 但是第一步转
// `Int` 失败了，所以通不过编译.
let a1 = UInt64(0x8000_0000_0000_0000)

// 下面会调用 `Character.init(_: String)`, 
// 因此在运行时(涉及慢速的堆内存)创建了 `String` 类型的 "a"
// 从它里面解析出字符，然后再把它释放掉。
// 这明显比合理的类型强制指定要慢。
let b = Character("a")

// 如上所述, 这里先创建 `String` 然后调用 `Character.init(_: String)`
// 尝试从中解析出单个字符。这让先决条件检查失效并且造成运行时陷阱。
let c = Character("ab")
```

#### Playground Literals
非 playground 的生产代码里面禁止使用渲染成图形的 playground 字面量，比如  #colorLiteral(...), #imageLiteral(...),  和 #fileLiteral(...) 。在 playground 代码里它们允许使用。

```swift
let color = UIColor(red: 1.0, green: 1.0, blue: 1.0, alpha: 1.0)  //✅
```
```swift
let color = #colorLiteral(red: 1.0, green: 1.0, blue: 1.0, alpha: 1.0) //⛔️
```

#### 算术的陷阱(Trapping) 和溢出
标准(溢出触发陷阱)算术和位运算符(+, -, *, <<, >>)用于绝大多数正常操作，而不是掩码操作(以&开头)。陷阱溢出更安全，因为它阻止了坏数据通过系统的其他层传播出去。(译注: trap 是中断，会导致程序退出啊！三思！)

```swift
// GOOD. 溢出不会导致存款余额 newBankBalance 为负数.//✅
let newBankBalance = oldBankBalance + recentHugeProfit  
```
```swift
// AVOID/避免. 如果被加数过大，溢出导致 newBankBalance 为负数  //⛔️
let newBankBalance = oldBankBalance &+ recentHugeProfit
```

掩码操作相对较少，但在使用取模运算的问题域(如加密算法，大整数实现，散列函数等)中是允许的(实际上对于正确性而言是必要的)。

```swift
var hashValue: Int {   //✅
  // GOOD. 这里起作用的是位模式的分布而不是实际的数值
  return foo.hashValue &+ 31 * (bar.hashValue &+ 31 &* baz.hashValue)
}
```
```swift
var hashValue: Int {   //⛔️
  // 不正确. 这会根据每一项的哈希值任意地、不可预测地触发陷阱(中断)
  return foo.hashValue + 31 * (bar.hashValue + 31 * baz.hashValue)
}
```

性能敏感的代码中也是允许使用掩码操作的，如果能确认其中的值不会造成溢出(或者溢出不是问题)。 在这种情况下，应该使用注释来说明为什么使用掩码操作很重要。 另外，考虑添加调试前提条件来检查这些假设，而不会影响优化后的构建版本的性能。

#### 定义新的运算符(operators) <a name="defining-new-operators">
当不明智地使用时，自定义运算符会显著降低代码可读性，因为它们经常缺少那些被编译进标准库里的更通用的运算符的历史背景。

一般来说，应该避免自定义运算符。 但是，当运算符在问题域中具有清晰明确的含义，并且使用运算符相比函数调用显著提高了代码的可读性时，它是允许的。 例如，由于 * 是由 Swift 定义的唯一乘法运算符(不包括掩码版本)，因此数学矩阵库可以定义其他运算符来支持其他运算，如叉乘和点乘。

禁用的一个例子是自定义 <~~ 和 ~~> 运算符来解码和编码 JSON 数据。这些运算符对于处理 JSON 数据的问题域来说不是原生的，在不查阅相关文档的情况下即使是有经验的 Swift 工程师也无法理解代码的目的。

如果你必须采用通过自定义运算符提供 API 的第三方代码，而这些代码的价值毋庸置疑，那么**强烈推荐**你考虑实现一个定义了更加可读的方法的外包类(wrapper)，将方法委托(delegate)给这些自定义运算符。这会显著减缓新同事和其他代码审查者对这些代码的学习曲线。

#### 运算符重载(Overloading Existing Operators)
当你对运算符的使用在语意上等价于在标准库里对它的使用时，可以对运算符重载。允许使用的例子是满足 Equatable 和 Hashable 时对运算符的要求的实现，或者定义一个支持算术运算的新的矩阵类型。

如果你希望重载一个运算符时赋予它不同于自然含义的新含义，请按照 [定义新的运算符](#defining-new-operators) 里的说明来决定哪些是允许的。也就是说，如果新的含义在问题域已经建立而且运算符的使用相比其他语法结构更能提高可读性，那么它就是允许的。

禁止重新赋予运算符含义的一个例子是，重载 * 和 + 来建立一个临时的正则表达式 API。和简单地将整个正则表达式表示为字符串相比，这种 API 并没有提供足够多的代码可读性方面的收益。

### 文档注释(Documentation Comments)
#### 通用格式
文档注释的每一行以三个斜杠(///)开头。不允许使用 Javadoc 风格的评论块(/** ... */)

```swift
                                                       //✅
/// Returns the numeric value of the given digit represented as a Unicode scalar.
///
/// - Parameters:
///   - digit: The Unicode scalar whose numeric value should be returned.
///   - radix: The radix, between 2 and 36, used to compute the numeric value.
/// - Returns: The numeric value of the scalar.
func numericValue(of digit: UnicodeScalar, radix: Int = 10) -> Int {
  // ...
}
```
```swift
                                                         //⛔️
/**
 * Returns the numeric value of the given digit represented as a Unicode scalar.
 *
 * - Parameters:
 *   - digit: The Unicode scalar whose numeric value should be returned.
 *   - radix: The radix, between 2 and 36, used to compute the numeric value.
 * - Returns: The numeric value of the scalar.
 */
func numericValue(of digit: UnicodeScalar, radix: Int = 10) -> Int {
  // ...
}

/**
Returns the numeric value of the given digit represented as a Unicode scalar.

- Parameters:
  - digit: The Unicode scalar whose numeric value should be returned.
  - radix: The radix, between 2 and 36, used to compute the numeric value.
- Returns: The numeric value of the scalar.
*/
func numericValue(of digit: UnicodeScalar, radix: Int = 10) -> Int {
  // ...
}
```

#### 一句话摘要(Single-Sentence Summary)
文档注释以描述声明的简短一句话摘要开始。(这句话可以分成多行，但如果分成太多行，作者应当考虑是不是摘要可以进一步简化，细节放到新段落里去。)

如果除了摘要的说明以外还需要更多的细节，可以在后面添加新的段落(用一个空行隔开)。

一句话摘要不必是完整的句子；比如方法摘要一般写成省略了主语(这个方法如何如何)的动词短语，因为主题很显然，写出来冗余。同样的，属性摘要通常是省略了主语(这个属性如何如何)的名词短语。然而所有情况下，它们都是以句号(.)结尾。

```swift
                                                      //✅
/// The background color of the view.
var backgroundColor: UIColor

/// Returns the sum of the numbers in the given array.
///
/// - Parameter numbers: The numbers to sum.
/// - Returns: The sum of the numbers.
func sum(_ numbers: [Int]) *->* Int {
  // ...
}
```
```swift
                                                      //⛔️
/// This property is the background color of the view.
var backgroundColor: UIColor

/// This method returns the sum of the numbers in the given array.
///
/// - Parameter numbers: The numbers to sum.
/// - Returns: The sum of the numbers.
func sum(_ numbers: [Int]) *->* Int {
  // ...
}
```

#### Parameter, Returns, Throws 标记
用 Parameter(s) , Returns, 和 Throws 标记和这样的顺序来清楚地记录参数，返回值以及错误抛出。不会出现空的描述。当一行放不下一个描述时，连续行将从连字符开始标记的位置缩进2个空格。

在Xcode中编写文档注释的推荐方法是将光标置于声明上，然后按**Command + Option + /**。这将自动生成正确的格式，在占位符位置继续输入即可。

当一句话摘要已经完全说明了参数和返回值的含义，添加相依的标签只能重复已有的说明时，Parameter(s) 和 Returns 标签可以省略。

Parameter(s), Returns, Throws 标签后的内容以句号结尾，即使它们不是完整的句子而是短语。

如果参数只有一个，那么使用单数单行形式的 Parameter 标签。当参数有多个，那么使用分组的复数的 Parameters 形式，每个参数以它的名称为标签，写成嵌套列表里的一项。

```swift
                                                      //✅
/// Returns the output generated by executing a command.
///
/// - Parameter command: The command to execute in the shell environment.
/// - Returns: A string containing the contents of the invoked process's
///   standard output.
func execute(command: String) -> String {
  // ...
}

/// Returns the output generated by executing a command with the given string
/// used as standard input.
///
/// - Parameters:
///   - command: The command to execute in the shell environment.
///   - stdin: The string to use as standard input.
/// - Returns: A string containing the contents of the invoked process's
///   standard output.
func execute(command: String, stdin: String) -> String {
  // ...
}
```

下面的例子是错误的，因为它们为单一参数使用了复数形式而多个参数的情况下使用了单数形式。

```swift
                                                      //⛔️
/// Returns the output generated by executing a command.
///
/// - Parameters:
///   - command: The command to execute in the shell environment.
/// - Returns: A string containing the contents of the invoked process's
///   standard output.
func execute(command: String) -> String {
  // ...
}

/// Returns the output generated by executing a command with the given string
/// used as standard input.
///
/// - Parameter command: The command to execute in the shell environment.
/// - Parameter stdin: The string to use as standard input.
/// - Returns: A string containing the contents of the invoked process's
///   standard output.
func execute(command: String, stdin: String) -> String {
  // ...
}
```

#### Apple’s Markup Format
强烈推荐使用 [苹果的标记格式](https://developer.apple.com/library/archive/documentation/Xcode/Reference/xcode_markup_formatting_ref/)为文档提供丰富的格式。这些标记有助于区分符号引用(比如参数名称)和注释里的描述性文本，而且可以被 Xcode 和其他文档生成工具渲染出来。下面列出的是一些常用的指示:

* 段落用以三个斜线(///)开头的空行分隔
* \**一对单个星号*\* 和 \__一对单下划线_\_  包住一段斜体/倾斜的文本
* **\*\*****一对双星号****\*\*** 和 \_\___一对双下划线__\_\_ 包住一段粗体文本
* 符号名称和行内的代码放在 \``反引号`\`里.
* 多行代码 (比如示例代码) 的前后各加三个反引号，形成一个代码块

#### 哪里需要文档
至少，每个 open 和 public 的声明，和这种声明里每个 open 和 public 的成员都需要提供文档注释，例外情况如下：

+ enum 的每个独立 case 如果名称已经做了完善的自我介绍，就不用再提供注释了。但是，使用关联值的 case ，如果值不是那么明显，就需要记录它的含义。
+ 覆盖超类型声明或者实现一个协议要求，或者在扩展里提供了协议要求的默认实现的声明上，文档注释不是必须出现。

  可以将覆盖的声明记录为描述它覆盖掉的声明的新行为。 在任何情况下，覆盖声明的文档注释都不应该仅仅是被覆盖的声明的副本。

+ 测试类和测试方法上文档注释不是总会出现。但是它们在功能测试类和多个测试共享的帮助类/帮助方法上很有用
+ 扩展声明里(即扩展本身)文档注释不是总会出现。如果有助于澄清扩展的用途，你可以添加一个，但是避免无意义和误导说明。
  
 下面的例子里，注释只是重复了代码里很明显的内容：

  ```swift
  /// Add `Equatable` conformance.                   //⛔️
  extension MyType: Equatable {
    // ...
  }
  ```

  下一个例子更加微妙，但它是一个不可延展(scalable)的文档示例，因为扩展(extension)或者符合性(conformace)可以在将来更新。考虑到在编写时可能会将类型设置为 Comparable，以便对值进行排序，但这不是该符合性的唯一可能用途，并且客户端代码将来可能会将其用于其他用途。

  ```swift
  /// Make `Candidate` comparable so that they can be sorted.  //⛔️
  extension Candidate: Comparable {
    // ...
  }
  ```

一般来说，如果你发现自己编写的文档只是简单地重复来自源代码的明显信息，并用诸如“表示”之类的词语加以描述，那么完全可以不要这些注释。

但是，引用这个例外来证明忽略典型读者可能需要知道的相关信息是*不*合适的。 例如，对于一个名为canonicalName的属性，如果一个典型的读者可能不知道术语“canonical name”在上下文的含义是什么，那么不要省略它的文档(以“它只会说 ///The canonical name”的理由)。 使用文档作为定义该术语的机会。


[原始英文链接](https://google.github.io/swift/)

感谢 [Swift 老司机](https://github.com/SwiftOldDriver) 带路