# 操作复杂对象结构——访问者模式（三）  

## 完整解决方案  

Sunny 软件公司开发人员使用访问者模式对 OA 系统中员工数据汇总模块进行重构，使得系统可以很方便地增加新类型的访问者，更加符合“单一职责原则”和“开闭原则”，重构后的基本结构如图所示：  

![](images/1333714391_6314.gif)  

在图中，FADepartment 表示财务部，HRDepartment 表示人力资源部，它们充当具体访问者角色，其抽象父类 Department 充当抽象访问者角色；EmployeeList 充当对象结构，用于存储员工列表；FulltimeEmployee 表示正式员工，ParttimeEmployee 表示临时工，它们充当具体元素角色，其父接口 Employee 充当抽象元素角色。完整代码如下所示：

```
import java.util.*;

//员工类：抽象元素类
interface Employee
{
	public void accept(Department handler); //接受一个抽象访问者访问
}

//全职员工类：具体元素类
class FulltimeEmployee implements Employee
{
	private String name;
	private double weeklyWage;
	private int workTime;

	public FulltimeEmployee(String name,double weeklyWage,int workTime)
	{
		this.name = name;
		this.weeklyWage = weeklyWage;
		this.workTime = workTime;
	}	

	public void setName(String name) 
    {
		this.name = name; 
	}

	public void setWeeklyWage(double weeklyWage) 
    {
		this.weeklyWage = weeklyWage; 
	}

	public void setWorkTime(int workTime) 
    {
		this.workTime = workTime; 
	}

	public String getName() 
    {
		return (this.name); 
	}

	public double getWeeklyWage() 
    {
		return (this.weeklyWage); 
	}

	public int getWorkTime() 
    {
		return (this.workTime); 
	}

	public void accept(Department handler)
    {
		handler.visit(this); //调用访问者的访问方法
	}
}

//兼职员工类：具体元素类
class ParttimeEmployee implements Employee
{
	private String name;
	private double hourWage;
	private int workTime;

	public ParttimeEmployee(String name,double hourWage,int workTime)
	{
		this.name = name;
		this.hourWage = hourWage;
		this.workTime = workTime;
	}	

	public void setName(String name) 
    {
		this.name = name; 
	}

	public void setHourWage(double hourWage) 
    {
		this.hourWage = hourWage; 
	}

	public void setWorkTime(int workTime) 
    {
		this.workTime = workTime; 
	}

	public String getName() 
    {
		return (this.name); 
	}

	public double getHourWage() 
    {
		return (this.hourWage); 
	}

	public int getWorkTime() 
    {
		return (this.workTime); 
	}

	public void accept(Department handler)
    {
		handler.visit(this); //调用访问者的访问方法
	}
}

//部门类：抽象访问者类
abstract class Department
{
    //声明一组重载的访问方法，用于访问不同类型的具体元素
	public abstract void visit(FulltimeEmployee employee);
	public abstract void visit(ParttimeEmployee employee);	
}

//财务部类：具体访问者类
class FADepartment extends Department
{
    //实现财务部对全职员工的访问
	public void visit(FulltimeEmployee employee)
	{
		int workTime = employee.getWorkTime();
		double weekWage = employee.getWeeklyWage();
		if(workTime > 40)
		{
			weekWage = weekWage + (workTime - 40) * 100;
		}
		else if(workTime < 40)
		{
			weekWage = weekWage - (40 - workTime) * 80;
			if(weekWage < 0)
			{
				weekWage = 0;
			}
		}
		System.out.println("正式员工" + employee.getName() + "实际工资为：" + weekWage + "元。");			
	}

    //实现财务部对兼职员工的访问
	public void visit(ParttimeEmployee employee)
	{
		int workTime = employee.getWorkTime();
		double hourWage = employee.getHourWage();
		System.out.println("临时工" + employee.getName() + "实际工资为：" + workTime * hourWage + "元。");		
	}		
}

//人力资源部类：具体访问者类
class HRDepartment extends Department
{
    //实现人力资源部对全职员工的访问
	public void visit(FulltimeEmployee employee)
	{
		int workTime = employee.getWorkTime();
		System.out.println("正式员工" + employee.getName() + "实际工作时间为：" + workTime + "小时。");
		if(workTime > 40)
		{
			System.out.println("正式员工" + employee.getName() + "加班时间为：" + (workTime - 40) + "小时。");
		}
		else if(workTime < 40)
		{
			System.out.println("正式员工" + employee.getName() + "请假时间为：" + (40 - workTime) + "小时。");
		}						
	}

    //实现人力资源部对兼职员工的访问
	public void visit(ParttimeEmployee employee)
	{
		int workTime = employee.getWorkTime();
		System.out.println("临时工" + employee.getName() + "实际工作时间为：" + workTime + "小时。");
	}		
}

//员工列表类：对象结构
class EmployeeList
{
    //定义一个集合用于存储员工对象
	private ArrayList<Employee> list = new ArrayList<Employee>();

	public void addEmployee(Employee employee)
	{
		list.add(employee);
	}

    //遍历访问员工集合中的每一个员工对象
	public void accept(Department handler)
	{
		for(Object obj : list)
		{
			((Employee)obj).accept(handler);
		}
	}
}
```

