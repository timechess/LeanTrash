# Lean的非形式化入门指南

## Introduction

在开始本文之前，先简单为不知道Lean是什么的读者介绍一下什么是Lean。简单来说，Lean是一门函数式编程语言。先别忙着退出，因为真的拿Lean去做刻板印象中程序员做的工作的人极少，并且Lean也不是为了做软件开发之类的工作而被设计出来的。实际上，它更多地被用于定理证明。

提到定理证明，大多数人可能联想到的就是教科书中一条条冰冷定理下的复杂证明，作者用晦涩深奥的语言颠来倒去，最后声称自己证明了上述定理，步骤中可能还充斥着令人不快的“显然”“易证”以及混沌邪恶的“留给读者作为练习”。事实上，对于一些并不是很懂语言艺术的数学家，人们往往会觉得他们不说人话。终于，有人决定彻底不讲人话了，与其让台下昏昏欲睡的学生听懂自己深奥的思想，不如用计算机能懂的语言讲给计算机听。

当然以上纯属戏言，数学家不是因为没人听得懂他们讲话而发明Lean的。实际上，Lean有着非常严肃的用途。在说明它的用途之前，不妨先思考这样几个问题：假如你是个数学家，研究着最前沿的数学问题，同领域中与你水平相近的大咖只有寥寥数人。某一天，你认为自己证出了一个世纪难题，但你生性谨慎，没法断言自己的证明是正确的，想要为自己近百页的论文找到够格的读者很难，找到的读者会不会也漏掉里面存在的错漏也是个问题，如何让同行理解并接受你的证明更是难上加难。另外，你的证明中引用了几个别人的结论，他们的论文说实话你还没有细看，里面会不会有问题尚不可知。哪怕你真的是正确的，想要确认这一点需要花多久呢？

程序员都知道，写代码容易debug难，想要从所谓的屎山里找到问题所在更难。而数学证明的debug难度则更上一层楼，毕竟程序真的会给你报错，但证明不会。已有数学文献中难道就不存在错误的证明？只能说有，而且还不少，但想找出这些错误几乎是不可能的工作。

读到这里想必读者也能推断出Lean是用来干嘛的了。人类需要经过大量训练才能做的验证纠错工作，交给计算机属于是专业对口。只需要有一门有着跟自然语言一样的数学表达能力的语言，并且让计算机能够理解，那么验证不过是CPU多转几圈的功夫。Lean就是这样一门语言。

当然读者不关心数学也没事，Lean这样的语言，或者说定理证明器，还能用于软件验证。当然读者不关心软件验证也没事，Lean本身当作玩具也挺好玩。当然读者如果觉得自己不可能对任何编程相关的东西有兴趣，那么就当听了个段子或者科普吧，我们好聚好散。

Lean的入门教程很多，虽然基本都是英文的，但圈子依然很小。我个人以为，目前Lean的教程目的都在于把读者快速培养成能够参与定理证明的科研民工，能够为数学形式化的发展添砖加瓦，属于是姜太公钓鱼了，没接触过这一领域的人根本不会因为读完教程而产生任何兴趣。Python这样的语言为什么火？因为除了程序员之外的大多数人都能用Python辅助完成一些工作，比如办公自动化，比如抢票，比如为了毕业收集、处理数据。

Lean当然没这么方便，本文将其纯粹当作玩具来介绍。如果看完本文觉得这玩具没意思，那情有可原，毕竟它不是真的玩具。如果觉得有意思，那欢迎成为Leaner。

后续会有一些代码，在本地安装Lean会劝退许多初学者，可以在官方社区提供的 https://live.lean-lang.org/ 平台玩耍。

## Basic Game

我们来用Lean写一个卡牌游戏。不知道读者是否玩过游戏王、炉石传说等卡牌，没玩过也没事，下面我们简单介绍一下我们将要构建的游戏规则。

我们的卡牌分为怪兽牌和魔法牌，怪兽牌有攻击力和防御力两个属性，魔法牌则是可以提高或降低怪兽牌的属性，当怪兽主动攻击时，如果被攻击怪兽的防御力低于攻击怪兽的攻击力，那么本次攻击成功，攻击者获胜，反之防御者获胜。

目前的规则很简单，我们后续再进行扩展。让我们来想想如何构建这么一款简单的卡牌游戏。如果读者有面向对象编程的经验，可能第一反应是定义怪兽牌和魔法牌各自对应的类，然后定义攻击等方法。这一思路没错，哪怕是没有编程经验也可以想到，首先我们需要定义出我们所描述的对象。在Lean中，我们可以这么写：

