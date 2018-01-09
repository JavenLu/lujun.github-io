---
title: 设计模式--简记
---

# {{ page.title }}

## 目录

+ [装饰模式](#partI)
+ [简单工厂模式](#partII)

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
    {{ page.date|date_to_string }}



