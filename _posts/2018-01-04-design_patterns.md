---
title: 设计模式--简记
---

# {{ page.title }}

## 目录

+ [装饰模式](#装饰模式)
+ [简单工厂模式](#简单工厂模式)
+ [策略模式](#策略模式)
+ [代理模式](#代理模式)
+ [工厂模式](#工厂模式)
+ [建造者模式](#建造者模式)
+ [适配器模式](#适配器模式)
+ [外观模式](#外观模式)

----------------------------------

## 装饰模式

    基本使用场景：对某个对象进行功能叠加扩展。
    代码片段：

    public class Person {
   
    public Person() {
    }

    private String name;

    public Person(String name) {
        this.name = name;
    }

    public void show() {
        Log.e("lala", "装扮者:" + name);
    }
    
    }


    public class CostumeDecorator extends Person {

    public Person component;

    public void decorate(Person component) {
        this.component = component;
    }

    @Override
    public void show() {

        if (component != null) {
            component.show();
        }
    }


    }


    public class TShirt extends CostumeDecorator {
    
    @Override
    public void show() {
        Log.e("lala", "大体恤");
        super.show();
    }
    }

    public class BigTrouser extends CostumeDecorator {
    @Override
    public void show() {
        Log.e("lala", "大裤衩儿");
        super.show();
    }
    }

    public class MainActivity extends Activity {
    
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Person person = new Person("Javen");

        TShirt tshirt = new TShirt();
        tshirt.decorate(person);

        BigTrouser bigTrouser = new BigTrouser();
        bigTrouser.decorate(tshirt);

        bigTrouser.show();

    }
    }
    

    结果：

    lala: 大裤衩儿
    lala: 大体恤
    lala: 装扮者:Javen
    
    
    ----------------------------------


## 简单工厂模式
 
    基本使用场景：使用时通过类型判断，以多态方式返回同一个父类，调用相同的方法，完成不同的逻辑。这种写法可以
    非常好的解耦、封装代码。
    
    代码片段：
    
    public abstract class Operation {
    private int result = 0;
    private int mum1, num2;
    
    public int getNum1() {
    return mum1;
    }
    
    public void setMum1(int mum1) {
    this.mum1 = mum1;
    }
    
    public int getNum2() {
    return num2;
    }
    
    public void setNum2(int num2) {
    this.num2 = num2;
    }
    
    public abstract int getResult();
    }
    
    
    public class AddOperation extends Operation {
    @Override
    public int getResult() {
    return getNum1() + getNum2();
    }
    
    }
    
    public class SubOperation extends Operation {
    @Override
    public int getResult() {
    return getNum1() - getNum2();
    }
    
    }
    
    public class MulOperation extends Operation {
    @Override
    public int getResult() {
    return getNum1() * getNum2();
    }
    
    }
    
    public class DivOperation extends Operation {
    @Override
    public int getResult() {
    return getNum1() / getNum2();
    }
    
    }
    
    public interface OperationFactoryInterface {
    Operation getOperationInstance(String operator);
    }
    
    
    public class OperationFactory implements OperationFactoryInterface {
    private static OperationFactory operationFactory;
    
    public static OperationFactory getOperationFactoryInstance() {
    if (operationFactory == null) {
    operationFactory = new OperationFactory();
    }
    return operationFactory;
    }
    
    @Override
    public Operation getOperationInstance(String operator) {
    switch (operator) {
    case "+":
    return new AddOperation();
    case "-":
    return new SubOperation();
    case "*":
    return new MulOperation();
    case "/":
    return new DivOperation();
    default:
    return null;
    }
    }
    }
    
    
    public class MainActivity extends AppCompatActivity {
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    Operation operation = OperationFactory.getOperationFactoryInstance().getOperationInstance("*");
    operation.setMum1(2);
    operation.setNum2(4);
    Log.e("result", operation.getResult() + "");
    
    }
    }
    }
    
    
    ----------------------------------

## 策略模式

    基本使用场景：使用时通过类型判断，将不同策略封装进同一个类中，调用时只需要调用此类的包装方法便可执行相关策略。与工厂方法
    比好处是前者需要让客户端认识两个类，而后者只需要让其认识一个就可以了。
    
    代码片段：
  
    public abstract class SuperStrategy {
    private int result = 0;
    private int mum1, num2;
    
    public int getNum1() {
    return mum1;
    }
    
    public void setMum1(int mum1) {
    this.mum1 = mum1;
    }
    
    public int getNum2() {
    return num2;
    }
    
    public void setNum2(int num2) {
    this.num2 = num2;
    }
    
    public abstract int getResult(){
    }
    }
    
    public class AddStrategy extends SuperStrategy {
    @Override
    public int getResult() {
    return getNum1() + getNum2();
    }
    }
    
    public class SubStrategy extends SuperStrategy {
    @Override
    public int getResult() {
    return getNum1() - getNum2();
    }
    }
    
    public class MulStrategy extends SuperStrategy {
    @Override
    public int getResult() {
    return getNum1() * getNum2();
    }
    }
    
    public class DivStrategy extends SuperStrategy {
    @Override
    public int getResult() {
    return getNum1() / getNum2();
    }
    }
    
    public class StrategyContext {
    private SuperStrategy superStrategy;
    
    public StrategyContext(SuperStrategy superStrategy) {
    this.superStrategy = superStrategy;
    }
    
    public int getResult() {
    return superStrategy.getResult();
    }
    
    public void setNum(int num1, int num2) {
    superStrategy.setMum1(num1);
    superStrategy.setNum2(num2);
    }
    
    }
    
    public class MainActivity extends AppCompatActivity {
    private StrategyContext superContext;
    
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    
    getStrategyContext("*");
    superContext.setNum(2, 4);
    superContext.getResult();
    }
    
    private void getStrategyContext(String operator) {
    switch (operator) {
    case "+":
    superContext = new StrategyContext(new AddStrategy());
    break;
    
    case "-":
    superContext = new StrategyContext(new SubStrategy());
    break;
    
    case "*":
    superContext = new StrategyContext(new MulStrategy());
    break;
    
    case "/":
    superContext = new StrategyContext(new DivStrategy());
    break;
    
    }
    
    }
    }
    
    但是策略模式有个缺点，客户端中出现switch case的代码，随着策略的增加，耦合度会更大，通过策略模式+工厂模式结合可以把语句封装到StrategyContext类中
    
    代码片段：
    
    public class StrategyContext {
    private SuperStrategy superStrategy;
    
    public StrategyContext(String operator) {
    
    switch (operator) {
    case "+":
    superStrategy = new AddStrategy();
    break;
    
    case "-":
    superStrategy = new SubStrategy();
    break;
    
    case "*":
    superStrategy = new MulStrategy();
    break;
    
    case "/":
    superStrategy = new DivStrategy();
    break;
    
    }
    }
    
    public int getResult() {
    return superStrategy.getResult();
    }
    
    public void setNum(int num1, int num2) {
    superStrategy.setMum1(num1);
    superStrategy.setNum2(num2);
    }
    
    }

    public class MainActivity extends AppCompatActivity {
    private StrategyContext superContext;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    superContext = new StrategyContext("*");
    superContext.setNum(2, 4);
    superContext.getResult();
    }

    }
    
    
    ----------------------------------

## 代理模式

    代理的概念：代理其实就是在访问对象时引入一定程度的间接性，因为这种间接性，可以附加多种用途。
    通俗理解：你在某些时候，直接去干一些事情，并不能起到很好的效果，反而适得其反。这时就需要一个中间人去帮你完成这些工作。
    好处：代理模式最直接的好处就是代码隔离，降低耦合，高度可维护性，封装性好。
    
    代理的应用：
    1.远程代理，也就是为一个对象在不同的地址空间提供局部代表，这样可以隐藏一个对象存在于不同地址空间的事实。
    2.虚拟代理，是根据需要创建开销很大的对象，通过它来存放实例化需要很长时间的真实对象。
    3.安全代理，用来控制真是对象访问时的权限。
    4.智能指引，当调用真是的对象时，代理处理另外一些事。
    5.个人已经讲这个模式应用在了RecyclerView 的Adapter中。
    
    public abstract class ProxyInterface {
    abstract void sendPhone();
    
    abstract void sendComputer();
    
    abstract void sendWatch();
    }
    
    public class ReceivesPerson {
    public String name;
    
    public String getName() {
    return name;
    }
    
    public void setName(String name) {
    this.name = name;
    }
    }
    
    
    public class RealPerson extends ProxyInterface {
    ReceivesPerson receivesPerson;
    
    public RealPerson(ReceivesPerson receivesPerson) {
    this.receivesPerson = receivesPerson;
    }
    
    @Override
    void sendPhone() {
    System.out.println("送手机给：" + receivesPerson.getName());
    }
    
    @Override
    void sendComputer() {
    System.out.println("送电脑给：" + receivesPerson.getName());
    }
    
    @Override
    void sendWatch() {
    System.out.println("送手表给：" + receivesPerson.getName());
    }
    
    
    }
    
    public class Proxy extends ProxyInterface {
    private RealPerson realPerson;
    
    public Proxy(ReceivesPerson receivesPerson) {
    if (realPerson == null) {
    realPerson = new RealPerson(receivesPerson);
    }
    }
    
    @Override
    void sendPhone() {
    realPerson.sendPhone();
    }
    
    @Override
    void sendComputer() {
    realPerson.sendComputer();
    }
    
    @Override
    void sendWatch() {
    realPerson.sendWatch();
    }
    }
    
    
    public class MainActivity extends AppCompatActivity {
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    ReceivesPerson receivesPerson = new ReceivesPerson();
    receivesPerson.setName("A Beautiful Girl");
    
    Proxy proxy = new Proxy(receivesPerson);
    proxy.sendPhone();
    proxy.sendComputer();
    proxy.sendWatch();
    }
    }
    
## 工厂模式

    基本使用场景：工厂模式与简单工厂模式的区别，简单来讲前者比后者更加抽象，客户端在使用时不用再通过条件进行判断，而是直接创造某一类的工厂，生成具体的某一类的对象，进行操作。好处是避免了switch case，需要什么类型就创建或添加类型工厂就可以了。克服了开放-封闭原则的缺点--对修改的封闭。页使其不做大的改动就可以实现。 同时降低了耦合性。缺点还是没有避免修改客户端代码。
    
    代码片段：
    
    public abstract class WarPlane {
    abstract void bomb();
    
    abstract void fight();
    
    abstract void cruise();
    }
    
    
    public class ChinaPlane extends WarPlane {
    @Override
    void bomb() {
    System.out.println("中国轰炸");
    }
    
    @Override
    void fight() {
    System.out.println("中国格斗");
    }
    
    @Override
    void cruise() {
    System.out.println("中国巡航");
    }
    }
    
    
    public class RussiaPlane extends WarPlane {
    @Override
    void bomb() {
    System.out.println("俄国轰炸");
    }
    
    @Override
    void fight() {
    System.out.println("俄国格斗");
    }
    
    @Override
    void cruise() {
    System.out.println("俄国巡航");
    }
    }
    
    public interface IFactory {
    WarPlane createWarPlane();
    }
    
    
    public class ChinaWarPlaneFactory implements IFactory {
    @Override
    public WarPlane createWarPlane() {
    return new ChinaPlane();
    }
    }
    
    
    public class RussiaWarPlaneFactory implements IFactory {
    @Override
    public WarPlane createWarPlane() {
    return new RussiaPlane();
    }
    }
    
    public class MainActivity extends AppCompatActivity {
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    IFactory iFactory = new ChinaWarPlaneFactory();
    WarPlane warPlane = iFactory.createWarPlane();
    warPlane.bomb();
    warPlane.fight();
    warPlane.cruise();
    }
    }
    
    
## 建造者模式

    概念：将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示。
    
    代码片段：
    
    构造对象
    public class CustomDialog extends AlertDialog {
    
    private String name;
    private String sex;
    private String age;
    
    protected CustomDialog(@NonNull Context context) {
    super(context);
    }
    
    public void setName(String name) {
    this.name = name;
    }
    
    public String getName() {
    return name;
    }
    
    public String getSex() {
    return sex;
    }
    
    public void setSex(String sex) {
    this.sex = sex;
    }
    
    public String getAge() {
    return age;
    }
    
    public void setAge(String age) {
    this.age = age;
    }
    
    public void showInfo() {
    System.out.println(getName());
    System.out.println(getSex());
    System.out.println(getAge());
    }
    }
    
    
    构造器抽象类
    public abstract class Builder {
    
    public abstract void setName();
    
    public abstract void setSex();
    
    public abstract void setAge();
    
    public abstract CustomDialog getDialogInstance();
    
    }
    
    
    构造器A
    public class CustomBuilderA extends Builder {
    private CustomDialog customDialog;
    
    public CustomBuilderA(Context context) {
    customDialog = new CustomDialog(context);
    }
    
    
    @Override
    public void setName() {
    customDialog.setName("李蕾");
    }
    
    @Override
    public void setSex() {
    customDialog.setSex("男");
    }
    
    @Override
    public void setAge() {
    customDialog.setAge("21");
    }
    
    @Override
    public CustomDialog getDialogInstance() {
    return customDialog;
    }
    }
    
    
    构造器B
    public class CustomBuilderB extends Builder {
    private CustomDialog customDialog;
    
    public CustomBuilderB(Context context) {
    customDialog = new CustomDialog(context);
    }
    
    
    @Override
    public void setName() {
    customDialog.setName("韩梅梅");
    }
    
    @Override
    public void setSex() {
    customDialog.setSex("女");
    }
    
    @Override
    public void setAge() {
    customDialog.setAge("18");
    }
    
    @Override
    public CustomDialog getDialogInstance() {
    return customDialog;
    }
    }
    
    
    导演者
    public class Director {
    
    public void Construct(Builder builder) {
    builder.setName();
    builder.setSex();
    builder.setAge();
    }
    }
    
    
    调用类
    public class BuilderModeActivity extends AppCompatActivity {
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    Builder builderA = new CustomBuilderA(this);
    Builder builderB = new CustomBuilderB(this);
    
    Director director = new Director();
    director.Construct(builderA);
    
    CustomDialog customDialogA = builderA.getDialogInstance();
    customDialogA.showInfo();
    
    director.Construct(builderB);
    CustomDialog customDialogB = builderB.getDialogInstance();
    customDialogB.showInfo();
    
    
    }
    }
    
    结果：
    李蕾
    男
    21
    韩梅梅
    女
    18
    

