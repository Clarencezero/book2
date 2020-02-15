# Spring的IoC容器之BeanFactory

Spring提供 了两种容器类型：BeanFactory和ApplicationContext。

- BeanFactory。基础类型IoC容器，提供完整的IoC服务支持。如果没有特殊指定，默认采用延 迟初始化策略dazy-load）。只有当客户端对象需要访问容器中的某个受管对象的吋候，才对 该受管对象进行初始化以及依赖注入操作。所以，相对来说，容器启动初期速度较快，所需 要的资源有限。对于资源有限，并且功能要求不是很严格的场景，BeanFactory是比较合适的 IoC容器选择。
- ApplicationContext。Applicationcontext在BeanFactory的基础上构建，是相对比较高 级的容器实现，除了拥有BeanFactory的所有支持，ApplicationContext还提供了其他高级特性，比如事件发布、国际化信息支持等，这些会在后面详述。Applicationcontext所管理 的对象，在该类型容器启动之后，默认全部初始化并绑定完成。所以，相对于BeanFactory来 说，Applicationcontext要求更多的系统资源，同时，因为在启动时就完成所有初始化，容器启动时间较之BeanFactory也会长一些。在那些系统资源充足，并且要求更多功能的场景中， Applicationcontext类型的容器是比较合适的选择。

