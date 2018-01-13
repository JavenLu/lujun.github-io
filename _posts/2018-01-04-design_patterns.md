---
title: 设计模式--简记
---

# {{ page.title }}

## 目录

+ [装饰模式](#partI)
+ [简单工厂模式](#partII)
+ [策略模式](#partIII)

----------------------------------

## 装饰模式

    基本使用场景：对某个对象进行功能扩展，且功能按新-老排序。
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
    {{ page.date|date_to_string }}



