# 自定义语言的实现——解释器模式（三）

## 解释器模式概述  

解释器模式是一种使用频率相对较低但学习难度较大的设计模式，它用于描述如何使用面向对象语言构成一个简单的语言解释器。在某些情况下，为了更好地描述某一些特定类型的问题，我们可以创建一种新的语言，这种语言拥有自己的表达式和结构，即文法规则，这些问题的实例将对应为该语言中的句子。此时，可以使用解释器模式来设计这种新的语言。对解释器模式的学习能够加深我们对面向对象思想的理解，并且掌握编程语言中文法规则的解释过程。  

解释器模式定义如下：  

解释器模式（Interpreter Pattern）：定义一个语言的文法，并且建立一个解释器来解释该语言中的句子，这里的“语言”是指使用规定格式和语法的代码。解释器模式是一种类行为型模式。  

由于表达式可分为终结符表达式和非终结符表达式，因此解释器模式的结构与组合模式的结构有些类似，但在解释器模式中包含更多的组成元素，它的结构如图所示：
       
![解释器模式结构图](images/1341331467_7271.jpg)  

在解释器模式结构图中包含如下几个角色：  

- AbstractExpression（抽象表达式）：在抽象表达式中声明了抽象的解释操作，它是所有终结符表达式和非终结符表达式的公共父类。  

- TerminalExpression（终结符表达式）：终结符表达式是抽象表达式的子类，它实现了与文法中的终结符相关联的解释操作，在句子中的每一个终结符都是该类的一个实例。通常在一个解释器模式中只有少数几个终结符表达式类，它们的实例可以通过非终结符表达式组成较为复杂的句子。  

- NonterminalExpression（非终结符表达式）：非终结符表达式也是抽象表达式的子类，它实现了文法中非终结符的解释操作，由于在非终结符表达式中可以包含终结符表达式，也可以继续包含非终结符表达式，因此其解释操作一般通过递归的方式来完成。  

- Context（环境类）：环境类又称为上下文类，它用于存储解释器之外的一些全局信息，通常它临时存储了需要解释的语句。  

在解释器模式中，每一种终结符和非终结符都有一个具体类与之对应，正因为使用类来表示每一条文法规则，所以系统将具有较好的灵活性和可扩展性。对于所有的终结符和非终结符，我们首先需要抽象出一个公共父类，即抽象表达式类，其典型代码如下所示：  
```
abstract class AbstractExpression {
       public  abstract void interpret(Context ctx);
}
```  
终结符表达式和非终结符表达式类都是抽象表达式类的子类，对于终结符表达式，其代码很简单，主要是对终结符元素的处理，其典型代码如下所示：  
```
class TerminalExpression extends  AbstractExpression {
       public  void interpret(Context ctx) {
              //终结符表达式的解释操作
       }
}
```
对于非终结符表达式，其代码相对比较复杂，因为可以通过非终结符将表达式组合成更加复杂的结构，对于包含两个操作元素的非终结符表达式类，其典型代码如下：  
```
class NonterminalExpression extends  AbstractExpression {
       private  AbstractExpression left;
       private  AbstractExpression right;
      
       public  NonterminalExpression(AbstractExpression left,AbstractExpression right) {
              this.left=left;
              this.right=right;
       }
      
       public void interpret(Context ctx) {
              //递归调用每一个组成部分的interpret()方法
              //在递归调用时指定组成部分的连接方式，即非终结符的功能
       }     
}
```

除了上述用于表示表达式的类以外，通常在解释器模式中还提供了一个环境类 Context，用于存储一些全局信息，通常在 Context 中包含了一个 HashMap 或 ArrayList 等类型的集合对象（也可以直接由 HashMap 等集合类充当环境类），存储一系列公共信息，如变量名与值的映射关系（key/value）等，用于在进行具体的解释操作时从中获取相关信息。其典型代码片段如下：  
```
class Context {
     private HashMap map = new HashMap();
     public void assign(String key, String value) {
         //往环境类中设值
     }
public String  lookup(String key) {
         //获取存储在环境类中的值
     }
}
```
当系统无须提供全局公共信息时可以省略环境类，可根据实际情况决定是否需要环境类。  

**思考**  
绘制加法/减法解释器的类图并编写核心实现代码。
