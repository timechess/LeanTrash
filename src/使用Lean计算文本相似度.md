# 使用Lean计算文本相似度

## Introduction

本文主要希望用Lean计算文本相似度作为例子，对Lean中的structure、class、instance等关键词与其应用做一个简要的阐释，同时也希望体现从抽象模型，到抽象代码，再到具体实现的程序设计流程。本文面对的对象并非Lean小白，但学了几天应当就够了。

本节主要先对刚刚所提到的关键词做一个简单介绍。在C中，我们有结构体 `struct`，用于告诉编译器，我们将几个数据类型的组合视为一个整体并给它一个新的名字。而在C++、Python等可以面向对象编程的语言中，类除了数据字段还能有函数字段，也就是类的方法，并且出现了多态、继承等机制。在Rust中，数据与方法又被解耦，结构体与特征独立开来，采用了组合优于继承的设计思想。

在Lean中同样有着上述语言中类似的特性，但又有许多微妙的不同。如果我们只是想把一些数据拼起来，比如我们想描述Lean中的定理，一个定理有名字、参数列表、输出类型以及证明字段，我们暂且将它们都视为字符串，那我们可以这么写：

```lean
structure Theorem where
  name  : String
  args  : Array String
  type  : String
  proof : String
```

如果我们想让它像Python中的类一样有一些独有的方法，我们一般不需要在结构声明中进行定义，而只需要这么写：

```lean
def Theorem.print (t : Theorem) : String :=
    s!"{t.name} {t.args.foldr (fun init arg => init ++ arg) ""} : {t.type}"
```

加入 `Theorem` 前缀并让其参数中带有一个 `Theorem`，那么如果 `t` 是一个 `Theorem` 的实例，你就能够通过 `t.print` 调用该函数，直接输出需要的字符串。

当然我们也可以在结构声明中加入函数类型，但由于结构声明中的字段在实例化时都需要进行单独实现，并不符合Python类中方法的特征。这一特性在定义许多数学对象时有着很大的作用，比如定义群时，同一个集合上可以有不同的群结构，也就是不同的二元运算，故对每个 `Group` 实例都需要单独给定一个二元运算，符合上述特性。

Lean作为定理证明器，相较于之前提到的语言有着独有特性，即命题也可以作为数据字段，比如我要求定理的名字长度不超过20，就可以加如下字段：

```lean
structure Theorem where
  name           : String
  args           : Array String
  type           : String
  proof          : String
  name_constrain : name.length ≤ 20
```

这样当你实例化一个 `Theorem` 时，就需要提供这一限制命题的证明，这保证了所有 `Theorem` 实例都满足这一性质。如果你不想每次都提供这个证明，因为任何 `Theorem` 实例的该限制命题证明都只需要 `by decide` 即可，那么你可以为该字段设置默认值，即：

```lean
structure Theorem where
  name           : String
  args           : Array String
  type           : String
  proof          : String
  name_constrain : name.length ≤ 20  := by decide
```

你可以用 `def` 来声明一个 `Theorem` 实例，如下：

```lean
def t : Theorem := {
  name := "Nat.mul_comm"
  args := #["(a b : Nat)"]
  type := "a * b = b * a"
  proof := "trivial"
}
```

注意到我们没有提供限制命题的证明，因为我们的默认值完成了证明，故Lean也没有抱怨。下面我们测试一下刚刚的方法：

```lean
#eval t.print  -- "Nat.mul_comm (a b : Nat) : a * b = b * a"
```

相比直接 `#check Nat.mul_comm`，输出除了参数名不一样，也就是多了两个引号。

在Lean中一个类型可以依赖于另一个类型，实际上也就达成了类似泛型的效果。比如，我们定义一个二维空间中的点，但该点的坐标可以是实数，可以是有理数，也可以是自然数，那么我们使用这个点时还需要看它坐标的类型。故我们可以这么写：

```lean
structure Point (α : Type) where
  x : α
  y : α
```

敏感的读者可能会觉得，此处对 `α` 没有任何限制不太安全，后续可能会存在一些问题。实际上确实，当我们想要为这个结构实现加法、乘法时，就会发现对 `α` 需要有一些要求，即它本身需要有加法和乘法。熟悉Rust的读者可能会想，在定义时要求 `α` 具有对应的特征即可。因为加法、乘法这种结构十分常见，并且与 `+`、`*` 等运算符绑定，故Lean中一定有相应的实现，实际上也确实有，但我们暂且按下不表。

