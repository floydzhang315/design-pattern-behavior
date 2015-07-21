## 对象间的联动——观察者模式（五）  

**5 观察者模式与Java事件处理**  

JDK 1.0 及更早版本的事件模型基于职责链模式，但是这种模型不适用于复杂的系统，因此在JDK 1.1 及以后的各个版本中，事件处理模型采用基于观察者模式的委派事件模型（DelegationEvent Model, DEM），即一个 Java 组件所引发的事件并不由引发事件的对象自己来负责处理，而是委派给独立的事件处理对象负责。  

在 DEM 模型中，目标角色（如界面组件）负责发布事件，而观察者角色（事件处理者）可以向目标订阅它所感兴趣的事件。当一个具体目标产生一个事件时，它将通知所有订阅者。事件的发布者称为事件源（Event Source），而订阅者称为事件监听器（Event Listener），在这个过程中还可以通过事件对象（Event Object）来传递与事件相关的信息，可以在事件监听者的实现类中实现事件处理，因此事件监听对象又可以称为事件处理对象。**事件源对象、事件监听对象（事件处理对象）和事件对象构成了 Java 事件处理模型的三要素。**事件源对象充当观察目标，而事件监听对象充当观察者。以按钮点击事件为例，其事件处理流程如下：  

(1) 如果用户在 GUI 中单击一个按钮，将触发一个事件（如A ctionEvent 类型的动作事件），JVM 将产生一个相应的 ActionEvent 类型的事件对象，在该事件对象中包含了有关事件和事件源的信息，此时按钮是事件源对象；  

(2) 将 ActionEvent 事件对象传递给事件监听对象（事件处理对象），JDK提供了专门用于处理 ActionEvent 事件的接口 ActionListener，开发人员需提供一个 ActionListener 的实现类（如MyActionHandler），实现在 ActionListener 接口中声明的抽象事件处理方法 actionPerformed()，对所发生事件做出相应的处理；  

(3) 开发人员将 ActionListener 接口的实现类（如 MyActionHandler）对象注册到按钮中，可以通过按钮类的 addActionListener() 方法来实现注册；  

(4) JVM在触发事件时将调用按钮的 fireXXX() 方法，在该方法内部将调用注册到按钮中的事件处理对象的 actionPerformed() 方法，实现对事件的处理。  

使用类似的方法，我们可自定义 GUI 组件，如包含两个文本框和两个按钮的登录组件 LoginBean，可以采用如图所示设计方案：  

![自定义登录组件结构图【省略按钮、文本框等界面组件】][images/1341504872_7751.jpg]  
  
相关类说明如下：  

(1) LoginEvent 是事件类，它用于封装与事件有关的信息，它不是观察者模式的一部分，但是它可以在目标对象和观察者对象之间传递数据，在 AWT 事件模型中，所有的自定义事件类都是 java.util.EventObject 的子类。  

(2) LoginEventListener 充当抽象观察者，它声明了事件响应方法validateLogin()，用于处理事件，该方法也称为事件处理方法，validateLogin() 方法将一个 LoginEvent 类型的事件对象作为参数，用于传输与事件相关的数据，在其子类中实现该方法，实现具体的事件处理。  

(3) LoginBean 充当具体目标类，在这里我们没有定义抽象目标类，对观察者模式进行了一定的简化。在 LoginBean 中定义了抽象观察者 LoginEventListener 类型的对象 lel 和事件对象 LoginEvent，提供了注册方法 addLoginEventListener() 用于添加观察者，在 Java 事件处理中，通常使用的是一对一的观察者模式，而不是一对多的观察者模式，也就是说，一个观察目标中只定义一个观察者对象，而不是提供一个观察者对象的集合。在LoginBean中还定义了通知方法 fireLoginEvent()，该方法在 Java 事件处理模型中称为“点火方法”，在该方法内部实例化了一个事件对象 LoginEvent，将用户输入的信息传给观察者对象，并且调用了观察者对象的响应方法 validateLogin()。  

(4) LoginValidatorA 和 LoginValidatorB 充当具体观察者类，它们实现了在 LoginEventListener 接口中声明的抽象方法 validateLogin()，用于具体实现事件处理，该方法包含一个 LoginEvent 类型的参数，在 LoginValidatorA 和 LoginValidatorB 类中可以针对相同的事件提供不同的实现。