```lean
class MonsterCard where
  Atk : Nat
  Def : Nat
```

这段代码定义了 `MonsterCard` 这么一个类型，这一类型的对象或者说实例都有着 `Atk` 与 `Def` 两个属性，表示攻击力与防御力，而这两个属性的类型是 `Nat`，也就是自然数。类型在Lean中是核心概念，所有东西都有类型，类型之间也有着复杂的计算法则，但本文不形式化地严格讨论这些东西。

魔法牌呢？我们做一个简单的抽象，提高或降低怪兽牌的属性，实际上是把一张怪兽牌映射到一张改变了属性后的怪兽牌，魔法牌就是函数，接受怪兽牌返回怪兽牌的函数。*换句话说，函数就是魔法。*

```lean
class MagicCard where
  magic : MonsterCard -> MonsterCard
```

我们依然定义 `MagicCard` 这么一个类型，这个类型的对象的属性不像怪兽牌是两个自然数，而是一个名为 `magic` 的函数，这一函数的类型是 `MonsterCard -> MonsterCard`，也就是把怪兽牌打到怪兽牌的函数。

现在我们可以处理怪兽牌之间攻击的逻辑了，不过在此之前，我们先来定义“输赢”。

```lean
inductive Result
| WIN : Result
| LOSE : Result
derving Repr
```

可以看到我们这里使用了与先前的 `class` 不同的关键词 `inductive`，语法也有所不同。实际上，`Result` 后面还是可以加个 `where` 的，表达的意思一样，但既然可以不加，那为了简洁还是不加吧。最后一行的 `deriving Repr` 是为了让Lean自动生成能够在信息窗口中展示该类型对象的代码，这里可以忽略。

这个类型有点类似于枚举类型，也就是说这个类型的对象就 `WIN` 和 `LOSE` 两个，使用时要写 `Result.WIN` 或者 `Result.LOSE`。事实上 `inductive` 可以定义更复杂的类型，它可以做到递归的定义，不过本文不提。

接下来我们就可以定义攻击了：

```lean
def MonsterCard.attack (m₁ m₂ : MonsterCard) :=
  if m₁.Atk < m₂.Def then Result.LOSE else Result.WIN
```

我们的攻击没有平局，只有同归于尽，而既然攻击者的目的就是消灭防御者，那么同归于尽算攻击者胜利，很合理。

之所以在 `attack` 之前加上 `MonsterCard` 前缀，是因为Lean提供了一种方便的机制，如此定义之后，我们应用这一函数就不用写 `attack m₁ m₂` 了，只需要 `m₁.attack m₂` 即可。

现在，我们来实际定义两个怪兽牌对象：

```lean
def Godzilla : MonsterCard := ⟨1000, 3000⟩
def Ghidorah : MonsterCard := ⟨2000, 1000⟩
```

尖角符号是Lean提供的一项便利，需要用 `\<` 和 `\>` 输入，它可以把我们输入的数值自动填入到先前定义怪兽牌时确定的 `Atk` 与 `Def`。这两句的意思是，`Godzilla` 是一张攻击力为1000，防御力为3000的怪兽牌，而 `Ghidorah` 是一张攻击力为2000，防御力为1000的怪兽牌。

那么当 `Godzilla` 攻击 `Ghidorah` 时，根据我们刚刚定义的攻击函数，结果当是同归于尽，`Godzilla` 胜出，反过来，当 `Ghidorah` 攻击 `Godzilla` 时，由于攻击力小于对方的防御力，依然是 `Godzilla` 胜出。

这一胜负结果可以被Lean直接计算：

```lean
#eval Godzilla.attack Ghidorah -- Result.WIN
#eval Ghidorah.attack Godzilla -- Result.LOSE
```

`--` 后的属于注释，对程序没有影响。

现在我们已经看到Lean作为普通编程语言的一面了，这些逻辑拿任何编程语言写都差不多，但Lean还有着定理证明器的一面。可以尝试如下语句：

```lean
#eval Godzilla.attack Ghidorah = Result.WIN
```

在Python等语言中，我们会直接计算得到true，但在Lean中，它返回了一串报错，说这个等式并不是 `Decidable` 的。也就是说，对于尚未确定是否可决定的计算，Lean拒绝执行，而其他所有编程语言都有类似的限制。当你想要让程序计算判断两个对象是否相等，程序首先得知道如何去判断，以及这一判断算法会不会终止，会不会导致程序崩溃。

而事实上，如果把数学定理的证明看作这样的算法，其他编程语言都无能为力，因为他们无法计算非决定性的算法。但Lean可以，你可以去证明这一结果。

