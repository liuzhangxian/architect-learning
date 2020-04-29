#### 2020年4月29日 早
纤云弄巧，飞星传恨，银汉迢迢暗渡。金风玉露一相逢，便胜却人间无数。

柔情似水，佳期如梦，忍顾鹊桥归路！两情若是久长时，又岂在朝朝暮暮。

#### 规则引擎构成
drools规则引擎由三部分组成:

1. Working Memory(工作内存)：存放插入的fact数据对象，fact对象是应用和规则引擎进行数据交互的桥梁。
2. Rule Base（规则库）：定义的所有规则，都会被加载到这里。
3. Inference Engine（推理引擎），又包括：

    Pattern Matcher（匹配器）：fact和规则做匹配，将匹配成功的规则放到Agenda中。

    Agenda（议程）：存放匹配到的被激活的规则。

    Execution Engine（执行引擎）：执行Agenda中的规则。

#### KIE（Knowledge Is Everything）
KIE是JBoss一系列项目的总称，包括OptaPlanner，Drools，UberFire，JBPM。

#### drools基本语法
1. update（$student)/retract($student)/insert($student) 更新、删除、插入工作内存对象，并触发规则重新匹配。

#### 规则属性
用在rule和when之间，可定义多个属性

|属性名|说明|
|:-----:|:-----:|
|salience| 指定规则执行优先级，取值Integer，数值越大优先级越高（默认为0，从上先下执行）|
|dialect|指定规则使用的语言类型，取值为"java"或"mvel"|
|enabled|指定规则是否启用，true/false|
|date-effective|指定生效起始时间, date-effective "2020-04-05 10:00:00"，System.setProperty("drools.dateformat","yyyy-MM-dd HH:mm:ss")|
|date-expires|指定失效时间,即生效终止时间|
|activation-group|激活分组，取值String，具有相同分组名称的规则只能有一个规则触发，取值String|
|agenda-group|议程分组，取值String，只有获取焦点的组中的规则才有可能触发，session.getAgenda().gentAgendaGroup("议程分组名").setFocus()|
|timer|定时器，session开启的时间内，指定规则触发时间，timer (cron:0/2 * * * * ?)没2秒触发一次|
|auto-focus|如果没有指定焦点，自动获取焦点，取值true/false,一般结合agenda-group一起使用|
|no-loop|防止死循环，取值true/false|

#### 高级语法
1. global，可以在规则文件中定义全局变量，程序给规则设置全局变量，Integer（规则内拷贝，自改其他规则不受影响），list（规则内引用，修改其他规则受影响），Java service对象，规则中可对全局变量调用方法、修改。调动方法返回值可以怎样处理?
2. query，可以在规则文件中定义查询，查询工作内存中符合条件的fact对象。
3. function，可以在规则文件中定义方法，供规则调用。
4. exist, 2个符合条件对象，规则触发一次，不用exist则会触发2次.
5. extends，规则继承，父规则和子规则条件都符合，则子规则才触发，同时子规则也触发。
6. drools.halt(),终止所有规则执行，类似于break。


