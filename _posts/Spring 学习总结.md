# Java中创建一个对象分两步： 
1.通过关键字new创建一个对象 
2.通过构造函数或setter函数为对象添加初始化参数 
当Spring出现后，对象的创建、成员变量的初始化、对象的销毁均由Spring完成。 
那么，要让Spring帮助我们创建对象，我们首先需要将要创建的对象的类型、初始化的值告诉Spring，然后Spring会在程序启动的时候根据我们的要求创建对象。我们通过配置文件来告诉Spring要创建哪些对象，并告诉Spring如何创建这些对象。

声明一个Bean

在Spring中，让Spring创建的对象叫做Bean，每一个bean都有成员函数和成员变量，如果bean的某些成员变量需要初始值，那么在bean的配置文件中声明即可，否则Spring会给bean的成员们赋上默认的初始值。 
如果已经有一个Person类，含有id和name两个属性：


```
class Person{
    private long id;
    private String name;

    public void setId(long id){
        this.id = id;
    }
    public long getId(){
        return this.id;
    }
    public void setName(String name){
        this.name = name;
    }
    public String getName(){
        return this.name;
    }
}
```


然后我们需要创建一个Spring的配置文件，在里面作如下定义：


```
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
    xsi:schemaLocation="http://www.springframework.org/schema/beans    
    http://www.springframework.org/schema/beans/spring-beans-4.5.xsd">  

    <bean id="person" class="com.njupt.Person"></bean>

</beans>
```


Spring的配置文件由beans标签开始，beans标签下的bean标签内即可声明一个bean。 
- id表示这个bean的名字，可以自定义，当我们需要使用这个对象时，通过这个id来获取对象。 
- class表示这个类的完整路径。Spring通过class属性找到这个类。 
到此为止，Person类的配置已经完成。当程序启动的时候，Spring会读取这个配置文件，根据class找到每个bean对应的类，并初始化它们。 
当我们需要使用一个对象时，通过如下操作即可获得：


```
ApplicationContext context = new ClassPathXmlApplicationContext("xml的路径");
Person person = (Person)context.getBean(“bean的名字”)
```

给Bean注入初始值

在上述示例中，Spring创建了一个Person对象，由于未指定初始值，因此Spring为person对象的属性赋上了默认值。下面我们来介绍如何让Spring为一个对象赋上指定的初始值。 
在Java中，给一个对象赋上初始值的方法有两种： 
1. 通过构造函数将初始值传递给对象 
2. 在创建完对象后通过set函数为对象赋上初始值

Spring也采用了这两种方式，分别叫做：构造器注入和属性注入。

构造器注入

1.注入基本类型的值 
我们首先为Person类添加一个构造函数：


```
class Person{
    private long id;
    private String name;

    public Person(String name,long id){
        this.name = name;
        this.id = id;
    }

    public void setId(long id){
        this.id = id;
    }
    public long getId(){
        return this.id;
    }
    public void setName(String name){
        this.name = name;
    }
    public String getName(){
        return this.name;
    }
}
```


然后在bean标签中添加属性：


```
<bean id="person" class="com.njupt.Person">
        <constructor-arg value="柴毛毛"/>
        <constructor-arg value="1"/>
</bean>
```


关于constructor-arg有如下注意点： 
- 属性constructor-arg的次序需要和构造函数中参数的顺序一致。 
- value表示属性值。 
- 属性constructor-arg中，无需指定属性名，只需填写属性值。 
- 属性constructor-arg中，所有的属性值均是String类型，Spring会在赋值的时候自动进行类型转换。

2.注入对象引用 
假设Person类中有一个Father类型的属性，并且添加参数为Father的构造函数：


```
class Person{
    private long id;
    private String name;
    private Father father;

    public Person(Father father){
        this.father = father;
    }
    ……省略所有setter、getter函数……
}
```


此时，构造函数的参数是一个引用类型的变量，我们可以做如下处理：


