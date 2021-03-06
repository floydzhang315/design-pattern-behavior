# 算法的封装与切换——策略模式（三）  

## 完整解决方案  

为了实现打折算法的复用，并能够灵活地向系统中增加新的打折方式，Sunny 软件公司开发人员使用策略模式对电影院打折方案进行重构，重构后基本结构如图所示：  

![](images/1343811809_8784.jpg)  

在图中，MovieTicket 充当环境类角色，Discount 充当抽象策略角色，StudentDiscount、 ChildrenDiscount 和 VIPDiscount 充当具体策略角色。完整代码如下所示：  

```
//电影票类：环境类
class MovieTicket {
	private double price;
	private Discount discount; //维持一个对抽象折扣类的引用

	public void setPrice(double price) {
		this.price = price;
	}

    //注入一个折扣类对象
	public void setDiscount(Discount discount) {
		this.discount = discount;
	}

	public double getPrice() {
        //调用折扣类的折扣价计算方法
		return discount.calculate(this.price);
	}
}

//折扣类：抽象策略类
interface Discount {
	public double calculate(double price);
}

//学生票折扣类：具体策略类
class StudentDiscount implements Discount {
	public double calculate(double price) {
		System.out.println("学生票：");
		return price * 0.8;
	}
} 

//儿童票折扣类：具体策略类
class ChildrenDiscount implements Discount {
	public double calculate(double price) {
		System.out.println("儿童票：");
		return price - 10;
	}
} 

//VIP会员票折扣类：具体策略类
class VIPDiscount implements Discount {
	public double calculate(double price) {
		System.out.println("VIP票：");
		System.out.println("增加积分！");
		return price * 0.5;
	}
}
```

为了提高系统的灵活性和可扩展性，我们将具体策略类的类名存储在配置文件中，并通过工具类 XMLUtil 来读取配置文件并反射生成对象，XMLUtil 类的代码如下所示：

```
import javax.xml.parsers.*;
import org.w3c.dom.*;
import org.xml.sax.SAXException;
import java.io.*;
class XMLUtil {
//该方法用于从XML配置文件中提取具体类类名，并返回一个实例对象
	public static Object getBean() {
		try {
			//创建文档对象
			DocumentBuilderFactory dFactory = DocumentBuilderFactory.newInstance();
			DocumentBuilder builder = dFactory.newDocumentBuilder();
			Document doc;							
			doc = builder.parse(new File("config.xml")); 
		
			//获取包含类名的文本节点
			NodeList nl = doc.getElementsByTagName("className");
            Node classNode=nl.item(0).getFirstChild();
            String cName=classNode.getNodeValue();
            
            //通过类名生成实例对象并将其返回
            Class c=Class.forName(cName);
	  	    Object obj=c.newInstance();
            return obj;
        }   
        catch(Exception e) {
           	e.printStackTrace();
           	return null;
       	}
    }
}
```

在配置文件 config.xml 中存储了具体策略类的类名，代码如下所示：  

```
<?xml version="1.0"?>  
<config>  
    <className>StudentDiscount</className>  
</config>  
```

编写如下客户端测试代码：  

```
class Client {
	public static void main(String args[]) {
		MovieTicket mt = new MovieTicket();
		double originalPrice = 60.0;
		double currentPrice;
		
		mt.setPrice(originalPrice);
		System.out.println("原始价为：" + originalPrice);
		System.out.println("---------------------------------");
			
		Discount discount;
		discount = (Discount)XMLUtil.getBean(); //读取配置文件并反射生成具体折扣对象
		mt.setDiscount(discount); //注入折扣对象
		
		currentPrice = mt.getPrice();
		System.out.println("折后价为：" + currentPrice);
	}
}
```

编译并运行程序，输出结果如下：  

```
原始价为：60.0
---------------------------------
学生票：
折后价为：48.0
```

如果需要更换具体策略类，无须修改源代码，只需修改配置文件，例如将学生票改为儿童票，只需将存储在配置文件中的具体策略类 StudentDiscount 改为 ChildrenDiscount，如下代码所示：

```
<?xml version="1.0"?>
<config>
    <className>ChildrenDiscount</className>
</config>
```

重新运行客户端程序，输出结果如下：  

```
原始价为：60.0
---------------------------------
儿童票：
折后价为：50.0
```

如果需要增加新的打折方式，原有代码均无须修改，只要增加一个新的折扣类作为抽象折扣类的子类，实现在抽象折扣类中声明的打折方法，然后修改配置文件，将原有具体折扣类类名改为新增折扣类类名即可，完全符合“开闭原则”。