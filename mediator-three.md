# 协调多个对象之间的交互——中介者模式（三）  
## 完整解决方案  

为了协调界面组件对象之间的复杂交互关系，Sunny 公司开发人员使用中介者模式来设计客户信息管理窗口，其结构示意图如图所示：  

![引入了中介者类的“客户信息管理窗口”结构示意图](images/1357652393_2992.jpg)  

上图只是一个重构之后的结构示意图，在具体实现时，为了确保系统具有更好的灵活性和可扩展性，我们需要定义抽象中介者和抽象组件类，其中抽象组件类是所有具体组件类的公共父类，完整类图如图所示：  

![重构后的“客户信息管理窗口”结构图](images/1357652403_1841.jpg)  

在图中，Component 充当抽象同事类，Button、List、ComboBox 和 TextBox 充当具体同事类，Mediator 充当抽象中介者类，ConcreteMediator 充当具体中介者类，ConcreteMediator 维持了对具体同事类的引用，为了简化 ConcreteMediator 类的代码，我们在其中只定义了一个 Button 对象和一个 TextBox 对象。完整代码如下所示：  

```
//抽象中介者
abstract class Mediator {
	public abstract void componentChanged(Component c);
}

//具体中介者
class ConcreteMediator extends Mediator {
	//维持对各个同事对象的引用
	public Button addButton;
	public List list;
	public TextBox userNameTextBox;
	public ComboBox cb;

    //封装同事对象之间的交互
	public void componentChanged(Component c) {
		//单击按钮
if(c == addButton) {
			System.out.println("--单击增加按钮--");
			list.update();
			cb.update();
			userNameTextBox.update();
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

//抽象组件类：抽象同事类
abstract class Component {
	protected Mediator mediator;
	
	public void setMediator(Mediator mediator) {
		this.mediator = mediator;
	}

	//转发调用
	public void changed() {
		mediator.componentChanged(this);
	}
	
	public abstract void update();	
}

//按钮类：具体同事类
class Button extends Component {
	public void update() {
		//按钮不产生交互
	}
}

//列表框类：具体同事类
class List extends Component {
	public void update() {
		System.out.println("列表框增加一项：张无忌。");
	}
	
	public void select() {
		System.out.println("列表框选中项：小龙女。");
	}
}

//组合框类：具体同事类
class ComboBox extends Component {
	public void update() {
		System.out.println("组合框增加一项：张无忌。");
	}
	
	public void select() {
		System.out.println("组合框选中项：小龙女。");
	}
}

//文本框类：具体同事类
class TextBox extends Component {
	public void update() {
		System.out.println("客户信息增加成功后文本框清空。");
	}
	
	public void setText() {
		System.out.println("文本框显示：小龙女。");
	}
}
```
编写如下客户端测试代码：

```
class Client {
	public static void main(String args[]) {
        //定义中介者对象
		ConcreteMediator mediator;
		mediator = new ConcreteMediator();
		
        //定义同事对象
		Button addBT = new Button();
		List list = new List();
	    ComboBox cb = new ComboBox();
	    TextBox userNameTB = new TextBox();

		addBT.setMediator(mediator);
		list.setMediator(mediator);
		cb.setMediator(mediator);
		userNameTB.setMediator(mediator);

		mediator.addButton = addBT;
		mediator.list = list;
		mediator.cb = cb;
		mediator.userNameTextBox = userNameTB;
		
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
-----------------------------
--从列表框选择客户--
组合框选中项：小龙女。
文本框显示：小龙女。
```
