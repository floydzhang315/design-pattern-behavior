# 撤销功能的实现——备忘录模式（三）  

## 完整解决方案  

为了实现撤销功能，Sunny 公司开发人员决定使用备忘录模式来设计中国象棋软件，其基本结构如图

在图中，Chessman 充当原发器，ChessmanMemento 充当备忘录，MementoCaretaker 充当负责人，在 MementoCaretaker 中定义了一个 ChessmanMemento 类型的对象，用于存储备忘录。完整代码如下所示：

```
//象棋棋子类：原发器
class Chessman {
	private String label;
	private int x;
	private int y;

	public Chessman(String label,int x,int y) {
		this.label = label;
		this.x = x;
		this.y = y;
	}

	public void setLabel(String label) {
		this.label = label; 
	}

	public void setX(int x) {
		this.x = x; 
	}

	public void setY(int y) {
		this.y = y; 
	}

	public String getLabel() {
		return (this.label); 
	}

	public int getX() {
		return (this.x); 
	}

	public int getY() {
		return (this.y); 
	}
	
    //保存状态
	public ChessmanMemento save() {
		return new ChessmanMemento(this.label,this.x,this.y);
	}
	
    //恢复状态
	public void restore(ChessmanMemento memento) {
		this.label = memento.getLabel();
		this.x = memento.getX();
		this.y = memento.getY();
	}
}

//象棋棋子备忘录类：备忘录
class ChessmanMemento {
	private String label;
	private int x;
	private int y;

	public ChessmanMemento(String label,int x,int y) {
		this.label = label;
		this.x = x;
		this.y = y;
	}

	public void setLabel(String label) {
		this.label = label; 
	}

	public void setX(int x) {
		this.x = x; 
	}

	public void setY(int y) {
		this.y = y; 
	}

	public String getLabel() {
		return (this.label); 
	}

	public int getX() {
		return (this.x); 
	}

	public int getY() {
		return (this.y); 
	}	
}

//象棋棋子备忘录管理类：负责人
class MementoCaretaker {
	private ChessmanMemento memento;

	public ChessmanMemento getMemento() {
		return memento;
	}

	public void setMemento(ChessmanMemento memento) {
		this.memento = memento;
	}
}
```
编写如下客户端测试代码：

```
class Client {
	public static void main(String args[]) {
		MementoCaretaker mc = new MementoCaretaker();
		Chessman chess = new Chessman("车",1,1);
		display(chess);
		mc.setMemento(chess.save()); //保存状态		
		chess.setY(4);
		display(chess);
		mc.setMemento(chess.save()); //保存状态
		display(chess);
		chess.setX(5);
		display(chess);
		System.out.println("******悔棋******");	
		chess.restore(mc.getMemento()); //恢复状态
		display(chess);
	}
	
	public static void display(Chessman chess) {
		System.out.println("棋子" + chess.getLabel() + "当前位置为：" + "第" + chess.getX() + "行" + "第" + chess.getY() + "列。");
	}
}
```
编译并运行程序，输出结果如下：  

```
棋子车当前位置为：第1行第1列。
棋子车当前位置为：第1行第4列。
棋子车当前位置为：第1行第4列。
棋子车当前位置为：第5行第4列。
******悔棋******
棋子车当前位置为：第1行第4列。
```

  