```lean
example : Godzilla.attack Ghidorah = Result.WIN := rfl
```

`rfl` 就是这一命题的证明，因为实际上只需把 `attack` 按定义展开即可，`rfl` 能够做到这一点。

当然，目前来看这种能力属实鸡肋，毕竟我们可以直接通过eval得到计算结果。那如果是以下的命题呢？

```lean
example : ∃ m : MagicCard, (m.magic Ghidorah).attack Godzilla = Result.WIN := sorry
```

我们想要证明存在一种魔法牌，作用于 `Ghidorah` 后能够使它成功攻击打败 `Godzilla`，这种命题就完全超出传统编程语言能够计算的范畴了，但使用Lean，你依然可以去尝试证明它。除了存在性命题，任意性的命题也可以。

我们先来定义一张魔法牌吧，用于强化攻击力。

```lean
def enhanceAtk : MagicCard := ⟨fun m => ⟨m.Atk + 1000, m.Def⟩⟩
```

我们依然使用了尖角符号，如果不用的话，以下写法也可以：

```lean
def enhanceAtk' : MagicCard := {
  magic := fun m => ⟨m.Atk + 1000, m.Def⟩
}
```

这一魔法牌把接收的怪兽牌攻击力加1000后返回，显然这样足够 `Ghidorah` 攻击 `Godzilla` 后同归于尽了，我们也能够证明刚刚提出的存在性命题了。

```lean
example : ∃ m : MagicCard, (m.magic Ghidorah).attack Godzilla = Result.WIN := by
  use enhanceAtk
  rfl
```

这里 `by` 的作用是进入到所谓的tactic mode，键入 `by` 后，正常来说Lean的信息窗口中能够展示出你当前证明的状态，包括你要证明的命题与可用的条件，当然我们这里不存在额外的条件，所以窗口中应当只有你要证明的命题。

输入 `by` 后，每一行开头的关键词即为tactic，比如这里的 `use` 和 `rfl`，这些tactic能够改写信息窗口中的证明状态，直到完成证明，显示 `No goals`。

我们还能再看一个带条件的例子，这里就不需要定义具体的魔法牌了，直接把刚刚我们证明的命题作为条件写入。

```lean
example (h : ∃ m : MagicCard, (m.magic Ghidorah).attack Godzilla = Result.WIN) :
  ¬(∀ m : MagicCard, (m.magic Ghidorah).attack Godzilla = Result.LOSE) := by
    push_neg
    rcases h with ⟨m, hm⟩
    use m
    rw [hm]
    simp
```

这里我们说，假设存在一张魔法牌能够使 `Ghidorah` 被施展魔法后攻击 `Godzilla` 能够胜利，那么“任意魔法牌都无法使 `Ghidorah` 被施展魔法后战胜 `Godzilla`”这一命题是错的。

这里我们使用了 `push_neg`、`rcases`、`rw`、`simp`等新的tactic，它们的具体作用我这里就不具体解释了，读者可以通过自行观察信息窗口变化来理解。

## Expansion

说实话估计没人觉得我们刚刚设计的游戏好玩，毕竟规则太简单了，下面我们对游戏进行一些扩展。

我们定义一类魔法怪兽牌，这些怪兽在攻击时能够对自己施展强化魔法，并且对敌人施展削弱魔法。魔法怪兽相互攻击时，攻击者首先对自己施展强化魔法，然后被敌方施展削弱魔法，敌方同样如此。

我们在怪兽牌的基础上扩展出魔法怪兽牌：

```lean
class MagicMonsterCard extends MonsterCard where
  self_magic : MonsterCard -> MonsterCard
  oppo_magic : MonsterCard -> MonsterCard
```

了解面向对象编程的读者可能会立刻想到继承，事实上也的确如此，魔法怪兽牌继承了怪兽牌攻击力与防御力的属性，同时增加了两个魔法属性。事实上，Lean还悄悄做了一些工作，你可以用 `toMonsterCard` 把一张魔法怪兽牌转换成怪兽牌，保持攻击力与防御力不变。这在定义魔法怪兽牌之间的攻击时非常有用。

```lean
def MagicMonsterCard.attack (m₁ m₂ : MagicMonsterCard) :=
  if (m₂.oppo_magic (m₁.self_magic m₁.toMonsterCard)).Atk < (m₁.oppo_magic (m₂.self_magic m₂.toMonsterCard)).Def
  then Result.LOSE
  else Result.WIN
```

