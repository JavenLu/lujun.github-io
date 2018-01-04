---
title: 设计模式--简记
---

# {{ page.title }}

## 目录

+ [装饰模式](#partI)

----------------------------------

## 装饰模式

    基本使用场景：对某个对象进行功能扩展，且功能按新-老排序。
    代码片段：
<pre><code>
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
    </code></pre>

    结果：

    lala: 大裤衩儿
    lala: 大体恤
    lala: 装扮者:Javen
{{ page.date|date_to_string }}