刚刚我们提到其他语言中的继承与多态，Lean中也有类似机制，比如我们希望将刚刚的点扩充到三维，可以这么写：

```lean
structure Point3D (α : Type) extends Point α where
  z : α
```

这样 `Point3D` 依然会有 `x y` 两个数据字段，但 `Point3D` 并不是 `Point`，一个接收 `Point` 作为参数的函数不能接收 `Point3D`，但你可以通过 `Point3D.toPoint` 把 `Point3D` 显式转换为 `Point`，或者实现一个 `Coe` 实例让Lean能够进行隐式类型转换，但后者就不在本文多说了。

讲了这么久structure，可能有人会疑惑，既然structure已经有了这么多特性，那么class和instance又是用来做什么的？事实上，之前例子中所有structure出现的地方，都可以直接替换成class，在上述用途中这两个是等价的。

class能够实现的是我们刚刚略过不提的部分，即如何说一个类型有着加法和乘法，用Rust术语就是某个结构体有着相应的特征。让我们把思维从刚刚过于具体的例子抽象出来，考虑一个抽象的结构，这个结构只有一个函数字段，即给定类型的加法，如下：

```lean
class Add (α : Type) where
  add : α → α → α
```

因为Lean标准库中已经有了这一类，故如果你输入到Lean文件中，Lean会抱怨重复定义。这一 `add` 函数被绑定到了 `+` 这一运算符上，因此对于 `Add` 的实例，使用 `a + b` 这样的语句实际上是调用了实例实现时的 `add` 函数。而当你希望在声明 `Point` 时要求 `α` 是 `Add` 的实例，可以用以下的语法：

```lean
structure Point (α : Type) [Add α] where
  x : α
  y : α
```

中括号中的 `Add α` 表示 `α` 是 `Add` 的实例，或者说 `α` 上实现了加法结构。这同样也可以是一个函数的参数，与小括号声明的显式参数以及花括号声明的隐式参数不同，实例参数并没有一个可供调用的名字，同时你在调用函数时也不需要提供这一参数，Lean会自动搜索该实例的实现。在Rust中，这可以被理解为特征，只不过Lean中的特征并没有换一个关键词来声明。

要实现 `Point` 的加法，只需用instance关键词进行声明：

```lean
instance {α : Type} [Add α] : Add (Point α) where
  add := fun p₁ p₂ => ⟨p₁.x + p₂.x, p₁.y + p₂.y⟩
```

这样，当你尝试用 `+` 计算两个点的和时，Lean会自动分析你的这两个点对应的 `α`，然后搜索 `α` 的 `Add` 实现，如果搜索到了，则调用这里实现的加法，否则报错说找不到 `Add α` 实例。

这一实例搜索的特性是class、instance独有的，而structure无法实现。

到这儿，实现算法需要的前置Lean背景知识就够了，虽然其实这一任务中难度最高的应该是函数式编程相关内容，但解释起来过于麻烦，就假设读者都会了吧。

## Design

首先我们需要对文本相似度计算这一任务有一个抽象层次的认识。我们大体可以把相似度计算方式分为两类，一是给定两篇独立的文档即可直接计算，二是需要一个文档库作为基础才能计算。前者的典型例子是当下基于深度学习将文本直接编码为低维语义向量然后计算向量相似度，后者的典例是TF-IDF算法，计算逆文档频率时需要有文档库支撑。

> TF-IDF虽然本身并不能计算文本相似度，但我们总可以进行一些自定义操作来达到使用它的目的，俗称为了这盘醋包的饺子，比如用两个文档前10个关键词的重合度来表示相似度。

不管是哪种计算方式，我们都可以定义基础数据类 `Document`，来表示一个文档，其中包含文档的内容和一些元信息，取决于我们的数据集是什么。而对于第一种计算方式，要求文档之间可以可以直接计算相似度，即 `Document` 需要具备“可与另一个文档计算相似度”的特征。对于第二种计算方式，我们需要一个 `Corpus` 对象用于存储对象，同时需要一种相似度计算算法，在文档之间无法直接计算相似度时也能使用。

首先我们可以定义可计算相似度的特征如下：

```lean
class CalcSim (α : Type) where
  sim : α → α → Float
```

