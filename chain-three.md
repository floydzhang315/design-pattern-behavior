# 请求的链式处理——职责链模式（三）  
## 完整解决方案  
为了让采购单的审批流程更加灵活，并实现采购单的链式传递和处理，Sunny 公司开发人员使用职责链模式来实现采购单的分级审批，其基本结构如图所示：
![采购单分级审批结构图](images/1333307860_9326.gif)  
在图中，抽象类 Approver 充当抽象处理者（抽象传递者），Director、VicePresident、President 和 Congress 充当具体处理者（具体传递者），PurchaseRequest 充当请求类。完整代码如下所示：
```
//采购单：请求类
class PurchaseRequest {
	private double amount;  //采购金额
	private int number;  //采购单编号
	private String purpose;  //采购目的
	
	public PurchaseRequest(double amount, int number, String purpose) {
		this.amount = amount;
		this.number = number;
		this.purpose = purpose;
	}
	
	public void setAmount(double amount) {
		this.amount = amount;
	}
	
	public double getAmount() {
		return this.amount;
	}
	
	public void setNumber(int number) {
		this.number = number;
	}
	
	public int getNumber() {
		return this.number;
	}
	
	public void setPurpose(String purpose) {
		this.purpose = purpose;
	}
	
	public String getPurpose() {
		return this.purpose;
	}
}

//审批者类：抽象处理者
abstract class Approver {
	protected Approver successor; //定义后继对象
	protected String name; //审批者姓名
	
	public Approver(String name) {
		this.name = name;
	}

	//设置后继者
	public void setSuccessor(Approver successor) { 
		this.successor = successor;
	}

    //抽象请求处理方法
    public abstract void processRequest(PurchaseRequest request);
}

//主任类：具体处理者
class Director extends Approver {
	public Director(String name) {
		super(name);
	}
	
    //具体请求处理方法
 	public void processRequest(PurchaseRequest request) {
 		if (request.getAmount() < 50000) {
 			System.out.println("主任" + this.name + "审批采购单：" + request.getNumber() + "，金额：" + request.getAmount() + "元，采购目的：" + request.getPurpose() + "。");  //处理请求
 		}
 		else {
 			this.successor.processRequest(request);  //转发请求
 		}	
 	}
}

//副董事长类：具体处理者
class VicePresident extends Approver {
	public VicePresident(String name) {
		super(name);
	}
	
    //具体请求处理方法
 	public void processRequest(PurchaseRequest request) {
 		if (request.getAmount() < 100000) {
 			System.out.println("副董事长" + this.name + "审批采购单：" + request.getNumber() + "，金额：" + request.getAmount() + "元，采购目的：" + request.getPurpose() + "。");  //处理请求
 		}
 		else {
 			this.successor.processRequest(request);  //转发请求
 		}	
 	}
}

//董事长类：具体处理者
class President extends Approver {
	public President(String name) {
		super(name);
	}
	
    //具体请求处理方法
 	public void processRequest(PurchaseRequest request) {
 		if (request.getAmount() < 500000) {
 			System.out.println("董事长" + this.name + "审批采购单：" + request.getNumber() + "，金额：" + request.getAmount() + "元，采购目的：" + request.getPurpose() + "。");  //处理请求
 		}
 		else {
 			this.successor.processRequest(request);  //转发请求
 		}
 	}
}

//董事会类：具体处理者
class Congress extends Approver {
	public Congress(String name) {
		super(name);
	}
	
    //具体请求处理方法
 	public void processRequest(PurchaseRequest request) {
 		System.out.println("召开董事会审批采购单：" + request.getNumber() + "，金额：" + request.getAmount() + "元，采购目的：" + request.getPurpose() + "。");	    //处理请求
 	}    
}
```

编写如下客户端测试代码：

```
class Client {
	public static void main(String[] args) {
		Approver wjzhang,gyang,jguo,meeting;
		wjzhang = new Director("张无忌");
		gyang = new VicePresident("杨过");
		jguo = new President("郭靖");
		meeting = new Congress("董事会");
	
		//创建职责链
		wjzhang.setSuccessor(gyang);
		gyang.setSuccessor(jguo);
		jguo.setSuccessor(meeting);
		
		//创建采购单
		PurchaseRequest pr1 = new PurchaseRequest(45000,10001,"购买倚天剑");
		wjzhang.processRequest(pr1);
		
		PurchaseRequest pr2 = new PurchaseRequest(60000,10002,"购买《葵花宝典》");
		wjzhang.processRequest(pr2);
	
		PurchaseRequest pr3 = new PurchaseRequest(160000,10003,"购买《金刚经》");
		wjzhang.processRequest(pr3);

		PurchaseRequest pr4 = new PurchaseRequest(800000,10004,"购买桃花岛");
		wjzhang.processRequest(pr4);
	}
} 
```

编译并运行程序，输出结果如下：  
```
主任张无忌审批采购单：10001，金额：45000.0元，采购目的：购买倚天剑。
副董事长杨过审批采购单：10002，金额：60000.0元，采购目的：购买《葵花宝典》。
董事长郭靖审批采购单：10003，金额：160000.0元，采购目的：购买《金刚经》。
召开董事会审批采购单：10004，金额：800000.0元，采购目的：购买桃花岛。
```

如果需要在系统增加一个新的具体处理者，如增加一个经理（Manager）角色可以审批 5 万元至 8 万元（不包括 8 万元）的采购单，需要编写一个新的具体处理者类 Manager，作为抽象处理者类 Approver 的子类，实现在 Approver 类中定义的抽象处理方法，如果采购金额大于等于 8 万元，则将请求转发给下家，代码如下所示：
```
//经理类：具体处理者
class Manager extends Approver {
	public Manager(String name) {
		super(name);
	}
	
    //具体请求处理方法
 	public void processRequest(PurchaseRequest request) {
 		if (request.getAmount() < 80000) {
 			System.out.println("经理" + this.name + "审批采购单：" + request.getNumber() + "，金额：" + request.getAmount() + "元，采购目的：" + request.getPurpose() + "。");  //处理请求
 		}
 		else {
 			this.successor.processRequest(request);  //转发请求
 		}	
 	}
}
```

由于链的创建过程由客户端负责，因此增加新的具体处理者类对原有类库无任何影响，无须修改已有类的源代码，符合“开闭原则”。  

在客户端代码中，如果要将新的具体请求处理者应用在系统中，需要创建新的具体处理者对象，然后将该对象加入职责链中。如在客户端测试代码中增加如下代码：

```
Approver rhuang;
rhuang = new Manager("黄蓉");
```
将建链代码改为：
```
//创建职责链
wjzhang.setSuccessor(rhuang); //将“黄蓉”作为“张无忌”的下家
rhuang.setSuccessor(gyang); //将“杨过”作为“黄蓉”的下家
gyang.setSuccessor(jguo);
jguo.setSuccessor(meeting);
```
重新编译并运行程序，输出结果如下：
```
主任张无忌审批采购单：10001，金额：45000.0元，采购目的：购买倚天剑。
经理黄蓉审批采购单：10002，金额：60000.0元，采购目的：购买《葵花宝典》。
董事长郭靖审批采购单：10003，金额：160000.0元，采购目的：购买《金刚经》。
召开董事会审批采购单：10004，金额：800000.0元，采购目的：购买桃花岛。
```

**思考**  
如果将审批流程由“主任-->副董事长-->董事长-->董事会”调整为“主任-->董事长-->董事会”，系统将做出哪些改动？预测修改之后客户端代码的输出结果。
