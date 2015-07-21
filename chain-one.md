 “一对二”，“过”，“过”……这声音熟悉吗？你会想到什么？对！纸牌。在类似“斗地主”这样的纸牌游戏中，某人出牌给他的下家，下家看看手中的牌，如果要不起上家的牌则将出牌请求再转发给他的下家，其下家再进行判断。一个循环下来，如果其他人都要不起该牌，则最初的出牌者可以打出新的牌。在这个过程中，牌作为一个请求沿着一条链在传递，每一位纸牌的玩家都可以处理该请求。在设计模式中，我们也有一种专门用于处理这种请求链式传递的模式，它就是本章将要介绍的职责链模式。

## 1 采购单的分级审批   
Sunny 软件公司承接了某企业 SCM（Supply Chain Management，供应链管理）系统的开发任务，其中包含一个采购审批子系统。该企业的采购审批是分级进行的，即根据采购金额的不同由不同层次的主管人员来审批，主任可以审批 5 万元以下（不包括 5 万元）的采购单，副董事长可以审批 5 万元至 10 万元（不包括 10 万元）的采购单，董事长可以审批 10 万元至 50 万元（不包括 50 万元）的采购单，50 万元及以上的采购单就需要开董事会讨论决定。如图所示：
![采购单分级审批示意图][images/1333307283_7751.gif]  
如何在软件中实现采购单的分级审批？Sunny 软件公司开发人员提出了一个初始解决方案，在系统中提供一个采购单处理类 PurchaseRequestHandler 用于统一处理采购单，其框架代码如下所示：  
```
//采购单处理类
class PurchaseRequestHandler {
	//递交采购单给主任
	public void sendRequestToDirector(PurchaseRequest request) {
		if (request.getAmount() < 50000) {
			//主任可审批该采购单
			this.handleByDirector(request);
		}
		else if (request.getAmount() < 100000) {
			//副董事长可审批该采购单
			this.handleByVicePresident(request);
		}
		else if (request.getAmount() < 500000) {
			//董事长可审批该采购单
			this.handleByPresident(request);
		}
		else {
			//董事会可审批该采购单
			this.handleByCongress(request);
		}
	}
	
	//主任审批采购单
	public void handleByDirector(PurchaseRequest request) {
		//代码省略
	}
	
	//副董事长审批采购单
	public void handleByVicePresident(PurchaseRequest request) {
		//代码省略
	}
	
	//董事长审批采购单
	public void handleByPresident(PurchaseRequest request) {
		//代码省略
	}
	
	//董事会审批采购单
	public void handleByCongress(PurchaseRequest request) {
		//代码省略
	}
}
```
问题貌似很简单，但仔细分析，发现上述方案存在如下几个问题：  

(1)PurchaseRequestHandler 类较为庞大，各个级别的审批方法都集中在一个类中，违反了“单一职责原则”，测试和维护难度大。  

(2)如果需要增加一个新的审批级别或调整任何一级的审批金额和审批细节（例如将董事长的审批额度改为 60 万元）时都必须修改源代码并进行严格测试，此外，如果需要移除某一级别（例如金额为 10 万元及以上的采购单直接由董事长审批，不再设副董事长一职）时也必须对源代码进行修改，违反了“开闭原则”。  

(3)审批流程的设置缺乏灵活性，现在的审批流程是“主任-->副董事长-->董事长-->董事会”，如果需要改为“主任-->董事长-->董事会”，在此方案中只能通过修改源代码来实现，客户端无法定制审批流程。  
如何针对上述问题对系统进行改进？Sunny 公司开发人员迫切需要一种新的设计方案，还好有职责链模式，通过使用职责链模式我们可以最大程度地解决这些问题，下面让我们正式进入职责链模式的学习。