如果文档具有这种特征，那么一种naive的计算相似度的方法即直接调用该方法：

```lean
def naiveSim {α : Type} [CalcSim α] (d₁ d₂ : α) : Float :=
  CalcSim.sim d₁ d₂
```

`Corpus` 包含文档以及相似度计算算法，即：

```lean
class Corpus (α : Type) where
  docs : Array α
  sim  : α → α → Float
```

我们希望对于实现了 `CalcSim` 特征的文档默认相似度计算方式为上述 `naiveSim`，此时不能直接在class定义中 `:= nativeSim`，因为Lean找不到 `CalcSim` 实例，但我们可以通过这样写达成同样的效果：

```lean
class Corpus (α : Type) where
  docs : Array α
  sim  : α → α → Float := by exact naiveSim
```

此时Lean会在构造 `Corpus` 时尝试使用该证明，如果 `α` 在实例化后没有实现 `CalcSim`，Lean才会报错，而不是在定义时就报错。

我们希望能用多种方式从 `Corpus` 中索引文档，不仅是用数组的索引，那么我们可以顶以一个 `Find` 特征如下：

```lean
class Find (α β : Type) where
  find : Corpus α → β → Option α

def Corpus.find? {α β : Type} [Find α β] (c : Corpus α) (q : β) : Option α := Find.find c q
```

其中 `β` 为索引使用的值的类型，定义 `Corpus.find?` 后即可直接使用类似 `c.find? q` 的语法来进行查询。这样我们可以通过实例化多个 `Find` 类来让 `Corpus` 的查询支持多种类型的输入。

这便是我们需要的所有基础设施了，代码的抽象模型搭建完成，下面我们具体化我们想要完成的任务。

## Implementation

首先我们花半分钟收集数据，打开https://nips.cc/virtual/2024/papers.html?filter=titles，点击 `Ctrl+Shift+I` 打开开发面板，在Console中找到一个大小为4542的Array，右键选择 `Copy Object`，创建个新的json文件粘贴。

好，2024年NIPS接收的所有论文的元数据爬取完毕。

根据数据格式，我们可以定义 `Document` 类：

```lean
structure Document where
  id  : Nat
  UID : String
  title : String
  authors : Array String
  keywords : Array String
  topic : Option String
  abstract : String
deriving Repr, ToJson, FromJson, Inhabited
```

部分无关信息不在类中定义了。`deriving` 的一些特征是为了便于读取与展示，在此不多做说明。

简单写个读取json文件的逻辑：

```lean
open Lean

def readData (path : String) : IO (Array Document) := do
  let file ← IO.FS.readFile path
  let json := Json.parse file
  match json with
  | .error e => throw <| IO.userError <| toString <| ("Could not parse JSON:\n" ++ e)
  | .ok j =>
    let docs := fromJson? (α := Array Document) j
    match docs with
    | .ok docs  => pure docs
    | .error e =>
        println! s!"{e}"
        pure #[]

def main (args : List String) : IO Unit := do
  match args with
  | [path] =>
    let docs ← readData path
    println! s!"{docs.map (fun doc => doc.title)}"
  | _ => throw $ IO.userError "Invalid arguments"
```

> 实际上这步在Lean缺乏足够文档指引与代码样例的情况下还挺费劲。

我们希望使用TF-IDF算法找到每篇文献摘要中的关键词，使用前十关键词重合度作为文献相似度。最终的可执行文件接收论文标题或ID作为输入，返回最相近的文献。

首先对我们在之前小节定义的一些抽象类进行实例化，这里我们默认两篇文献直接计算相似度的值为较小标题长度与较长标题长度的比值。可以用论文的数值id或标题进行索引，其中标题索引只要求输入为实际标题的前缀。

```lean
instance : Find Document Nat where
  find := fun c q => c.docs.find? (fun doc => doc.id == q)

instance : Find Document String where
  find := fun c q => c.docs.find? (fun doc => q.isPrefixOf doc.title)

instance : CalcSim Document where
  sim := fun d₁ d₂ =>
    (min d₁.title.length.toFloat d₂.title.length.toFloat) /
    (max d₁.title.length.toFloat d₂.title.length.toFloat)
```

用这一naive的实现先把完整逻辑测试跑通，下面是搜索的逻辑：