为了提高系统的灵活性和可扩展性，我们将具体访问者类的类名存储在配置文件中，并通过工具类 XMLUtil 来读取配置文件并反射生成对象，XMLUtil 类的代码如下所示：

```
import javax.xml.parsers.*;
import org.w3c.dom.*;
import org.xml.sax.SAXException;
import java.io.*;
class XMLUtil
{
    //该方法用于从XML配置文件中提取具体类类名，并返回一个实例对象
    public static Object getBean()
    {
		try
		{
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
        catch(Exception e)
        {
           	e.printStackTrace();
           	return null;
       	}
    }
}
```

配置文件config.xml中存储了具体访问者类的类名，代码如下所示：  

```
<?xml version="1.0"?>
<config>
	<className>FADepartment</className>
</config>
```

编写如下客户端测试代码：  
 
```
class Client
{
	public static void main(String args[])
	{
		EmployeeList list = new EmployeeList();
		Employee fte1,fte2,fte3,pte1,pte2;

		fte1 = new FulltimeEmployee("张无忌",3200.00,45);
		fte2 = new FulltimeEmployee("杨过",2000.00,40);
		fte3 = new FulltimeEmployee("段誉",2400.00,38);
		pte1 = new ParttimeEmployee("洪七公",80.00,20);
		pte2 = new ParttimeEmployee("郭靖",60.00,18);

		list.addEmployee(fte1);
		list.addEmployee(fte2);
		list.addEmployee(fte3);
		list.addEmployee(pte1);
		list.addEmployee(pte2);

		Department dep;
		dep = (Department)XMLUtil.getBean();
		list.accept(dep);
	}
}
```

编译并运行程序，输出结果如下：  

```
正式员工张无忌实际工资为：3700.0元。
正式员工杨过实际工资为：2000.0元。
正式员工段誉实际工资为：2240.0元。
临时工洪七公实际工资为：1600.0元。
临时工郭靖实际工资为：1080.0元。
```

如果需要更换具体访问者类，无须修改源代码，只需修改配置文件，例如将访问者类由财务部改为人力资源部，只需将存储在配置文件中的具体访问者类 FADepartment 改为 HRDepartment，如下代码所示：  

```
<?xml version="1.0"?>
<config>
    <className>HRDepartment</className>
</config>
```

重新运行客户端程序，输出结果如下：  
```
正式员工张无忌实际工作时间为：45小时。
正式员工张无忌加班时间为：5小时。
正式员工杨过实际工作时间为：40小时。
正式员工段誉实际工作时间为：38小时。
正式员工段誉请假时间为：2小时。
临时工洪七公实际工作时间为：20小时。
临时工郭靖实际工作时间为：18小时。
```

如果要在系统中增加一种新的访问者，无须修改源代码，只要增加一个新的具体访问者类即可，在该具体访问者中封装了新的操作元素对象的方法。从增加新的访问者的角度来看，访问者模式符合“开闭原则”。  

如果要在系统中增加一种新的具体元素，例如增加一种新的员工类型为“退休人员”，由于原有系统并未提供相应的访问接口（在抽象访问者中没有声明任何访问“退休人员”的方法），因此必须对原有系统进行修改，在原有的抽象访问者类和具体访问者类中增加相应的访问方法。从增加新的元素的角度来看，访问者模式违背了“开闭原则”。  

综上所述，访问者模式与抽象工厂模式类似，对“开闭原则”的支持具有倾斜性，可以很方便地添加新的访问者，但是添加新的元素较为麻烦。
