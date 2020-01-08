---
title: Hexo UML
date: 2020-01-06 16:29:56
categories:
- tools
tags:
- tools
typora-root-url: Hexo-UML
typora-copy-images-to: Hexo-UML
---

# PlantUML 简介

[PlantUML](http://plantuml.com/)是一个画图脚本语言，官方介绍如下：

> Generate UML diagram from textual description



用它可以快速地画出：

1. [顺序图](http://plantuml.com/sequence-diagram)
2. [用例图](http://plantuml.com/use-case-diagram)
3. [类图](http://plantuml.com/class-diagram)
4. [活动图](http://plantuml.com/activity-diagram-beta)
5. [组件图](http://plantuml.com/component-diagram)
6. [状态图](http://plantuml.com/state-diagram)
7. [对象图](http://plantuml.com/object-diagram)
8. [部署图](http://plantuml.com/deployment-diagram)
9. [时序图](http://plantuml.com/timing-diagram)

对于工程师们来说，用代码的方式来画图，简直是为其量身定做的。PlantUML语法也非常简单，参见[PlantUML Language Reference Guide](http://plantuml.com/PlantUML_Language_Reference_Guide.pdf)，它支持很多[工具](http://plantuml.com/running.html)，可以生成PNG、SVG、LaTeX和二进制图片。



# 插件安装

```shell
npm install hexo-tag-plantuml --save
```



# 语法

## 顺序图

### 声明参与者

-  关键字 `participant` 用于改变参与者的先后顺序。 

  你也可以使用其它关键字来声明参与者： 

  - `actor`
  - `boundary`
  - `control`
  - `entity`
  - `database`

-  关键字 `as` 用于重命名参与者 

-  您可以使用关键字 `order`自定义顺序来打印参与者。 

- 如果需要使用非字母符号，则需要使用引号

- 你可以使用[RGB](https://plantuml.com/zh/color)值或者颜色名修改 actor 或参与者的背景颜色。
-  `-->` 绘制一个虚线箭头 `->`绘制实线箭头

```markdown
{% plantuml %}
actor Foo1					#red
boundary Foo2		 #green
control Foo3 			 #blue
entity Foo4 
database Foo5 
collections Foo6 
Foo1 -> Foo2 : To boundary 
Foo1 -> Foo3 : To control 
Foo1 -> Foo4 : To entity 
Foo1 -> Foo5 : To database 
Foo1 -> Foo6 : To collections
{% endplantuml %}
```

{% plantuml %}
actor Foo1					#red
boundary Foo2		 #green
control Foo3 			 #blue
entity Foo4 
database Foo5 
collections Foo6 
Foo1 -> Foo2 : To boundary 
Foo1 -> Foo3 : To control 
Foo1 -> Foo4 : To entity 
Foo1 -> Foo5 : To database 
Foo1 -> Foo6 : To collections
{% endplantuml %}

### 修改箭头样式

 修改箭头样式的方式有以下几种: 

- 表示一条丢失的消息：末尾加 `x`
- 让箭头只有上半部分或者下半部分：将`<`和`>`替换成`\`或者 `/`
- 细箭头：将箭头标记写两次 (如 `>>` 或 `//`) 
- 虚线箭头：用 `--` 替代 `-`
- 箭头末尾加圈：`->o`
- 双向箭头：`<->`

```markdown
{% plantuml %}
Bob ->x Alice
Bob -> Alice
Bob ->> Alice
Bob -\ Alice
Bob \\- Alice
Bob //-- Alice

Bob ->o Alice
Bob o\\-- Alice

Bob <-> Alice
Bob <->o Alice
{% endplantuml %}
```

{% plantuml %}
Bob ->x Alice
Bob -> Alice
Bob ->> Alice
Bob -\ Alice
Bob \\- Alice
Bob //-- Alice

Bob ->o Alice
Bob o\\-- Alice

Bob <-> Alice
Bob <->o Alice
{% endplantuml %}

### 修改箭头颜色

```markdown
{% plantuml %}
Bob -[#red]> Alice : hello
Alice -[#0000FF]->Bob : ok
{% endplantuml %}
```

{% plantuml %}
Bob -[#red]> Alice : hello
Alice -[#0000FF]->Bob : ok
{% endplantuml %}

### 对消息序列编号

-  关键字 `autonumber` 用于自动对消息编号。 

-  语句 `autonumber *start*` 用于指定编号的初始值，而 `autonumber *start**increment*` 可以同时指定编号的初始值和每次增加的值。 

```markdown
{% plantuml %}
autonumber
Bob -> Alice : Authentication Request
Bob <- Alice : Authentication Response

autonumber 15
Bob -> Alice : Another authentication Request
Bob <- Alice : Another authentication Response

autonumber 40 10
Bob -> Alice : Yet another authentication Request
Bob <- Alice : Yet another authentication Response
{% endplantuml %}
```

{% plantuml %}
autonumber
Bob -> Alice : Authentication Request
Bob <- Alice : Authentication Response

autonumber 15
Bob -> Alice : Another authentication Request
Bob <- Alice : Another authentication Response

autonumber 40 10
Bob -> Alice : Yet another authentication Request
Bob <- Alice : Yet another authentication Response
{% endplantuml %}

### 分割示意图

 关键字 `newpage` 用于把一张图分割成多张。 

 在 `newpage` 之后添加文字，作为新的示意图的标题。 

 这样就能很方便地在 *Word* 中将长图分几页打印。 

```markdown
{% plantuml %}
Alice -> Bob : message 1
Alice -> Bob : message 2

newpage

Alice -> Bob : message 3
Alice -> Bob : message 4

newpage A title for the\nlast page

Alice -> Bob : message 5
Alice -> Bob : message 6
{% endplantuml %}
```

{% plantuml %}
Alice -> Bob : message 1
Alice -> Bob : message 2

newpage

Alice -> Bob : message 3
Alice -> Bob : message 4

newpage A title for the\nlast page

Alice -> Bob : message 5
Alice -> Bob : message 6
{% endplantuml %}

### 组合消息

 我们可以通过以下关键词将组合消息： 

- `alt/else`
- `opt`
- `loop`
- `par`
- `break`
- `critical`
- `group`, 后面紧跟着消息内容

 可以在标头(header)添加需要显示的文字(`group`除外)。 

 关键词 `end` 用来结束分组。 

 注意，分组可以嵌套使用。

```markdown
{% plantuml %}
alt successful case
	Bob -> Alice: Authentication Accepted
else some kind of failure
	Bob -> Alice: Authentication Failure
	group My own label
		Alice -> Log : Log attack start
	    loop 1000 times
	        Alice -> Bob: DNS Attack
	    end
		Alice -> Log : Log attack end
	end	
else Another type of failure
   Bob -> Alice: Please repeat
end
{% endplantuml %}
```

 {% plantuml %}
alt successful case
	Bob -> Alice: Authentication Accepted
else some kind of failure
	Bob -> Alice: Authentication Failure
	group My own label
		Alice -> Log : Log attack start
	    loop 1000 times
	        Alice -> Bob: DNS Attack
	    end
		Alice -> Log : Log attack end
	end	
else Another type of failure
   Bob -> Alice: Please repeat
end
{% endplantuml %}

### 添加注释

- 给消息添加注释

   我们可以通过在消息后面添加 `note left` 或者 `note right` 关键词来给消息添加注释。 

   你也可以通过使用 `end note` 来添加多行注释。 

  ```markdown
  {% plantuml %}
  Alice->Bob : hello
  note left: this is a first note
  
  Bob->Alice : ok
  note right: this is another note
  
  Bob->Bob : I am thinking
  note left
  	a note
  	can also be defined
  	on several lines
  end note
  {% endplantuml %}
  ```

  {% plantuml %}
  Alice->Bob : hello
  note left: this is a first note

  Bob->Alice : ok
  note right: this is another note

  Bob->Bob : I am thinking
  note left
  	a note
  	can also be defined
  	on several lines
  end note
  {% endplantuml %}

- 其他注释

   可以使用`note left of`，`note right of`或`note over`在节点(participant)的相对位置放置注释。 

   还可以通过修改[背景色](https://plantuml.com/zh/color)来高亮显示注释。 

   以及使用关键字`end note`来添加多行注释。 

  ```markdown
  {% plantuml %}
  participant Alice
  participant Bob
  note left of Alice #aqua
  	This is displayed 
  	left of Alice. 
  end note
   
  note right of Alice: This is displayed right of Alice.
  
  note over Alice: This is displayed over Alice.
  
  note over Alice, Bob #FFAAAA: This is displayed\n over Bob and Alice.
  
  note over Bob, Alice
  	This is yet another
  	example of
  	a long note.
  end note
  {% endplantuml %}
  ```

  {% plantuml %}
  participant Alice
  participant Bob
  note left of Alice #aqua
  	This is displayed 
  	left of Alice. 
  end note

  note right of Alice: This is displayed right of Alice.

  note over Alice: This is displayed over Alice.

  note over Alice, Bob #FFAAAA: This is displayed\n over Bob and Alice.

  note over Bob, Alice
  	This is yet another
  	example of
  	a long note.
  end note
  {% endplantuml %}

- 改变备注框的形状

   你可以使用 `hnote` 和 `rnote` 这两个关键字来修改备注框的形状。 

  ```markdown
  {% plantuml %}
  caller -> server : conReq
  hnote over caller : idle
  caller <- server : conConf
  rnote over server
  	"r" as rectangle
  	"h" as hexagon
  endrnote
  {% endplantuml %}
  ```

  {% plantuml %}
  caller -> server : conReq
  hnote over caller : idle
  caller <- server : conConf
  rnote over server
  	"r" as rectangle
  	"h" as hexagon
  endrnote
  {% endplantuml %}

### 分隔符

 你可以通过使用 `==` 关键词来将你的图表分割多个步骤。 

```markdown
{% plantuml %}
== Initialization ==

Alice -> Bob: Authentication Request
Bob --> Alice: Authentication Response

== Repetition ==

Alice -> Bob: Another authentication Request
Alice <-- Bob: another authentication Response
{% endplantuml %}
```

{% plantuml %}
== Initialization ==

Alice -> Bob: Authentication Request
Bob --> Alice: Authentication Response

== Repetition ==

Alice -> Bob: Another authentication Request
Alice <-- Bob: another authentication Response
{% endplantuml %}

### 引用

 你可以在图中通过使用`ref over`关键词来实现引用 

```markdown
{% plantuml %}
participant Alice
actor Bob

ref over Alice, Bob : init

Alice -> Bob : hello

ref over Bob
  This can be on
  several lines
end ref
{% endplantuml %}
```

{% plantuml %}
participant Alice
actor Bob

ref over Alice, Bob : init

Alice -> Bob : hello

ref over Bob
  This can be on
  several lines
end ref
{% endplantuml %}

### 延迟

 你可以使用`...`来表示延迟，并且还可以给延迟添加注释。 

```markdown
{% plantuml %}
Alice -> Bob: Authentication Request
...
Bob --> Alice: Authentication Response
...5 minutes latter...
Bob --> Alice: Bye !
{% endplantuml %}
```

{% plantuml %}
Alice -> Bob: Authentication Request
...
Bob --> Alice: Authentication Response
...5 minutes latter...
Bob --> Alice: Bye !
{% endplantuml %}

### 增加空间

 你可以使用`|||`来增加空间。 

 还可以使用数字指定增加的像素的数量。 

```markdown
{% plantuml %}
Alice -> Bob: message 1
Bob --> Alice: ok
|||
Alice -> Bob: message 2
Bob --> Alice: ok
||45||
Alice -> Bob: message 3
Bob --> Alice: ok
{% endplantuml %}
```

{% plantuml %}
Alice -> Bob: message 1
Bob --> Alice: ok
|||
Alice -> Bob: message 2
Bob --> Alice: ok
||45||
Alice -> Bob: message 3
Bob --> Alice: ok
{% endplantuml %}

### 生命线的激活与撤销

 关键字`activate`和`deactivate`用来表示参与者的生命活动。 

 一旦参与者被激活，它的生命线就会显示出来。 

`activate`和`deactivate`适用于以上情形。 

`destroy`表示一个参与者的生命线的终结。 

 还可以使用嵌套的生命线，并且运行给生命线添加颜色。 

```markdown
{% plantuml %}
participant User

User -> A: DoWork
activate A #FFBBBB

A -> A: Internal call
activate A #DarkSalmon

A -> B: << createRequest >>
activate B

B --> A: RequestCreated
deactivate B
deactivate A
A -> User: Done
deactivate A
{% endplantuml %}
```

{% plantuml %}
participant User

User -> A: DoWork
activate A #FFBBBB

A -> A: Internal call
activate A #DarkSalmon

A -> B: << createRequest >>
activate B

B --> A: RequestCreated
deactivate B
deactivate A
A -> User: Done
deactivate A
{% endplantuml %}

### 创建参与者

 你可以把关键字`create`放在第一次接收到消息之前，以强调本次消息实际上是在*创建*新的对象。 

```markdown
{% plantuml %}
Bob -> Alice : hello

create Other
Alice -> Other : new

create control String
Alice -> String
note right : You can also put notes!

Alice --> Bob : ok
{% endplantuml %}
```

{% plantuml %}
Bob -> Alice : hello

create Other
Alice -> Other : new

create control String
Alice -> String
note right : You can also put notes!

Alice --> Bob : ok
{% endplantuml %}

### 激活，停用，创建的快捷语法

在指定目标参与者之后，可以立即使用以下语法：

- `++`  Activate the target (optionally a #color may follow this)
- `--` Deactivate the source
- `**` Create an instance of the target
- `!!` Destroy an instance of the target

```markdown
{% plantuml %}
alice -> bob ++ : hello
bob -> bob ++ : self call
bob -> bib ++  #005500 : hello
bob -> george ** : create
return done
return rc
bob -> george !! : delete
return success
{% endplantuml %}
```

{% plantuml %}
alice -> bob ++ : hello
bob -> bob ++ : self call
bob -> bib ++  #005500 : hello
bob -> george ** : create
return done
return rc
bob -> george !! : delete
return success
{% endplantuml %}

进入和发出消息

 如果只想关注部分图示，你可以使用进入和发出箭头。 

 使用方括号`[`和`]`表示图示的左、右两侧。 

```markdown
{% plantuml %}
[-> A: DoWork

activate A

A -> A: Internal call
activate A

A ->] : << createRequest >>

A<--] : RequestCreated
deactivate A
[<- A: Done
deactivate A
{% endplantuml %}
```

{% plantuml %}
[-> A: DoWork

activate A

A -> A: Internal call
activate A

A ->] : << createRequest >>

A<--] : RequestCreated
deactivate A
[<- A: Done
deactivate A
{% endplantuml %}

### 包裹参与者

 可以使用`box`和`end box`画一个盒子将参与者包裹起来。 

 还可以在`box`关键字之后添加标题或者背景颜色。 

```markdown
{% plantuml %}
box "Internal Service" #LightBlue
	participant Bob
	participant Alice
end box
participant Other

Bob -> Alice : hello
Alice -> Other : hello
{% endplantuml %}
```

{% plantuml %}
box "Internal Service" #LightBlue
	participant Bob
	participant Alice
end box
participant Other

Bob -> Alice : hello
Alice -> Other : hello
{% endplantuml %}

### 移除脚注

 使用`hide footbox`关键字移除脚注。 

```markdown
{% plantuml %}
hide footbox
title Footer removed

Alice -> Bob: Authentication Request
Bob --> Alice: Authentication Response
{% endplantuml %}
```

{% plantuml %}
hide footbox
title Footer removed

Alice -> Bob: Authentication Request
Bob --> Alice: Authentication Response
{% endplantuml %}

### 外观参数(skinparam)

 可以在如下场景中使用： 

-  在图示的定义中，
- [在引入的文件中](https://plantuml.com/zh/preprocessing)，
-  在[命令行](https://plantuml.com/zh/command-line)或者[ANT任务](https://plantuml.com/zh/ant-task)提供的配置文件中。

 你也可以修改其他渲染元素，如以下示例： 

```markdown
{% plantuml %}
skinparam sequenceArrowThickness 2
skinparam roundcorner 20
skinparam maxmessagesize 60
skinparam sequenceParticipant underline

actor User
participant "First Class" as A
participant "Second Class" as B
participant "Last Class" as C

User -> A: DoWork
activate A

A -> B: Create Request
activate B

B -> C: DoWork
activate C
C --> B: WorkDone
destroy C

B --> A: Request Created
deactivate B

A --> User: Done
deactivate A
{% endplantuml %}
```

{% plantuml %}
skinparam sequenceArrowThickness 2
skinparam roundcorner 20
skinparam maxmessagesize 60
skinparam sequenceParticipant underline

actor User
participant "First Class" as A
participant "Second Class" as B
participant "Last Class" as C

User -> A: DoWork
activate A

A -> B: Create Request
activate B

B -> C: DoWork
activate C
C --> B: WorkDone
destroy C

B --> A: Request Created
deactivate B

A --> User: Done
deactivate A
{% endplantuml %}

```markdown
{% plantuml %}
skinparam backgroundColor #EEEBDC
skinparam handwritten true

skinparam sequence {
	ArrowColor DeepSkyBlue
	ActorBorderColor DeepSkyBlue
	LifeLineBorderColor blue
	LifeLineBackgroundColor #A9DCDF
	
	ParticipantBorderColor DeepSkyBlue
	ParticipantBackgroundColor DodgerBlue
	ParticipantFontName Impact
	ParticipantFontSize 17
	ParticipantFontColor #A9DCDF
	
	ActorBackgroundColor aqua
	ActorFontColor DeepSkyBlue
	ActorFontSize 17
	ActorFontName Aapex
}

actor User
participant "First Class" as A
participant "Second Class" as B
participant "Last Class" as C

User -> A: DoWork
activate A

A -> B: Create Request
activate B

B -> C: DoWork
activate C
C --> B: WorkDone
destroy C

B --> A: Request Created
deactivate B

A --> User: Done
deactivate A
{% endplantuml %}
```

{% plantuml %}
skinparam backgroundColor #EEEBDC
skinparam handwritten true

skinparam sequence {
	ArrowColor DeepSkyBlue
	ActorBorderColor DeepSkyBlue
	LifeLineBorderColor blue
	LifeLineBackgroundColor #A9DCDF
	

	ParticipantBorderColor DeepSkyBlue
	ParticipantBackgroundColor DodgerBlue
	ParticipantFontName Impact
	ParticipantFontSize 17
	ParticipantFontColor #A9DCDF
	
	ActorBackgroundColor aqua
	ActorFontColor DeepSkyBlue
	ActorFontSize 17
	ActorFontName Aapex
}

actor User
participant "First Class" as A
participant "Second Class" as B
participant "Last Class" as C

User -> A: DoWork
activate A

A -> B: Create Request
activate B

B -> C: DoWork
activate C
C --> B: WorkDone
destroy C

B --> A: Request Created
deactivate B

A --> User: Done
deactivate A
{% endplantuml %}

## 用例图

### 用例

用例用圆括号括起来。 

也可以用关键字`usecase`来定义用例。 还可以用关键字`as`定义一个别名，这个别名可以在以后定义关系的时候使用。 

```markdown
{% plantuml %}
(First usecase)
(Another usecase) as (UC2)  
usecase UC3
usecase (Last\nusecase) as UC4
{% endplantuml %}
```

{% plantuml %}
(First usecase)
(Another usecase) as (UC2)  
usecase UC3
usecase (Last\nusecase) as UC4
{% endplantuml %}

### 角色

 角色用两个冒号包裹起来。 

 也可以用`actor`关键字来定义角色。 还可以用关键字`as`来定义一个别名，这个别名可以在以后定义关系的时候使用。 

 后面我们会看到角色的定义是可选的。 

```markdown
{% plantuml %}
:First Actor:
:Another\nactor: as Men2  
actor Men3
actor :Last actor: as Men4
{% endplantuml %}
```

{% plantuml %}
:First Actor:
:Another\nactor: as Men2  
actor Men3
actor :Last actor: as Men4
{% endplantuml %}

### 用例描述

 如果想定义跨越多行的用例描述，可以用双引号将其裹起来。 

 还可以使用这些分隔符：`--``..``==``__`。 并且还可以在分隔符中间放置标题。 

```markdown
{% plantuml %}
usecase UC1 as "You can use
several lines to define your usecase.
You can also use separators.
--
Several separators are possible.
==
And you can add titles:
..Conclusion..
This allows large description."
{% endplantuml %}
```

{% plantuml %}
usecase UC1 as "You can use
several lines to define your usecase.

You can also use separators.
--
Several separators are possible.
==
And you can add titles:
..Conclusion..
This allows large description."
{% endplantuml %}

### 基础示例

 用箭头`-->`连接角色和用例。 

 横杠`-`越多，箭头越长。 通过在箭头定义的后面加一个冒号及文字的方式来添加标签。 

 在这个例子中，*User*并没有定义，而是直接拿来当做一个角色使用。 

```markdown
{% plantuml %}
User -> (Start)
User --> (Use the application) : A small label

:Main Admin: ---> (Use the application) : This is\nyet another\nlabel
{% endplantuml %}
```

{% plantuml %}
User -> (Start)
User --> (Use the application) : A small label

:Main Admin: ---> (Use the application) : This is\nyet another\nlabel
{% endplantuml %}

### 继承

如果一个角色或者用例继承于另一个，那么可以用`<|--`符号表示。

```markdown
{% plantuml %}
:Main Admin: as Admin
(Use the application) as (Use)

User <|-- Admin
(Start) <|-- (Use)
{% endplantuml %}
```

{% plantuml %}
:Main Admin: as Admin
(Use the application) as (Use)

User <|-- Admin
(Start) <|-- (Use)
{% endplantuml %}

### 注释

 可以用`note left of` , `note right of` , `note top of` , `note bottom of`等关键字给一个对象添加注释。 

 注释还可以通过`note`关键字来定义，然后用`..`连接其他对象。 

```markdown
{% plantuml %}
:Main Admin: as Admin
(Use the application) as (Use)

User -> (Start)
User --> (Use)

Admin ---> (Use)

note right of Admin : This is an example.

note right of (Use)
  A note can also
  be on several lines
end note

note "This note is connected\nto several objects." as N2
(Start) .. N2
N2 .. (Use)
{% endplantuml %}
```

{% plantuml %}
:Main Admin: as Admin
(Use the application) as (Use)

User -> (Start)
User --> (Use)

Admin ---> (Use)

note right of Admin : This is an example.

note right of (Use)
  A note can also
  be on several lines
end note

note "This note is connected\nto several objects." as N2
(Start) .. N2
N2 .. (Use)
{% endplantuml %}

### 构造类型

 用 `<<` 和 `>>` 来定义角色或者用例的构造类型。 

```markdown
{% plantuml %}
User << Human >>
:Main Database: as MySql << Application >>
(Start) << One Shot >>
(Use the application) as (Use) << Main >>

User -> (Start)
User --> (Use)

MySql --> (Use)
{% endplantuml %}
```

{% plantuml %}
User << Human >>
:Main Database: as MySql << Application >>
(Start) << One Shot >>
(Use the application) as (Use) << Main >>

User -> (Start)
User --> (Use)

MySql --> (Use)
{% endplantuml %}

### 箭头方向

-  默认连接是竖直方向的，用`--`表示，可以用一个横杠或点来表示水平连接。 
-  也可以通过翻转箭头来改变方向。 
-  还可以通过给箭头添加`left`, `right`, `up`或`down`等关键字来改变方向。 

```markdown
{% plantuml %}
:user: --> (Use case 1)
:user: -> (Use case 2)
{% endplantuml %}

{% plantuml %}
(Use case 1) <.. :user:
(Use case 2) <- :user:
{% endplantuml %}

{% plantuml %}
:user: -left-> (dummyLeft) 
:user: -right-> (dummyRight) 
:user: -up-> (dummyUp)
:user: -down-> (dummyDown)
{% endplantuml %}
```

{% plantuml %}
:user: --> (Use case 1)
:user: -> (Use case 2)
{% endplantuml %}

{% plantuml %}
(Use case 1) <.. :user:
(Use case 2) <- :user:
{% endplantuml %}

{% plantuml %}
:user: -left-> (dummyLeft) 
:user: -right-> (dummyRight) 
:user: -up-> (dummyUp)
:user: -down-> (dummyDown)
{% endplantuml %}

### 分割图示

 用`newpage`关键字将图示分解为多个页面。 

```markdown
{% plantuml %}
:actor1: --> (Usecase1)
newpage
:actor2: --> (Usecase2)
{% endplantuml %}
```

{% plantuml %}
:actor1: --> (Usecase1)
newpage
:actor2: --> (Usecase2)
{% endplantuml %}

### 构图方向

-  默认从上往下构建图示。 
-  你可以用`left to right direction`命令改变图示方向。 

```markdown
{% plantuml %}
'default
top to bottom direction
user1 --> (Usecase 1)
user2 --> (Usecase 2)
{% endplantuml %}

{% plantuml %}
left to right direction
user1 --> (Usecase 1)
user2 --> (Usecase 2)
{% endplantuml %}
```

{% plantuml %}
'default top to bottom direction
user1 --> (Usecase 1)
user2 --> (Usecase 2)
{% endplantuml %}

{% plantuml %}
left to right direction
user1 --> (Usecase 1)
user2 --> (Usecase 2)
{% endplantuml %}

### 显示参数

 用`skinparam`改变字体和颜色。 

 可以在如下场景中使用： 

-  在图示的定义中，
- [在引入的文件中](https://plantuml.com/zh/preprocessing)，
-  在[命令行](https://plantuml.com/zh/command-line)或者[ANT任务](https://plantuml.com/zh/ant-task)提供的配置文件中。

 你也可以给构造的角色和用例指定特殊颜色和字体。 

```markdown
{% plantuml %}
skinparam handwritten true

skinparam usecase {
	BackgroundColor DarkSeaGreen
	BorderColor DarkSlateGray

	BackgroundColor<< Main >> YellowGreen
	BorderColor<< Main >> YellowGreen
	
	ArrowColor Olive
	ActorBorderColor black
	ActorFontName Courier

	ActorBackgroundColor<< Human >> Gold
}

User << Human >>
:Main Database: as MySql << Application >>
(Start) << One Shot >>
(Use the application) as (Use) << Main >>

User -> (Start)
User --> (Use)

MySql --> (Use)
{% endplantuml %}
```

{% plantuml %}
skinparam handwritten true

skinparam usecase {
	BackgroundColor DarkSeaGreen
	BorderColor DarkSlateGray

	BackgroundColor<< Main >> YellowGreen
	BorderColor<< Main >> YellowGreen
	
	ArrowColor Olive
	ActorBorderColor black
	ActorFontName Courier
	
	ActorBackgroundColor<< Human >> Gold
}

User << Human >>
:Main Database: as MySql << Application >>
(Start) << One Shot >>
(Use the application) as (Use) << Main >>

User -> (Start)
User --> (Use)

MySql --> (Use)
{% endplantuml %}



## 活动图

### 开始/结束

 你可以使用关键字`start`和`stop`表示图示的开始和结束。 

```markdown
{% plantuml %}
start
:Hello world;
:This is on defined on
several **lines**;
stop
{% endplantuml %}
```

{% plantuml %}
start
:Hello world;
:This is on defined on
several **lines**;
stop
{% endplantuml %}

### 条件语句

- 可以使用关键字`if`，`then`和`else`设置分支测试。标注文字则放在括号中。 
- 也可以使用关键字`elseif`设置多个分支测试

```markdown
{% plantuml %}
start
if (condition A) then (yes)
  :Text 1;
elseif (condition B) then (yes)
  :Text 2;
  stop
elseif (condition C) then (yes)
  :Text 3;
elseif (condition D) then (yes)
  :Text 4;
else (nothing)
  :Text else;
endif
stop
{% endplantuml %}
```

{% plantuml %}
start
if (condition A) then (yes)
  :Text 1;
elseif (condition B) then (yes)
  :Text 2;
  stop
elseif (condition C) then (yes)
  :Text 3;
elseif (condition D) then (yes)
  :Text 4;
else (nothing)
  :Text else;
endif
stop
{% endplantuml %}

### 循环

使用关键字`repeat`和`repeatwhile`进行重复循环。

```markdown
{% plantuml %}
start

repeat
  :read data;
  :generate diagrams;
repeat while (more data?)

stop
{% endplantuml %}
```

{% plantuml %}
start

repeat
  :read data;
  :generate diagrams;
repeat while (more data?)

stop
{% endplantuml %}

使用关键字`while`和`end while`进行while循环。

```markdown
{% plantuml %}
start

while (data available?)
  :read data;
  :generate diagrams;
endwhile

stop
{% endplantuml %}
```

{% plantuml %}
start

while (data available?)
  :read data;
  :generate diagrams;
endwhile

stop
{% endplantuml %}

### 并行处理

 你可以使用关键字`fork`，`fork again`和`end fork`表示并行处理。 

```markdown
{% plantuml %}
start

if (multiprocessor?) then (yes)
  fork
	:Treatment 1;
  fork again
	:Treatment 2;
  end fork
else (monoproc)
  :Treatment 1;
  :Treatment 2;
endif

stop
{% endplantuml %}
```

{% plantuml %}
start

if (multiprocessor?) then (yes)
  fork
	:Treatment 1;
  fork again
	:Treatment 2;
  end fork
else (monoproc)
  :Treatment 1;
  :Treatment 2;
endif

stop
{% endplantuml %}

### 注释

```markdown
{% plantuml %}
start
:foo1;
floating note left: This is a note
:foo2;
note right
  This note is on several
  //lines// and can
  contain <b>HTML</b>
  ====
  * Calling the method ""foo()"" is prohibited
end note
stop
{% endplantuml %}
```

{% plantuml %}
start
:foo1;
floating note left: This is a note
:foo2;
note right
  This note is on several
  //lines// and can
  contain <b>HTML</b>
  ====
  * Calling the method ""foo()"" is prohibited
end note
stop
{% endplantuml %}

### 颜色

你可以为活动(activity)指定一种[颜色](https://plantuml.com/zh/color)。 

```markdown
{% plantuml %}
start
:starting progress;
#HotPink:reading configuration files
These files should edited at this point!;
#AAAAAA:ending of the process;
stop
{% endplantuml %}
```

{% plantuml %}
start
:starting progress;
#HotPink:reading configuration files
These files should edited at this point!;
#AAAAAA:ending of the process;
stop
{% endplantuml %}

### 箭头

 使用`->`标记，你可以给箭头添加文字或者修改箭头[颜色](https://plantuml.com/zh/color)。 

 同时，你也可以选择点状 (dotted)，条状(dashed)，加粗或者是隐式箭头 

```markdown
{% plantuml %}
:foo1;
-> You can put text on arrows;
if (test) then
  -[#blue]->
  :foo2;
  -[#green,dashed]-> The text can
  also be on several lines
  and **very** long...;
  :foo3;
else
  -[#black,dotted]->
  :foo4;
endif
-[#gray,bold]->
:foo5;
{% endplantuml %}
```

{% plantuml %}
:foo1;
-> You can put text on arrows;
if (test) then
  -[#blue]->
  :foo2;
  -[#green,dashed]-> The text can
  also be on several lines
  and **very** long...;
  :foo3;
else
  -[#black,dotted]->
  :foo4;
endif
-[#gray,bold]->
:foo5;
{% endplantuml %}

### 连接器

你可以使用括号定义连接器

```markdown
{% plantuml %}
start
:Some activity;
(A)
detach
(A)
:Other activity;
stop
{% endplantuml %}
```

{% plantuml %}
start
:Some activity;
(A)
detach
(A)
:Other activity;
stop
{% endplantuml %}

### 组合

 通过定义分区(partition)，你可以把多个活动组合(group)在一起。 

```markdown
{% plantuml %}
start
partition Initialization {
	:read config file;
	:init internal variable;
}
partition Running {
	:wait for user interaction;
	:print information;
}

stop
{% endplantuml %}
```

{% plantuml %}
start
partition Initialization {
	:read config file;
	:init internal variable;
}
partition Running {
	:wait for user interaction;
	:print information;
}

stop
{% endplantuml %}

### 泳道

 你可以使用管道符`|`来定义泳道。 

 还可以改变泳道的[颜色](https://plantuml.com/zh/color)。 

```markdown
{% plantuml %}
|Swimlane1|
start
:foo1;
|#AntiqueWhite|Swimlane2|
:foo2;
:foo3;
|Swimlane1|
:foo4;
|Swimlane2|
:foo5;
stop
{% endplantuml %}
```

{% plantuml %}
|Swimlane1|
start
:foo1;
|#AntiqueWhite|Swimlane2|
:foo2;
:foo3;
|Swimlane1|
:foo4;
|Swimlane2|
:foo5;
stop
{% endplantuml %}

### 分离

 可以使用关键字`detach`移除箭头。 

```markdown
{% plantuml %}
 :start;
 fork
   :foo1;
   :foo2;
 fork again
   :foo3;
   detach
 endfork
 if (foo4) then
   :foo5;
   detach
 endif
 :foo6;
 detach
 :foo7;
 stop
{% endplantuml %}
```

{% plantuml %}
 :start;
 fork
   :foo1;
   :foo2;
 fork again
   :foo3;
   detach
 endfork
 if (foo4) then
   :foo5;
   detach
 endif
 :foo6;
 detach
 :foo7;
 stop
{% endplantuml %}

### 特殊领域语言(SDL)

 通过修改活动标签最后的分号分隔符(`;`)，可以为活动设置不同的形状。 

- `|`
- `<`
- `>`
- `/`
- `]`
- `}`

```markdown
{% plantuml %}
:Ready;
:next(o)|
:Receiving;
split
 :nak(i)<
 :ack(o)>
split again
 :ack(i)<
 :next(o)
 on several line|
 :i := i + 1]
 :ack(o)>
split again
 :err(i)<
 :nak(o)>
split again
 :foo/
split again
 :i > 5}
stop
end split
:finish;
{% endplantuml %}
```

{% plantuml %}
:Ready;
:next(o)|
:Receiving;
split
 :nak(i)<
 :ack(o)>
split again
 :ack(i)<
 :next(o)
 on several line|
 :i := i + 1]
 :ack(o)>
split again
 :err(i)<
 :nak(o)>
split again
 :foo/
split again
 :i > 5}
stop
end split
:finish;
{% endplantuml %}

### 完整实例

```markdown
{% plantuml %}
start
:ClickServlet.handleRequest();
:new page;
if (Page.onSecurityCheck) then (true)
  :Page.onInit();
  if (isForward?) then (no)
	:Process controls;
	if (continue processing?) then (no)
	  stop
	endif
	
	if (isPost?) then (yes)
	  :Page.onPost();
	else (no)
	  :Page.onGet();
	endif
	:Page.onRender();
  endif
else (false)
endif

if (do redirect?) then (yes)
  :redirect process;
else
  if (do forward?) then (yes)
	:Forward request;
  else (no)
	:Render page template;
  endif
endif

stop

{% endplantuml %}
```

{% plantuml %}
start
:ClickServlet.handleRequest();
:new page;
if (Page.onSecurityCheck) then (true)
  :Page.onInit();
  if (isForward?) then (no)
	:Process controls;
	if (continue processing?) then (no)
	  stop
	endif
	
	if (isPost?) then (yes)
	  :Page.onPost();
	else (no)
	  :Page.onGet();
	endif
	:Page.onRender();
  endif
else (false)
endif

if (do redirect?) then (yes)
  :redirect process;
else
  if (do forward?) then (yes)
	:Forward request;
  else (no)
	:Render page template;
  endif
endif

stop

{% endplantuml %}



## 类图



# 思维导图

## 安装插件

```shell
npm install hexo-simple-mindmap
```

## 使用

```markdown
{% pullquote mindmap mindmap-md %}
- [在 Hexo 中使用思维导图](https://hunterx.xyz/use-mindmap-in-hexo.html)
  - 前言
  - 操作指南
    - 准备需要的文件
    - 为主题添加 CSS/JS 文件
  - 使用方法
{% endpullquote %}
```

{% pullquote mindmap mindmap-md %}
- [在 Hexo 中使用思维导图](https://hunterx.xyz/use-mindmap-in-hexo.html)
  - 前言
  - 操作指南
    - 准备需要的文件
    - 为主题添加 CSS/JS 文件
  - 使用方法

{% endpullquote %}