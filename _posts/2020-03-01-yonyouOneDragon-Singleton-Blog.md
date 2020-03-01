# 基于NC的单例模式讲解及说明
**本次的说明全部是基于NCV6的系列进行的**

##1、饿汉式
###静态变量

我们来看看NC的一个源码：（nc.vo.arappub.calculator.data.RelationItemForCal_Credit）

    public class RelationItemForCal_Credit extends RelationItemForCal
      {
    private static RelationItemForCal_Credit instance = new RelationItemForCal_Credit();
     
    public static RelationItemForCal_Credit getInstance() {
      return instance;
      }
    public RelationItemForCal_Credit()
      {...}
      }

这个的写法是使用了对应的静态变量，但是有两个地方要注意：

1、这个instance的对象没有用到final作为修饰符。加了final的静态变量必须直接赋值或者在静态代码块中赋值，这块其实不影响，因为最后还是给instance赋值了

2、构造方法没有设置成private。这块我的理解是，作者为了省事通过构造方法去set属性，此处不予深究了。

####缺点与优点
#####缺点：
1、容易造成内存浪费，饿汉式的单例模式是无论是否调用到都会在类加载的时候创建对象
#####优点：
1、使用类加载机制（classsloader）解决了多线程情况下对象可能创建不成功的问题

###静态代码块
待定，暂无从NC找到的代码块

##2、懒汉式
###线程安全，同步代码块
使用synchronized修饰方法，对方法进行加锁，属于线程锁的一种，暂无发现NC的代码所在。

###线程不安全，同步代码块
我们来看一段portal的代码：nc.uap.lfw.core.formular.LfwFormulaParser

    public static FormulaParse getInstance() {
      if (instance == null) {
    synchronized (LfwFormulaParser.class) {
      if (instance == null) {
     instance = new FormulaParse("printtemplate");
     IFormulaProvider fp = (IFormulaProvider)ServiceLocator.getService(IFormulaProvider.class);
     FormulaVO[] fvos = fp.getFormulas();
这个写法看似是解决了前面的代码问题，但是出现了一个新的问题，这段代码其实是线程不安全的，我理解这段代码的原意，应该是说我每次去做模板更新的时候，他都是去load同一个模板并按照特定的条件查询相应的模板属性，如果考虑多线程的情况下，A线程与B线程可能同时进入这段代码的第一个if语句里面，这种情况下，A线程还未创建完实例后B线程就又可能创建了新的实例，造成创建了多个实例的情况了。