## 适配器模式


    概念：将一个类的接口转换成客户希望的另外一个接口。Adapter 模式使得原本由于接口不兼容而不能一起工作的那些类可以一起工作。

    基本使用场景：系统的数据和行为都正确，但接口不符时，我们应该考虑用适配器，目的是使控制范围之外的一个原有对象与某个接口匹配。适配器模式主要应用于希望复用一些现存的类，但是接口又与复用环境要求不一致的情况。

    种类：适配器模式有两种，类适配器和对象适配器，由于类适配器模式通过多继承对一个接口与另一个接口进行匹配，但java不支持多继承，所以主要研究对象适配器。

    代码片段：
    
    public abstract class Player {
    public String name;
    
    public Player(String name) {
    this.name = name;
    }
    
    public abstract void attack();
    
    public abstract void defense();
    }
    
    
    
    public class Forwards extends Player {
    
    public Forwards(String name) {
    super(name);
    }
    
    @Override
    public void attack() {
    System.out.println("前锋 " + name + " 进攻");
    }
    
    @Override
    public void defense() {
    System.out.println("前锋 " + name + " 防守");
    }
    }
    
    
    public class Center extends Player {
    
    public Center(String name) {
    super(name);
    }
    
    @Override
    public void attack() {
    System.out.println("中锋 " + name + " 进攻");
    }
    
    @Override
    public void defense() {
    System.out.println("中锋 " + name + " 防守");
    }
    }
    
    
    public class Guards extends Player {
    
    public Guards(String name) {
    super(name);
    }
    
    @Override
    public void attack() {
    System.out.println("后卫 " + name + " 进攻");
    }
    
    @Override
    public void defense() {
    System.out.println("后卫 " + name + " 防守");
    }
    }
    
    
    public class ForeignCenter {
    private String name;
    
    public String getName() {
    return name;
    }
    
    public void setName(String name) {
    this.name = name;
    }
    
    public void 进攻() {
    System.out.println("外籍中锋 " + name + " 进攻");
    }
    
    public void 防守() {
    System.out.println("外籍中锋 " + name + " 防守");
    }
    
    }
    
    
    public class Translator extends Player {
    
    private ForeignCenter foreignCenter;
    
    public Translator(String name) {
    super(name);
    foreignCenter = new ForeignCenter();
    foreignCenter.setName(name);
    }
    
    @Override
    public void attack() {
    foreignCenter.进攻();
    }
    
    @Override
    public void defense() {
    foreignCenter.防守();
    }
    }
    
    
    public class AdapterModeActivity extends AppCompatActivity {
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    Player player = new Center("奥尼尔");
    player.attack();
    player.defense();
    
    Player player1 = new Forwards("诺维茨基");
    player1.attack();
    player1.defense();
    
    Player player2 = new Guards("林书豪");
    player2.attack();
    player2.defense();
    
    Player foreignCenter = new Translator("姚明");
    foreignCenter.attack();
    foreignCenter.defense();
    }
    }
    
    
    结果：
    
    中锋 奥尼尔 进攻
    中锋 奥尼尔 防守
    前锋 诺维茨基 进攻
    前锋 诺维茨基 防守
    后卫 林书豪 进攻
    后卫 林书豪 防守
    外籍中锋 姚明 进攻
    外籍中锋 姚明 防守
    
    