```lean
def Corpus.search {α β : Type} [Inhabited α] [Find α β] (c : Corpus α) (q : β) (k : Nat := 10) : Array α :=
  match c.find? q with
  | some doc =>
    let similarities := (c.docs.zip (Array.range c.docs.size)).map (fun ⟨d, i⟩ => (⟨c.sim d doc, i⟩ : Float × Nat))
    let sorted_similarities := similarities.qsort (fun s₁ s₂ => s₁.1 > s₂.1)
    let slice := min k sorted_similarities.size
    sorted_similarities[:slice].toArray.map (fun ⟨_, i⟩ => c.docs[i]!)
  | none => #[]
```

这样主函数便可以定义如下：

```lean
def main (args : List String) : IO Unit := do
  match args with
  | [path, query] =>
    let docs ← readData path
    let corpus := Corpus.mk docs
    let topk := corpus.search query
    for doc in topk do
      println! s!"{doc.title}"
  | _ => throw $ IO.userError "Invalid arguments"
```

若需要更改相似度计算方式，只需将其中的 `corpus` 换成使用另一种相似度计算方式的Corpus即可。

下面我们实现TF-IDF算法，注意这种算法naive的实现效率极低，以下实现仅为参考，若要加速至少需要添加缓存与持久化存储机制。具体算法细节本文就不提了，读者可以自行搜索。

首先是词频的统计：

```lean
def Document.termFrequency (d : Document) : Std.HashMap String Float :=
  let words := d.abstract.trim.splitOn
  let insertOrCreate (m : Std.HashMap String Nat) (w : String) (c : Nat) : Std.HashMap String Nat :=
    match m.get? w with
    | some count => (m.erase w).insert w (c + count)
    | none => m.insert w c
  words.foldr (fun w m => insertOrCreate m w (words.count w)) {}
    |>.map (fun _ c => c.toFloat / words.length.toFloat)
```

由于Lean的函数式特性，创建词频字典的操作看起来很蠢。当然可以使用State Monad等方法对此进行优化，但为了节省脑力这里就不写了。

然后是逆文档词频：

```lean
def inverseDocumentFrequency (docs : Array Document) (word : String) : Float :=
  let contain_docs := docs.filter (fun d => d.abstract.trim.splitOn.contains word)
  Float.log <| contain_docs.size.toFloat / (docs.size.toFloat + 1)
```

这就是可以通过缓存来优化的部分，如果不进行优化，会出现大量的重复计算，让搜索时延抵达无法接受的地步。当然，在这里依然不做优化。

最后，实现我们的TF-IDF Corpus实例：

```lean
instance TF_IDF_Corpus (docs : Array Document) (k : Nat) : Corpus Document where
  docs := docs
  sim := fun d₁ d₂ =>
    let tf_idf₁ := d₁.termFrequency.map
      (fun w tf => tf * inverseDocumentFrequency docs w)
      |>.toArray.qsort (fun w₁ w₂ => w₁.2 > w₂.2)
    let tf_idf₂ := d₂.termFrequency.map
      (fun w tf => tf * inverseDocumentFrequency docs w)
      |>.toArray.qsort (fun w₁ w₂ => w₁.2 > w₂.2)
    let slice := min k (min tf_idf₁.size tf_idf₂.size)
    let topk₁ := tf_idf₁[:slice].toArray.map (fun ⟨w, _⟩ => w)
    let topk₂ := tf_idf₂[:slice].toArray.map (fun ⟨w, _⟩ => w)
    let intersection := topk₁.filter (fun w => topk₂.contains w)
    intersection.size.toFloat / slice.toFloat
```

主函数实现如下：

```lean
def main (args : List String) : IO Unit := do
  match args with
  | [path, query] =>
    let docs ← readData path
    let corpus := TF_IDF_Corpus docs 10
    let topk := corpus.search query
    for doc in topk do
      println! s!"{doc.title}"
  | _ => throw $ IO.userError "Invalid arguments"
```

在上面的例子中，相似度为两个文章摘要TF-IDF值最高的10个词重合的比例。

读者可以根据以上代码自行组织项目尝试运行，以上代码在4.13.0的Lean中进行测试，后续版本可能存在一定API的变化，不保证能够适配。另外正如先前所说，目前的TF-IDF代码未经优化，效率极低，如果使用4542篇NIPS论文进行测试，基本跑不出来，建议缩小数据集进行测试，或自行优化。