![1551927332393](https://cdn.jsdelivr.net/gh/Clarencezero/poi/1551927332393.png)

作为Spring提供的基本的IoC容器,beanFactory可以完成作为IoC Service Provider的所有职责, 包括业务对象的注册和对象间依赖关系的绑定。

BeanFactory只是一个接口, DefaultListableBeanFactory就是这么一个比较通用的BeanFactory实现类。它还实现了BeanDefinitionRegistry接口,该接口才是在BeanFactory的实现中担当Bean注册管理的角色。BeanDefinitionRegistry 接口定义抽象了Bean的注册逻辑。

![1551929173331](https://cdn.jsdelivr.net/gh/Clarencezero/poi/1551929173331.png)

打个比方说，BeanDefin让ionRegistry就像图书馆的书架，所有的书是放在书架上的。虽然你 还书或者借书都是跟图书馆（也就是BeanFactory,或许BookFactory可能更好些）打交道，但书架才是图书馆存放各类图书的地方。所以，书架相对于图书馆来说，就是它的“BookDefinitionRegistry”。

第一个受管的对象,在容器中都会有一个BeanDefinition的实例(instance)与之对应,该BeanDefinition的实例负责保存对象的所有必要信息,包括其对应的对象的class类型、是否是抽象类、构造方法参数心脏其他属性等。当客户端向BeanFactory请求相应对象的时候, BeanFactory会通过这些信息为客户端返回一个完备可用的对象实例。RootBeanDefinition和ChildBeanDefinition是BeanDefinition的两个主要的实现类。

![1551929874194](https://cdn.jsdelivr.net/gh/Clarencezero/poi/1551929874194.png)



### 4.2.2 外部配置文件方式

SPring的IoC容器支持两种配置文件格式: Properties文件格式和XML文件格式。

通常情况下,需要根据不同的外部配置文件格式,给出相应的BeanDefinitionReader实现类, 由BeanDefinitionReader的相应实现类负责将相应的配置文件内容读取并映射到BeanDefinition,然后将映射后的BeanDefinition注册到一个BeanDefinitionRegistry,之后,BeanDefinitionRegistry即完成Bean的注册和加载。当然, 大部分工作,包括解析文件格式、装配BeanDefinition之类的工作,都是由BeanDefinitionReader的相应实现类来做的,BeanDefinitionRegistry只不过负责保管而已。

#### 4.2.2.1 XML配置格式的加载

``` java
public class LoadingXmlTest {
    public static void main(String[] args) {
        DefaultListableBeanFactory beanRegistry = new DefaultListableBeanFactory();
        BeanFactory container = (BeanFactory)bindViaXMLFile(beanRegistry);
        Sunday sunday = (Sunday) container.getBean("sunday");
        System.out.println(sunday.getAge());
        System.out.println(sunday.getUserName());
    }

    private static BeanFactory bindViaXMLFile(DefaultListableBeanFactory beanRegistry) {
        XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(beanRegistry);
        reader.loadBeanDefinitions("spring.xml");
        return (BeanFactory)beanRegistry;
    }
}	
```

### 4.3 BeanFactory的XML之旅

#### 4.3.1 <beans>和<bean>

所有注册到容器的业务对象,在Spring中称之为Bean。所以, 第一个对象在XML中的映射也自然而然地对应一个叫做<Bean>的元素。把这些<bean>的元素组织起来的, 就叫做<beans>。

＜beans＞作为所有＜bean＞的"统帅"，它拥有相应的属性(attribute)对所辖的＜bean＞进行统一 的默认行为设置，包括如下几个。

- default-lazy-init。其值可以指定为true或者false,默认值为false。用来标志是否对所有的＜bean＞进行延迟初始化。
- default-autowire。可以取值为no、byName、 byType、 constructor以及autodetect。默 认值为no, 如果使用自动绑定的话，用来标志全体bean使用哪一种默认绑定方式。
- default-dependency-check。可以取值 none、objects、simple 以及 all,默认值为 none, 即不做依赖检查。
- default-init-method。如果所管辖的＜bean＞按照某种规则，都有同样名称的初始化方法的话，可以在这里统一指定这个初始化方法名，而不用在每一个＜bean＞上都重复单独指定。
- default-destroy-method。与default-init-method相对应，如果所管辖的bean有按照某种规则使用了相同名称的对象销毁方法，可以通过这个属性统一指定。
- <import> 可以根据模块功能或者层次关系,将配置信息分门别类地放到多个配置文件中。这个功能价值不大, 因为容器可以同时加载多个配置,没有必要非通过一个配置文件来加载所有配置。

##### 4.3.3.1 构造方法注入的XML之道

```xml
<bean id="test" class="com.test.Test">
	<constructor-arg>
		<ref bean="newBean" />
	</constructor-arg>
</bean>

<!-- 也可以表示成这个样子-->
<constructor-arg ref bean="newBean" />
```

因为某些原因, 无法明确配置项与对象的构造方法参数列表的一一对应关系, 就需要请<constructor-arg>的type或者index属性。

```java
public Mock(String str) {}
public Mock(int int) {}
```

```xml
<bean id = "mock" class = "com.mock">
	<constructor-arg type="int">
		<value>123</value>
	</constructor-arg>		
</bean>

// 也可以使用 index="0"的方式。标从0开始
```

##### 4.3.3.2 Setter方法注入的XML之道

<property name="hello" class="">

注: <constructor-arg> 和 <property>可以共存。如果两者对同一个对象的话, 只有<property>有用

#####  针对<property>和<constructor-arg>等通用的内嵌元素

1. ##### <value>。不但可以指定String类型的数据,而且还可以指定其他Java语言中的原始类型以及它们的包装类型。int、Integer等。它是最底层元素。

2. <ref>。引用容器中其他的对象实例。可能通过ref的local、parent和bean属性来指定引用的对象的beanName是什么。

   ```xml
   <ref local="" />
   ```

   local、parent和bean的区别在于:

   - local只能指定与当前配置的对象在同一个配置文件的对象定义的名称（可以获得XML解析器 的id约束验证支持）；
   - parent则只能指定位于当前容器的父容器中定义的对象引用；

   注意 BeanFactory以分层次（通过实现HierarchicalBeanFactory接口 ）,容器A在初始化的时候，可以首先加载容器B中的所有对象定义，然后再加载自身的对象定义，这样，容器 B就成为了容器A的父容器，容器A可以引用容器B中的所有对象定义：
   
   ```java
   BeanFactory parentcontainer = new XmlBeanFactory(new ClassPathResource("父容器酉己置文件路径"))；
   BeanFactory childcontainer = new XmlBeanFactory(new ClassPathResource("子容器百己置文件路径"),parentContainer);
   ```
   
   
   childcontainer中定义的对象，如果通过parents指定依赖，则只能引用parentContainer中 的对象定义。

   - bean 则基本通吃,所以,通常情况下,直接使用bean来指定对象引用就可以了。

3. <idref>。如果要为当前对象注入所依赖的对象的名称,而不是引用,那么通常情况下,可以使用<value>来达到这个目的。

   ```xml
   <property name="newListenterBean">
   	<value>newList</value>
   </property>	
   ```

   但是对于这种场合,使用idref都是最合适的,因为使用idref,容器在解析配置的时候就可以帮你检查这个beanName到底是否存在,而不用等到运行时才发现这个beanName对应的对象实例不存在。idref注入的是目标bean的id而不是目标bean的实例。而ref完全不同,ref元素是将目标bean定义的实例注入到属性或构造函数中。

4. 内部<bean>。有点像内部类, 所依赖的对象只有当前一个对象引用,或者某个对象定义我们不想其他对象通过<ref>引用到它。

5. <list>。<list>对应注入对象类型为java.util.List及其子类或者数组类型的依赖对象。

6. <set>。无序,对应注入类型为java.util.Set或者子类的依赖对象。

7. ``<map>``。映射可以通过指定的键(key)来获取相应的值。注入java.util.Map或者其子类类型的依赖对象。它可以内嵌任意多个<entry>, 每一个<entry>都需要为其指定一个键和一个值。

   - <entry>的键, key或者key-ref来指定键。
   - <entry>的值, 除了<key>是用来指定键的,其他元素可以任意使用,来指定entry对应的值。也可以直接使用value、value-ref这两个属性来指定。

   ![1552011681218](https://cdn.jsdelivr.net/gh/Clarencezero/poi/1552011681218.png)

8. <props>。是简化后的<map>。该元素对应配置类型为java.util.Properties的对象依赖。

   ```xml
   <property name="mylover">
       <props>
       	<prop key="name">张汶沣</prop>
           ...
       </props>
   </property>
   ```

9. <null/>。这是最简单的一个元素。对于String来说,如果使用<value></value>注入的话,得到的结果为"", 而不是null。所以, 如果需要为这个string对应的值注入null的话,请使用<null/>。

##### 4.3.3.3 depends-on

对象A依赖于对象B,需要在实例化对象A之前首先实例化对象B。大部分情况下,是那些拥有静态代码块初始化代码或者数据库驱动注册之类的场景。

如果A拥有多个类似的非显示依赖关系,那么, 你可以在A的depends-on中通过逗号分割各个beanName。

##### 4.3.3.4 autowire

可以指定当前bean定义采用某种类型的自动绑定模式。这样,你就无需手工明确指定该bean定义相关的依赖关系,从而也可以免去一些手工输入的工作量。

1. no。默认的自动绑定模式, 也就是不采用任何形式的自动绑定,完全依赖手工明确配置各个bean之间的依赖关系。
2. byName。按照类中声明的实例变量的名称,与XML配置文件中声明的bean定义的beanName的值进行匹配,相匹配的bean定义将被自动绑定到当前实例变量上。

其实也就是说不用显示指明哪个属于哪个。而是自动通过名称来匹配。

3. byType。容器会根据当前bean定义类型,分析其相应的依赖对象类型,然后到容器所管理的所有bean定义中寻找与依赖对象类型相同的bean定义,然后将找到的符合条件的bean自动绑定到当前bean定义。

4. constructor。针对构造方法参数的类型而进行的自动绑定。是byType类型的绑定。如果找到不止一个符合条件的bean定义,那么容器会返回错误。

5. autodetect。是byTeyp和constructor模式的结合体,如果对象拥有默认无参数的构造方法,容器会优先考虑byType的自动绑定模式。否则,会使用constructor模式。

通常情况下, 不介意多敲几个字符。

##### 4.3.3.5 lazy-init

延迟初始化。主要是可以针对Applicationcontext容器的bean初始化行为施以更多的控制。与BeanFactory不同,ApplicationContext在容器启动的时候,就会马上对所有的"singleton的bean定义"进行实例化操作。通常这种默认行为是好的,因为如果系统存在问题, 可以第一时间发现这些问题, 但有时候, 我们不想在容器启动后就直接实例化,总之, 我们想改变某个或者某些bean定义在ApplicationContext容器中的默认实例化时机。这样, 就可以通过<bean>的lazy-init属性来控制这种初始化行为。

```xml
<bena id="mylove" class=".." lazy-init="true" />
```

这样,ApplicationContext容器在启动的时候, 只会默认初始化非延迟加载的bean。

对于存在依赖关系的bean来说, 如果某个非延迟的bean定义依赖于延迟bean, 那么,容器还是会首先实例化延迟bean,再实例化后者。

如果我们真想保 证lazy-in让-bean—定会被延迟初始化的话，就需要保证依赖于该bean定义的其他bean定义也同样设置为延迟初始化。当然,可以在<beans>上设置default-lazy-init="true"来定义所有bean需要延迟初始化。

#### 4.3.4 继承

```xml
<bean id="subBean" parent="supBean" class="com.SupBean">
</bean>
```

这样就继承了supBean定义的默认值,只需要将特定的属性进行更改,而不要全部又重新定义一遍。

parent属性还可以与abstract属性结合使用,达到将相应bean宝岛模板化的目的。

abstract="true": 表明这个bean不需要实例化。通过parent指向这个模板定义,就拥有了该模板定义的所有属性配置。当多个bean定义拥有多个相同属性配置的时候,你会发现这种方式可以带来很大的便利。

#### 4.3.5 bean的scope

BeanFactory除了作为轻量级容器,还包括对象的生命周期管理。

scope用来声明容器中的对象所应该处的限定场景或者说该对象的存活时间,即容器在对象进入其相应scope之前,生成并装配这些对象, 在该对象不再处于这些scope的限定之后, 容器通常会销毁这些对象。

- 1. singleton。

     在Spring IoC容器中只存在一个实例,所有对该对象的引用将共享这个实例。该实例从容器启动,并因为第一次被请求而初始化之后,将一直存活到容器退出,也就是说, 它与IoC容器"几乎"拥有相同的"寿命"。

     ``注意``:不要因为名字的原因与单例模式混淆,二者的语音是不同的: 标记为singleton的ben是由容器来保证这种类型的bean在同一个容器中只存在一个共享实例; 而Singleton模式则是保证在同一个Classloader中只存在一个这种类型的实例。 

     可以从两个方面来看待singleton的bean所具有的特性。

     - 对象实例数量。
     - 对象存活时间。singleton类型的bean定义,从容器启动,到它第一次被请求而实例化开始,只要容器不销毁或者退出,该类型bean的单一实例就会一直存活。

     ```xml
     <bean id="mock" class="..." />
     <bean id="mock1" class="..." singleton="true"/>
     <bean id="mock2" class="..." scope="singleton" />
     ```

- 2. prototype。

  ​	容器在接到该类型对象的请求的时候,会每次都重新生成一个新的对象实例给请求方。虽然这种类型的对象的实例化以及属性设置等工作都是由容器负责的,但是只要准备完毕,并且对象实例返回给请求方之后,容器就不再拥有当前返回对象的引用,请求方需要自己负责当前返回对象的后继生命周期的管理工作,包括该对象销毁。

  ​	通常,声明为prototype的scope的bean定义类型,都是一些有状态的,比如保存每个顾客信息的对象。

- 3. request、session和global session

     这三个只适用于web应用程序,通常是与XmlWebApplicationContext共同使用。

     - reqeust。

       Spring容器,即XmlWebApplicationContext会为每个HTTP请求创建一个全新的Request-Processor对象代当前请求使用,当请求结束后,该对象实例的生命周期即告结束。当同时有10个HTTP请求进来时,容器会分别针对这10个请求返回10个全新的RequestProcessor对象实例,且它们之间互不干扰。不严格讲, request可以看作prototype的一种特例,除了场景更加具体之外,语意上差不多。

     - session。

       对于Web应用来说, 放到session中的最普遍的信息就是用户的登录信息,对于这种放到session中的信息, 我们可以指定其scope为session。

       ```xml
       <bean id="userSession" class="xxx" scope="session" />
       ```

       与reqeust相比, 除了拥有session scope的bean的实例具有比reqeust scope的bean 可能更长的存活时间,其他方面真是没有什么差别。

     - global session

       global session只有应用在基于protlet的web应用程序中都有意义,它映射到protlet的global范围的session。如果在普通的基于servlet的web应用中使用了这个类型的scope,容器会将其作为普通的session类型的scope对待。

     - 自定义scope类型

       你可以根据自己的需要或者应用场景,来添加自定义的scope类型。它们都实现了``org.springframework.beans.factory.config.Scope``接口,定义如下:

       ```java
       public interafce Scope {
           Object get(String name, ObjectFactory objectFactory);
           Object remove(String name);
           void registerDestructionCallback(String name, Runnable callback);
           String getConversationId();
       }
       ```

       ``get``和``remove``方法必须实现。

       ```xml
       <bean class="org.springframework.beans.factory.config.CustomScopeConfigurer">
       	<property name="scopes">
       		<map>
       			<entry key="thread" value="lchmagd.springjiemi.chapter4.ThreadScope" />
               </map>
           </property>
       </bean>
       
       <bean id="jiangjiajun" class="lchmagd.wuyuedecangjie.beans.Guangdong" scope="thread">
           <constructor-arg index="0" value="姜家俊" />
           <constructor-arg index="1" value="25" />
       </bean>
       ```

       ``注意:``使用了自定义scope的bean定义,需要该元素来为其在合适的时间创建和销毁相应的代理对象实例。对于request、session和global session 来说,也是如此。

#### 4.3.6 工厂方法与FactoryBean

如果该类是由我们设计并开发的，那么还好说，我们可以通过依赖注入，让容器帮助我们解除接口与实现类之间的耦合性。但是，有时，我们需要依赖第三方库，需要实例化并使用第三方库中的相关类，这时，接口与实现类的耦合性需要其他方式来避免。

通常的做法是通过使用工厂方法(Factory Method)模式,提供一个工厂类来实现实例化具体的接口实现类,这样,主体对象只需要依赖工厂类,具体使用的实现类有变更的话,只是变更工厂类,而主体对象不需要做任何变动。

```xml
<bean id="factory" class="..." factory-mothod="getInstance" />
```

``factory-method:`` 指定工厂方法名称,然后容器调用该静态工厂类的指定工厂方法,并返回方法调用后的结果。可以通过``<constructor-arg>``来指定工厂方法需要的参数。 

```xml
<bean id="factory" class="..." factory-mothod="getInstance" >
 <constructor-arg>
 	<ref bean="foobar" />
 </constructor-arg>
</bean>
```

- 2. 非静态工厂方法(Instance Factory Method)

  使用`factory-bean`属性来指定工厂方法所在的工厂类实例,而不是通过class属性来指定工厂方法所在类的类型。指定工厂方法名则相同,都是通过`factory-method`属性进行的。

- 3. FactoryBean

     `FactoryBean`是Spring容器提供的一种可以扩展容器对象实例化逻辑接口, 不要将其与容器名称BeanFactory相混淆。它本身与其他注册到容器的对象一样, 只是一个Bean而已,只不过, 这种类型的Bean本身就是生产对象的工厂(Factory)。
     
     当某些对象的实例化过程过于烦琐，通过XML配置过于复杂，使我们宁愿使用Java代码来完成这个实例化过程的时候，或者，某些第三方库不能直接注册到Spring容器的时候，就可以实现`org.springframework, beans, factory. FactoryBean `接口，给出自己的对象实例化逻辑代码。当然，不使用FactoryBean,而像通常那样实现自定义的工厂方法类也是可以的。不过，FactoryBeann可是Spring提供的对付这种情况的“制式装备”哦！
     
     ```java
     public interface FactoryBean {
     	Object getObject() throw Exception;
     Class getObjectType();
     boolean isSingleton();
     }
     ```
     
     - getObject()方法会返回该FactoryBean生产"的对象实例，我们需要实现该方法以给出自己 的对象实例化逻辑；
     - getObjectType ()方法仅返回getObject ()方法所返回的对象的类型，如果预先无法确定，则返回null;
     - isSingletonO方法返回结果用于表明，工厂方法(getObject ())所"生产啲对象是否要以singleton形式存在于容器中。如果以singleton形式存在,则返回true,否则返回false；
     
     ```xml
     <!--要使用实现FactoryBean的类, 只需要将其注册到容器即可-->
     <bean id="mockBean" class="xx">
     </bean>
     
     ```
     
     FactoryBean类型的bean定义,通过正常的id引用,窗口返回的是FactoryBean所"生产"的对象类型,而非FactoryBean实现本身。
     
     ​	如果一定要取得FactoryBean本身的话,可以通过在bean定义的id之前加前缀&来达到目的。Spring内部的许多地方都使用了FactoryBean。如
     
     - JndiObjectFactoryBean
     - LocalSessionFactoryBean
     - SqlMapClientFactoryBean
     - ProxyFactoryBean
     - TransactionProxyFactoryBean

#### 4.3.7 偷梁换柱之术

Spring窗口较为独特的功能特性:

- 方法注入(method injection)。只要让getNewBean方法声明符合规定的格式,并在配置文件中通知窗口,当该方法被调用的时候,每次返回指定类型的对象实例即可。该方法必须能够被子类实现或者覆写,因为窗口会为我们要进行方法注入的对象使用Cglib动态生成一个子类实现,从而替代当前对象。

  ```xml
  <bean id="xx" class="xx" >
  	<lookup-method name="getNewsBean" bean ="newBean" />
  </bean>
  ```

  通过<lookup-method>的name属性指定需要注入的方法名,bean属性指定需要注入的对象,当getNewBean方法被调用的时候,容器可以每次返回一个新的类型实例。

  △ 除了使用方法注入来达到"每次调用都让容器返回新的对象实例"的目的,还可以通过以下两种方式:

  - 1. BeanFactoryAware接口。这个方法是持有一个BeanFactory的引用, 这样就可以直接从容器当中获取实例。
  - 2. ObjectFactoryCreatingFactoryBean。这个类是Spring提供了一个FactoryBean实现, 它返回一个ObjectFactory实例。通过这个实例我们可以返回容器管理的相关对象。实际上,这个Bean实现了BeanFactoryFactoryAware接口,它返回的ObjectFActory实例只是特定于与Spring容器进行交互的一个实现而已。使用它的好处是, 隔离了客户端对象对BeanFactory的直接引用。

- 方法替换(Method replacement)。这种方式知道就行。实现MethodReplacer接口。在配置文件中注入即可。



### 4.4 容器背后的秘密

#### 4.4.1 战略性观望

Spring的IoC容器在实现的时候,基本上可以划分为两个阶段:

- 容器启动阶段
- Bean实例化阶段

Spring在实现的时候根据现阶段的不同特点,在每个阶段都加入了相应的容器扩展点,以便我们可以根据具体场景的需要加入自定义的扩展逻辑。

![1552290794731](https://cdn.jsdelivr.net/gh/Clarencezero/poi/1552290794731.png)

1. 容器启动阶段

   首先通过某种途径加载Configuration MetaData。在大部分情况下,容器需要依赖某些工具类(BeanDefinitionReader)对加载的Configuration MetaData进行解析和分析,并将分析后的信息编组为相应 的BeanDefinition,最后把这些保存了bean定义必要信息的BeanDefinition,注册到相应的BeanDefinitionRegistry,这样容器启动工作就完成了。总的来说, 该阶段工作可以认为是准备性的,重点更加侧重于对象管理信息的收集。当然,一些验证性或者辅助性的工作也可以在这个阶段完成。

2. Bean实例化阶段

   当某个请求方通过容器的`getBean`方法明确地请求某个对象,或者因依赖关系容器需要隐式地调用`getBean`方法时,就会触发第二阶段的活动。

#### 4.4.2 插手"容器的启动"

Spring提供了一种叫做`BeanFactoryPostProcessor`的容器扩展机制。该机制允许我们在容器实例化相应对象之前,对注册到容器的BeanDefinition所保存的信息做相应的修改。这就相当于在容器实现的第一阶段最后加入一道工序,让我们对最终的BeanDefinition做一些额外的操作,比如修改其中bean定义的某种属性,为bean定义增加其他信息等。

我们很少去直接实现BeanFactoryPostProcessor接口。其中，`org.springframework.beans. 
factory.config.PropertyPlaceholderConfigure`r和`org.springframework.beans.factory. 
config.Property OverrideConfigurer`是两个比较常用的BeanFactoryPostProcessor。另外，为
了处理配置文件中的数据类型与真正的业务对象所定义的数据类型转换，Spring还允许我们通过
org.springframework.beans.factory.config.CustomEditorConfigurer来注册自定义的PropertyEditor以补助容器中默认的PropertyEditor。

我们可以通过两种方式来应用`BeanFactoryPostProcessor`,分别针对基本的 IoC 容 器
BeanFactory和较为先进的容器ApplicationContext。

`BeanFactoryPostProcessor`实现可以完成的功能:

- 1. PropertyPlaceholderConfigurer

     通常情况下，我们不想将类似于系统管理相关的信息同业务对象相关的配置信息混杂到XML配置
     文件中，以免部署或者维护期间因为改动繁杂的XML配置文件而出现问题。我们会将一些数据库连接
     信息、邮件服务器等相关信息单独配置到一个properties文件中，这样，如果因系统资源变动的话，只
     需要关注这些简单properties配置文件即可。
     
     PropertyPlaceholderConfigurer允许我们在XML配置文件中使用占位符（PlaceHolder）,并将这些占位符所代表的资源单独配置到简单的properties文件中来加载。
     
     当BeanFactory在第一阶段加载完成所有配置信息时，BeanFactory中保存的对象的属性信息还只是以占位符的形式存在，如${jdbc.url}、${jdbc.driver}。当
     PropertyPlaceholderConfigurer作为BeanFactoryPostProcessor被应用时，它会使用properties
     配置文件中的配置信息来替换相应BeanDefinition中占位符所表示的属性值。这样，当进入容器实
     现的第二阶段实例化bean时，bean定义中的属性值就是最终替换完成的了。
     
     这个类不单从配置properties文件中加载配置项,同时还会检查Java的System类中的properties。

- 2. PropertyOverrideConfigurer

- 3. CustomEditorConfigurer

     - 自定义PropertyEditor

       通常情况下,对于Date类型,不同的Locale、不同的系统在表现形式上存在不同的需求。就好像这次，我们仅
       仅 让 DatePropertyEditor 完成从 String 到 java.util.Date 的转换，只需要实现
       setAsText(String)方法，而其他方法一概不管。
       
       ![1552442748489](https://cdn.jsdelivr.net/gh/Clarencezero/poi/1552442748489.png)

       我们需要将刚实现的DatePropertyEditor注册到容器，以告知容器按照DatePropertyEditor的形式进行String到java.util.Date类型的转换工作。
       
       ```java
       XmlBeanFactory beanFactory = new XmlBeanFactory(new ClassPathResource("..."));
       CustomEditorConfigurer ceConfigurer = new CustomEditorConfigurer(); 
       Map customEditors = new HashMap();
       CustomerEditors.put(java.util.Date.class, new DatePropertyEditor());
       ceConfigurer.setCustomEditors(customerEditors);
       ceConfigurer.postProcessBeanFactory(beanFactory);
       ```



#### 4.4.3 了解bean的一生

只有当请求方通过BeanFactory的getBean()方法来请求某个对象实例的时候,才有可能触发bean实例化阶段的活动。BeanFactory的getBean方法可能被客户端对象调用,也可以在容器内部隐式地被调用。

- 对于BeanFactory来说，对象实例化默认采用延迟初始化。
- ApplicationContext启动之后会实例化所有的bean定义。

![1552444157113](https://cdn.jsdelivr.net/gh/Clarencezero/poi/1552444157113.png)

1. Bean的实例化与BeanWrapper

   容器在内部实现的时候，采用“策略模式（Strategy Pattern）”来决定采用何种方式初始化bean实例。InstantiationStrategy定义是实例化策略
   的抽象接口，其直接子类SimpleInstantiationStrategy实现了简单的对象实例化功能，CglibSubclassingInstantiation￾Strategy继承了SimpleInstantiationStrategy的以反射方式实例化对象的功能，并且通过CGLIB
   的动态字节码生成功能，该策略实现类可以动态生成某个类的子类，进而满足了方法注入所需的对象
   实例化需求。默认情况下，容器内部采用的是CglibSubclassingInstantiationStrategy。
   
   但是，返回方
   式上有些“点缀”。不是直接返回构造完成的对象实例，而是以BeanWrapper对构造完成的对象实例
   进行包裹，返回相应的BeanWrapper实例。
   
   BeanWrapper接口通常在Spring框架内部使用。

2. 各色的Aware接口

   当对象实例化完成并且相关属性以及依赖设置完成之后，Spring容器会检查当前对象实例是否实
   现了一系列的以Aware命名结尾的接口定义。如果是,则将这些Aware接口定义中规定的依赖注入给当前对象实例。
   
   这些接口分为如下几个:
   
   - BeanNameAware。如果Spring容器检测到当前对象实
     例实现了该接口，会将该对象实例的bean定义对应的beanName设置到当前对象实例。
   - BeanClassLoaderAware。如果容器检测到当前对
     象实例实现了该接口，会将对应加载当前bean的Classloader注入当前对象实例。默认会使用
     加载org.springframework.util.ClassUtils类的Classloader。
   - BeanFactoryAware。在介绍方法注入的时候，我们
     提到过使用该接口以便每次获取prototype类型bean的不同实例。如果对象声明实现了
     BeanFactoryAware接口，BeanFactory容器会将自身设置到当前对象实例。这样，当前对象
     实例就拥有了一个BeanFactory容器的引用，并且可以对这个容器内允许访问的对象按照需要
     进行访问。
   
   这几个接口只针对BeanFactory类型的容器而言,对于ApplicationContext类型的容器,也存在几个Aware相关的接口。不过在检测这些接口并设置相关依赖的实现机理上,与以上几个接口处理方式有所不同,使用的是BeanPostProcessor方式。
   
   - ResourceLoaderAware 。 ApplicationContext 实现了
     Spring的ResourceLoader接口（后面会提及详细信息）。当容器检测到当前对象实例实现了ResourceLoaderAware接口之后，会将当前ApplicationContext自身设置到对象实例，这样当前对象实例就拥有了其所在ApplicationContext容器的一个引用。
   - ApplicationEventPublisherAware。ApplicationContext
     作为一个容器，同时还实现了ApplicationEventPublisher接口，这样，它就可以作为ApplicationEventPublisher来使用。所以，当前ApplicationContext容器如果检测到当前实例
     化的对象实例声明了ApplicationEventPublisherAware接口，则会将自身注入当前对象。
   - MessageSourceAware。ApplicationContext通过Message￾Source接口提供国际化的信息支持，即I18n（Internationalization）。它自身就实现了Message￾Source接口，所以当检测到当前对象实例实现了MessageSourceAware接口，则会将自身注入当前对象实例。
   - ApplicationContextAware。 如果ApplicationContext
     容器检测到当前对象实现了ApplicationContextAware接口，则会将自身注入当前对象实例。

3. BeanPostProcessor

   BeanPostProcessor的概念容易与BeanFactoryPostProcessor的概念混淆。但只要记住BeanPostProcessor是存在于对象实例化阶段，而BeanFactoryPostProcessor则是存在于容器启动阶段，这两个概念就比较容易区分了。
   
   BeanPostProcessor会处理容器内所有符合条件的实例化后的对象实例。
   
   通常比较常见的使用BeanPostProcessor的场景，是处理标记接口实现类，或者为当前对象提供代理实现。
   
   BeanPostProcessor是容器提供的对象实例化阶段的强有力的扩展点。
   
   - 自定义BeanPostProcessor
   
     - (1) 标注需要进行解密的实现类。声明一个接口。要求相关实现类实现该接口。
   
     - (2) 实现相应的BeanPostProcessor对符合条件的Bean实例进行处理。
   
     - (3) 将自定义的BeanPostProcessor注册到容器。
   
       对于BeanFactory类型的容器来说，我们需要通过手工编码的方式将相应的BeanPostProcessor注册到容器，也就是调ConfigurableBeanFactory的addBeanPostProcessor()方法。对于ApplicationContext容器来说，事情则方便得多，直接将相应的BeanPostProcessor实现类通过通常的XML配置文件配置一下即可。

4. InitializingBean和init-method

   InitializingBean是容器内部广泛使用的一个对象生命周期标识接口。
   
   作用在于，在对象实例化过程调用过“BeanPostProcessor的前置处理”
   之后，会接着检测当前对象是否实现了InitializingBean接口，如果是，则会调用其afterPropertiesSet()方法进一步调整对象实例的状态。
   
   此接口在Spring容器内部广泛使用, 但是如果让我们的业务实现这个接口,则显得Spring容器比较具有侵入性。所以Spring 提供了另一种方式指定对象初始化操作,那就是在XML配置的时候,使用<bean>的init-mehotd属性。
   
   如果系统开发过程中规定：所有业务对象的自定义初
   始化操作都必须以init()命名，为了省去挨个<bean>的设置init-method这样的烦琐，我们还可以通过最顶层的<beans>的default-init-method统一指定这一init()
   
   方法名。
   
   一般，我们是在集成第三方库，或者其他特殊的情况下，才会需要使用该特性。

5. DisposableBean与destroy-method

   对应的bean定义是否通过<bean>的destroy-method属性指定了自定义的对象销毁方法。
   
   最常见到的该功能的使用场景就是在Spring容器中注册数据库连接池，在系统退出后，连接池应该关闭，以释放相应资源。



#### 4.5 小结

Spring的IoC容器主要有两种，即BeanFactory和ApplicationContext。





































