## 外观模式


    概念：为子系统中的一组接口提供一个一致的界面，此模式定义了一个高层接口，这个接口使得这子系统更加容易使用。
    白话：客户端无需对应多个子系统，关注多个变化，只需要通过外观类统一管理子系统就可以，客户端只与外观类进行互动。
    基本使用场景：1.程序架构时，层与层之间建立外观，使耦合大大降低。 2.减少各子系统的依赖。3.在维护一个大型系统时，可能这个系统
    已经非常难以维护和扩展了，构建一个新系统，为新系统开发一个外观类，来提供设计粗糙或高度复杂的遗留代码的比较清晰的接口，让新系统
    与外观类对象交互，外观类与遗留代码交互所有复杂的工作。
    
    代码片段：
    
    
    public abstract class BaseSystem {
    public String name;
    
    public BaseSystem(String name) {
    this.name = name;
    }
    
    public abstract void executeTask();
    
    }
    
    
    
    public class SystemOne extends BaseSystem {
    
    public SystemOne(String name) {
    super(name);
    }
    
    @Override
    public void executeTask() {
    System.out.println("SystemOne " + name);
    }
    
    
    }
    
    
    
    public class SystemTwo extends BaseSystem {
    
    public SystemTwo(String name) {
    super(name);
    }
    
    @Override
    public void executeTask() {
    System.out.println("SystemTwo " + name);
    }
    
    
    }
    
    
    public class Facade {
    private SystemOne systemOne;
    private SystemTwo systemTwo;
    
    private static Facade facade;
    
    public Facade() {
    systemOne = new SystemOne("李蕾");
    systemTwo = new SystemTwo("韩梅梅");
    }
    
    public static Facade getInstants() {
    if (facade == null) {
    facade = new Facade();
    }
    
    return facade;
    }
    
    
    public void executeTaskA() {
    systemOne.executeTask();
    }
    
    public void executeTaskB() {
    systemTwo.executeTask();
    }
    
    
    }
    
    
    public class Client extends AppCompatActivity {
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    Facade.getInstants().executeTaskA();
    Facade.getInstants().executeTaskB();
    }
    }
    
    
    
----------------------------------
    {{ page.date|date_to_string }}