```
<!-- 首先创建Father类型的bean -->
    <bean id="father" class="com.njupt.Father">
        <constructor-arg value="柴毛毛的爸爸"/>
        <constructor-arg value="1"/>
    </bean>

    <!-- 通过ref引用father对象 -->
    <bean id="person" class="com.njupt.Person">
        <constructor-arg ref="father"/>
    </bean>
```


3.通过工厂创建bean 
如果一个类是工厂类，它没有构造函数，因此我们没有办法通过构造器来初始化这个对象。但工厂模式会提供给我们一个静态函数，用来获取工厂对象，那么在配置bean的时候做如下操作：

   
```
<bean id="person" class="com.njupt.Person" factory-method="getIntance"/>
```


对于工厂类，我们需要调用getInstance来获取工厂对象，因此在配置bean的时候，我们需要通过factory-method属性告诉Spring，获取这个工厂对象的函数是getInstance。

属性注入

通过上面我们了解到，Spring通过bean标签下的constructor-arg标签为构造函数注入参数值，接下来介绍Spring通过property标签为成员变量注入初始值。

1.注入基本类型的值 
为Person对象的name属性注入一个初始值“chaiMaoMao”： 
- name：属性名 
- value：属性值


```
<bean id="person" class="com.njupt.Person">
        <property name="name" value="chaiMaoMao"/>
    </bean>
```

注意：在编写配置文件的时候，Spring中value的全是String类型，但Spring会在运行的时候自动将其转换为相应的类型。

2.注入对象引用 
在bean中通过ref属性注入一个引用类型的变量。如：Person类中有一个Father类型的属性father，如果我们需要将一个father对象注入Person的father属性中，需要进行如下操作：


```
<!-- 引用name为father的bean注入给Person的father属性 -->
    <bean id="person" class="com.njupt.Person">
        <property name="father" ref="father"/>
    </bean>

    <!-- 定义name为father的bean -->
    <bean id="father" class="com.njupt.Father">
        <constructor-arg value="柴毛毛的爸爸"/>
        <constructor-arg value="1"/>
    </bean>

```


3.注入内部bean 
如果一个bean仅供某一个bean使用，那么可以将这个bean声明为内部bean。如：name为father的bean只允许注入给name为person的bean，那么就将father这个bean以放在person的内部，如下所示：


```
<bean id="person" class="com.njupt.Person">
        <property name="father">
            <bean class="com.njupt.Father" />
        </property>
    </bean>
```


注意： 
1. 内部bean没有名字，只适应于一次性注入，不能被其他bean引用。 
2. 注入内部bean不仅限于属性注入，也可用于构造器注入，如下所示：


```
<bean id="person" class="com.njupt.Person" init-method="createInstance" destory-method="destoryInstance">
        <constructor-arg>
            <bean class="com.njupt.Father" />
        </constructor-arg>  
    </bean>
```


### Bean的作用域

在Spring中，默认情况下bean都是单例。也就是说，当我们向Spring请求一个bean对象时，Spring总给我们返回同一个bean对象。 
**注意：**Spring 中所说的“单例”与Java中的单例稍有不同。Spring中的单例是指：在同一个ApplicationContext中相同名字的bean对象是同一个；而Java中的单例是指：整个JVM中单例的对象只有一个。 
当然，我们可以通过改变bean标签的scope参数来设置bean的作用域。常用的scope对应的值有： 
- singleton：在同一个Spring Context中，一个bean只有一个实例对象。(默认) 
- prototype：每次向Spring请求一个bean对象，Spring都会创建一个新的实例。

初始化和销毁Bean

如果需要在bean对象初始化之后或销毁之前做一些额外的操作的话，可以作如下配置： 
1. 首先需要在bean中定义函数，供Spring创建该类对象或销毁该对象的时候调用：


```
public void createInstance(){
        System.out.println("对象被创建啦！");
    }

    public void destoryInstance(){
        System.out.println("对象被销毁啦！");
    }
```

在XML中作如下配置： 
告诉Spring，这个bean在被创建的时候调用这个类中哪个函数，这个类被销毁的时候调用这个类中的哪个函数。

