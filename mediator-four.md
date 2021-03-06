# 协调多个对象之间的交互——中介者模式（四）  

## 中介者与同事类的扩展  

Sunny 软件公司 CRM 系统的客户对“客户信息管理窗口”提出了一个修改意见：要求在窗口的下端能够及时显示当前系统中客户信息的总数。修改之后的界面如图所示：

![修改之后的“客户信息管理窗口”界面图](images/1357653249_4096.jpg)  

从图中我们不难发现，可以通过增加一个文本标签（Label）来显示客户信息总数，而且当用户点击“增加”按钮或者“删除”按钮时，将改变文本标签的内容。  

由于使用了中介者模式，在原有系统中增加新的组件（即新的同事类）将变得很容易，我们至少有如下两种解决方案：  

**【解决方案一】**增加一个界面组件类 Label，修改原有的具体中介者类 ConcreteMediator，增加一个对 Label 对象的引用，然后修改 componentChanged() 方法中其他相关组件对象的业务处理代码，原有组件类无须任何修改，客户端代码也需针对新增组件 Label 进行适当修改。  

**【解决方案二】**与方案一相同，首先增加一个 Label 类，但不修改原有具体中介者类 ConcreteMediator 的代码，而是增加一个 ConcreteMediator 的子类 SubConcreteMediator 来实现对 Label 对象的引用，然后在新增的中介者类 SubConcreteMediator 中通过覆盖 componentChanged() 方法来实现所有组件（包括新增 Label 组件）之间的交互，同样，原有组件类无须做任何修改，客户端代码需少许修改。  

引入 Label 之后“客户信息管理窗口”类结构示意图如图所示：  

![增加Label组件类后的“客户信息管理窗口”结构示意图](images/1357653254_7868.jpg) 

由于**【解决方案二】**无须修改 ConcreteMediator 类，更符合“开闭原则”，因此我们选择该解决方案来对新增 Label 类进行处理，对应的完整类图如图所示：  

![修改之后的“客户信息管理窗口”结构图](images/1357653264_4751.jpg)

在图中，新增了具体同事类 Label 和具体中介者类 SubConcreteMediator，代码如下所示：  

```
//文本标签类：具体同事类
class Label extends Component {
	public void update() {
		System.out.println("文本标签内容改变，客户信息总数加1。");
	}
}

//新增具体中介者类
class SubConcreteMediator extends ConcreteMediator {
	//增加对Label对象的引用
	public Label label;
	
	public void componentChanged(Component c) {
	    //单击按钮
if(c == addButton) {
			System.out.println("--单击增加按钮--");
			list.update();
			cb.update();
			userNameTextBox.update();
			label.update(); //文本标签更新
		}
        //从列表框选择客户
		else if(c == list) {
			System.out.println("--从列表框选择客户--");
			cb.select();
			userNameTextBox.setText();
		}
        //从组合框选择客户
		else if(c == cb) {
			System.out.println("--从组合框选择客户--");
			cb.select();
			userNameTextBox.setText();
		}
	}
}
```
修改客户端测试代码：  

```
class Client {
	public static void main(String args[]) {
        //用新增具体中介者定义中介者对象
		SubConcreteMediator mediator;
		mediator = new SubConcreteMediator();
		
		Button addBT = new Button();
		List list = new List();
	    ComboBox cb = new ComboBox();
	    TextBox userNameTB = new TextBox();
	    Label label = new Label();

		addBT.setMediator(mediator);
		list.setMediator(mediator);
		cb.setMediator(mediator);
		userNameTB.setMediator(mediator);
		label.setMediator(mediator);
		
		mediator.addButton = addBT;
		mediator.list = list;
		mediator.cb = cb;
		mediator.userNameTextBox = userNameTB;
		mediator.label = label;
			
		addBT.changed();
		System.out.println("-----------------------------");
		list.changed();
	}
}
```  
编译并运行程序，输出结果如下：  
```
--单击增加按钮--
列表框增加一项：张无忌。
组合框增加一项：张无忌。
客户信息增加成功后文本框清空。
文本标签内容改变，客户信息总数加1。
-----------------------------
--从列表框选择客户--
组合框选中项：小龙女。
文本框显示：小龙女。
```

由于在本实例中不同的组件类（即不同的同事类）所拥有的方法并不完全相同，因此中介者类没有针对抽象同事类编程，导致在具体中介者类中需要维持对具体同事类的引用，客户端代码无法完全透明地对待所有同事类和中介者类。在某些情况下，如果设计得当，可以在客户端透明地对同事类和中介者类编程，这样系统将具有更好的灵活性和可扩展性。  

**思考**  
如果不使用中介者模式，按照图所示设计方案，增加新组件时原有系统该如何修改？  

在中介者模式的实际使用过程中，**如果需要引入新的具体同事类**，只需要继承抽象同事类并实现其中的方法即可，由于具体同事类之间并无直接的引用关系，因此原有所有同事类无须进行任何修改，它们与新增同事对象之间的交互可以通过修改或者增加具体中介者类来实现；**如果需要在原有系统中增加新的具体中介者类**，只需要继承抽象中介者类（或已有的具体中介者类）并覆盖其中定义的方法即可，在新的具体中介者中可以通过不同的方式来处理对象之间的交互，也可以增加对新增同事的引用和调用。在客户端中只需要修改少许代码（如果引入配置文件的话有时可以不修改任何代码）就可以实现中介者的更换。
