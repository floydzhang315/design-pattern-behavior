## 撤销功能的实现——备忘录模式（四）  

**4 实现多次撤销**  

Sunny 软件公司开发人员通过使用备忘录模式实现了中国象棋棋子的撤销操作，但是使用上述代码只能实现一次撤销，因为在负责人类中只定义一个备忘录对象来保存状态，后面保存的状态会将前一次保存的状态覆盖，但有时候用户需要撤销多步操作。如何实现多次撤销呢？本节将提供一种多次撤销的解决方案，那就是在负责人类中定义一个集合来存储多个备忘录，每个备忘录负责保存一个历史状态，在撤销时可以对备忘录集合进行逆向遍历，回到一个指定的历史状态，而且还可以对备忘录集合进行正向遍历，实现重做（Redo）操作，即取消撤销，让对象状态得到恢复。  

改进之后的中国象棋棋子撤销功能结构图如图所示：  

![](images/1335892489_9232.jpg)  

在图中，我们对负责人类 MementoCaretaker 进行了修改，在其中定义了一个 ArrayList 类型的集合对象来存储多个备忘录，其代码如下所示：  

```
import java.util.*;

class MementoCaretaker {
    //定义一个集合来存储多个备忘录
	private ArrayList mementolist = new ArrayList();

	public ChessmanMemento getMemento(int i) {
		return (ChessmanMemento)mementolist.get(i);
	}

	public void setMemento(ChessmanMemento memento) {
		mementolist.add(memento);
	}
}
```
编写如下客户端测试代码：

```
class Client {
private static int index = -1; //定义一个索引来记录当前状态所在位置
	private static MementoCaretaker mc = new MementoCaretaker();

	public static void main(String args[]) {
		Chessman chess = new Chessman("车",1,1);
		play(chess);		
		chess.setY(4);
		play(chess);
		chess.setX(5);
		play(chess);	
		undo(chess,index);
		undo(chess,index);	
		redo(chess,index);
		redo(chess,index);
	}
	
    //下棋
	public static void play(Chessman chess) {
		mc.setMemento(chess.save()); //保存备忘录
		index ++; 
		System.out.println("棋子" + chess.getLabel() + "当前位置为：" + "第" + chess.getX() + "行" + "第" + chess.getY() + "列。");
	}

	//悔棋
	public static void undo(Chessman chess,int i) {
		System.out.println("******悔棋******");
		index --; 
		chess.restore(mc.getMemento(i-1)); //撤销到上一个备忘录
		System.out.println("棋子" + chess.getLabel() + "当前位置为：" + "第" + chess.getX() + "行" + "第" + chess.getY() + "列。");
	}

	//撤销悔棋
	public static void redo(Chessman chess,int i) {
		System.out.println("******撤销悔棋******");	
		index ++; 
		chess.restore(mc.getMemento(i+1)); //恢复到下一个备忘录
		System.out.println("棋子" + chess.getLabel() + "当前位置为：" + "第" + chess.getX() + "行" + "第" + chess.getY() + "列。");
	}
} 
```

编译并运行程序，输出结果如下：  

```
棋子车当前位置为：第1行第1列。
棋子车当前位置为：第1行第4列。
棋子车当前位置为：第5行第4列。
******悔棋******
棋子车当前位置为：第1行第4列。
******悔棋******
棋子车当前位置为：第1行第1列。
******撤销悔棋******
棋子车当前位置为：第1行第4列。
******撤销悔棋******
棋子车当前位置为：第5行第4列。
```

**扩展**  
本实例只能实现最简单的 Undo 和 Redo 操作，并未考虑对象状态在操作过程中出现分支的情况。如果在撤销到某个历史状态之后，用户再修改对象状态，此后执行 Undo 操作时可能会发生对象状态错误，大家可以思考其产生原因。【注：可将对象状态的改变绘制成一张树状图进行分析。】  

在实际开发中，可以使用链表或者堆栈来处理有分支的对象状态改变，大家可通过链表或者堆栈对上述实例进行改进。