```
<bean id="person" class="com.njupt.Person" init-method="createInstance" destory-method="destoryInstance">
        <constructor-arg ref="father"/>
</bean>
```

如果所有的bean在初始化或销毁的时候都需要调用函数，那么可以在beans标签中设置一个全局的参数：default-init-method和default-destroy-method。这样，这个beans标签下的所有bean在创建或销毁时都会调用createInstance()和destoryInstance()。


```
<beans xmlns="http://www.springframework.org/schema/beans"  
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
    xsi:schemaLocation="http://www.springframework.org/schema/beans    
    http://www.springframework.org/schema/beans/spring-beans-4.5.xsd"

    default-init-method="createInstance" 
    default-destory-method="destoryInstance">  

</beans>
```


装配集合

到此为止，不管是装配基本类型还是装配对象的引用，都是在装配单个属性，那么该如果装配一个集合呢？

集合元素	用途
list	用于装配单值集合，允许重复
set	用于装配单值集合，不允许重复
map	用于装配键值对集合，key和value可以为任意类型
props	用于装配键值对集合，key和value只能为String类型
- 对于单个值的集合，如数组、List、Set，均可以用list标签来装配，list标签和set标签的不同之处仅仅是前者允许重复，而后者不允许重复。 
- 对于键值对集合，可以使用set标签或props标签，两者的区别是：前者的key和value可以是任何类型，而后者的key和value只能是String类型。

1.装配List：


```
<bean id="person" class="com.njupt.Person">
        <property name="list">
            <list>
                <ref bean="a"/>
                <ref bean="b"/>
                <ref bean="c"/>
            </list>
        </property>
    </bean>
```


2.装配Map：


```
<bean id="person" class="com.njupt.Person">
        <property name="map">
            <map>
                <entry key="key_a" value-ref="a"/>
                <entry key="key_b" value-ref="b"/>
                <entry key="key_c" value-ref="c"/>
            </map>
        </property>
    </bean>
```


3.装配props：


```
<bean id="person" class="com.njupt.Person">
        <property name="map">
            <props>
                <prop key="key_a">value_a</prop>
                <prop key="key_b">value_b</prop>
                <prop key="key_c">value_c</prop>
            </props>
        </property>
    </bean>
```


SpEL表达式

SpEL＝Spring Express Language 
这种表达式用在Spring的配置文件中，可以直接获取某个bean的某一个属性值或获得一个函数的执行结果，相当屌！ 
SpEL表达式不难，下面举几个例子相信聪明的你就能理解了。

引用一个bean的属性值

```
<bean id="person" class="com.njupt.Person">
        <property name="name" value="#{father.name}" />
    </bean>
```


引用一个bean的函数的运行结果
    <bean id="person" class="com.njupt.Person">
        <property name="name" value="#{father.getName()}" />
    </bean>
1
2
3
引用Java中某个静态函数的运行结果
    <bean id="person" class="com.njupt.Person">
        <property name="name" value="#{T(java.lang.Math).random()}" />
    </bean>
1
2
3
数值运算
    <bean id="person" class="com.njupt.Person">
        <property name="id" value="#{count.id + 100}" />
    </bean>
1
2
3
字符连接
    <bean id="person" class="com.njupt.Person">
        <property name="name" value="#{father.name +' '+ mother.name}" />
    </bean>
1
2
3
比较运算
    <bean id="person" class="com.njupt.Person">
        <property name="equal" value="#{father.name == mother.name}" />
    </bean>
1
2
3
访问集合成员
    <bean id="person" class="com.njupt.Person">
        <property name="name" value="#{list['chaiMaoMao']}" />
    </bean>
1
2
3
    <bean id="person" class="com.njupt.Person">
        <property name="name" value="#{list[2]}" />
    </bean>
1
2
3
查询集合成员
    <bean id="person" class="com.njupt.Person">
        <property name="name" value="#{list.?[age gt 20]}" />
    </bean>
1
2
3
投影集合
    <bean id="person" class="com.njupt.Person">
        <property name="names" value="#{list.![name]}" />
    </bean>