注意先前提到的，**Lean中一切皆有类型**，所以既然我们定义 `self_magic` 与 `oppo_magic` 的类型为 `MonsterCard -> MonsterCard`，我们就不能把 `m₁ m₂` 直接给它们，因为 `m₁ m₂` 的类型为 `MagicMonsterCard`。这与Python等语言中的“鸭子类型”设计十分不同。

但无论如何，我们现在已经定义完魔法怪兽牌和他们的攻击了，理论上我们的工作只剩下设计大量的魔法怪兽牌，再做个好看的游戏界面，就能直接上线了（雾）。当然我们可以添加更多的机制，比如加两个玩家，给玩家固定血量，在回合制中攻击者的怪兽攻击力与被攻击怪兽防御力之差直接扣玩家血量之类的，但这都是小问题。

真正的问题是，我们的设计真的没问题吗？

来看下面这个命题：

```lean
example : ∃ m : MagicMonsterCard, ∀ n : MagicMonsterCard, m.attack n = Result.WIN := sorry
```

它的意思是，理论上存在一张魔法怪兽牌，它攻击任何魔法怪兽牌都能获胜，包括它自己。这个命题很好证，我们能非常简单地构造出这种怪兽牌：

```lean
example : ∃ m : MagicMonsterCard, ∀ n : MagicMonsterCard, m.attack n = Result.WIN := by
  let superMonster : MagicMonsterCard := {
    Atk := 1
    Def := 1
    self_magic := fun m => ⟨10000, 10000⟩
    oppo_magic := fun m => ⟨0, 0⟩
  }
  use superMonster
  dsimp [superMonster, MagicMonsterCard.attack]
  exact fun n => rfl
```

这种怪兽牌把对面的攻击力和防御力都变成0，由于在设计中作用于对方的削弱魔法是后生效的，因此对面不管怎么增强自己也没用，最后全部归零。另外我们设计同归于尽为胜利，因此两个0对碰也依然是攻击者胜利。

这一命题显然说明，我们的游戏机制存在巨大漏洞，魔法怪兽牌的魔法太随便了。另外，思考为什么把对面归零之后我们就能始终胜利，因为我们定义怪兽牌的攻击力和防御力都是自然数，没有比0更小的自然数了。但如果某个魔法怪兽的削弱魔法是让对面的属性减1000，而对面的属性甚至连1000也没有，那不直接成负的了？但属性又必须是自然数，这时候程序就崩溃了。

我们当然不能指望设计师不设计这样的卡牌，直接把这样的行为扼杀在摇篮才是最安全的做法，详情请见炉石传说的狂野模式。既然都用Lean了，当然要追求安全可信，我们为魔法怪兽牌添加一个命题属性：

```lean
class MagicMonsterCard extends MonsterCard where
  self_magic : MonsterCard -> MonsterCard
  oppo_magic : MonsterCard -> MonsterCard
  constraint : ∀ m : MonsterCard, (oppo_magic m).Def > 0
```

从此以后，设计师设计任何一张魔法怪兽牌，都必须提供证明，要求削弱魔法生效后对面的防御力必须大于0，否则不能构成合法的魔法怪兽牌。

这当然是一个很拙劣的补丁，但我们本来也不是想要真的设计一个可以上线卖钱的游戏。以上均是为了展示Lean在各个层面对命题对象的操作，我们既可以使用Lean去证明一个命题，也可以把命题作为函数的参数或者证明的条件，甚至可以定义含命题约束的数据结构。

注意到我们在上面的补丁里 `constraint` 的类型是后面的命题，而我们构造魔法怪兽牌时，为这一属性提供的是它的证明。从中可见Lean中的类型层级，具体命题也是类型，而它的对象则是它的证明。

更多有关Lean中类型系统的东西本文不会涉及，毕竟，这只是一篇非形式化的入门指南。

## Lean More

> 本节标题没打错。

如果上述卡牌游戏的例子对你来说还算有趣，那么Lean可能是适合你的一款玩具，你可以尝试去正式地学习如何使用Lean。下面是一些推荐材料：

- Mathematics in Lean：从具体实践角度介绍Lean的经典材料，适合想要立刻上手的有数学背景的读者。
- Functional Programming in Lean：介绍作为函数式编程语言的Lean，更适合计算机背景的读者。
- Metaprogramming in Lean 4：Lean的元编程，介绍怎么写tactic，建议进阶读者阅读。

以上材料都是开源的，但可能需要一点魔法才能看。

Lean社区还有一个游戏网站 https://adam.math.hhu.de/ ，里面是人为设计的习题，附带指导，新人可以尝试从此开始。