## 第 5 章 Spring IoC容器 ApplicationContext

作为Spring提供的较之BeanFactory更为先进的IoC容器实现，ApplicationContext除了拥有BeanFactory支持的所有功能之外，还进一步扩展了基本容器的功能，包括BeanFactoryPostProcessor、BeanPostProcessor以及其他特殊类型bean的自动识别、容器启动后bean实例的自动初始化、国际化的信息支持、容器内事件发布等。

Spring为基本的BeanFactory类型容器提供了XmlBeanFactory实现。相应地, 它也为ApplicationContext类型容器提供了以下几个常用的实现。

- FileSystemXmlApplicationContext。
- ClassPathXmlApplicationContext
- XmlWebApplicationContext

### 5.1 统一资源加载策略

#### 5.1.1 Spring中的Resource

#### 5.1.2 ResourceLoader, "更广义的URL"

查找和定位这些资源，则应该是ResourceLoader的职责所在了。

![1552455093586](https://cdn.jsdelivr.net/gh/Clarencezero/poi/1552455093586.png)

#### 5.1.3 ApplicationContext与ResourceLoader

会发现，ApplicationContext继承了ResourcePatternResolver，当
然就间接实现了ResourceLoader接口。所以，任何的ApplicationContext实现都可以看作是一个ResourceLoader甚至ResourcePatternResolver。而这就是ApplicationContext支持Spring内统一资源加载策略的真相。

AbstractApplicationContext 这个类可以看到ApplicationContext与ResourceLoader之间的所有关系。

![1552455686018](https://cdn.jsdelivr.net/gh/Clarencezero/poi/1552455686018.png)

作为ResourceLoader或者ResourcePatternResolver的ApplicationContext,到底何等神通。

1. 扮演ResourceLoader的角色

   既然ApplicationContext可以作为ResourceLoader或者ResourcePatternResolver来使用，那么我们可以通过ApplicationContext来加载任何Spring支持的Resource类型。
   
   而ResourceLoaderAware和ApplicationContextAware接口可以帮助我们将ApplicationContext容器本身注入到类中，只不过现在的FooBar需要依赖于Spring的API了。
   
   ```java
   public class FooBar implements applicationContextAware {
    private ResourceLoader resourceLoader;
    ...
   }
   ```
   
   容器启动的时候，就会自动将当前ApplicationContext容器本身
   注入到FooBar中，因为ApplicationContext类型容器可以自动识别Aware接口。



ResourcePatternResolver增加了一种: `classpath*:`。

classpath*:与classpath:的唯一区别就在于，如果能够在classpath中找到多个指定的资源，则返回多个。

当ClassPathXmlApplicationContext在实例化的时候，即使没有指明
classpath:或者classpath*:等前缀，它会默认从classpath中加载bean定义配置文件。而FileSystemXmlApplicationContext则有些不同，如果我们像如下代码那样指定conf/ appContext.xml，它会尝试从文件系统中加载bean定义文件：



### 5.2 国际化信息支持

对于Java中的国际化信息处理，主要涉及两个类，即java.util.Locale和java.util.ResourceBundle。

1. Locale。不同的Locale代表不同的国家和地区,每个国家和地区在Locale这里都有相应的简写代码表示。常用的Locale都提供静态常量。如Locale.CHINA。
2. ResourceBundle。用来保存特定于某个Locale的信息。通常，ResourceBundle管理一组信息序列，所有的信息序列有统一的一个basename，然后
   特定的Locale的信息，可以根据basename后追加的语言或者地区代码来区分。

#### 5.2.2 MessageSource与ApplicationContext

Spring在Java SE的国际化支持的基础上，进一步抽象了国际化信息的访问接口，也就是org.springframework.context.MessageSource。

```java
public interface MessageSource {

 /**
    * 资源条目的键、信息参数、Locale 。如果信息查找不到,返回默认信息
    **/
	@Nullable
	String getMessage(String code, @Nullable Object[] args, @Nullable String defaultMessage, Locale locale);

	/**
	* 
	**/
	String getMessage(String code, @Nullable Object[] args, Locale locale) throws NoSuchMessageException;

	// 使用MessageSourceResolvable对象对资源条目的键、信息参数等进行封装，将封住了这些信息的MessageSourceResolvable对象作为查询参数来调用以上方法。如果根据MessageSourceResolvable中的信息查找不到相应条目内容，将抛出NoSuchMessageException异常。

	String getMessage(MessageSourceResolvable resolvable, Locale locale) throws NoSuchMessageException;

}
```

通过该接口，我们统一了国际化信息的访问方式。传入相应的Locale、资源的键以及相应参数，就可以取得相应的信息，再也不用先根据Locale取得ResourceBundle，然后再从ResourceBundle查询信息了。

ApplicationContext除了实现了ResourceLoader以支持统一的资源加载，它还实现了MessageSource接口，那么就跟ApplicationContext因为实现了ResourceLoader而可以当作ResourceLoader来使用一样，ApplicationContext现在也是一个MessageSource了。

所以通常情况下，如果要提供容器内的国际化信息支持，我们会添加如代码清
单5-9类似的配置信息到容器的配置文件中。

![1552457146889](https://cdn.jsdelivr.net/gh/Clarencezero/poi/1552457146889.png)

![1552457161060](https://cdn.jsdelivr.net/gh/Clarencezero/poi/1552457161060.png)

1. 可用的MessageSource实现

   ResourceBundleMessageSource。提供对多个ResourceBundle的缓存以提高查询速度。它是最常用的、用于正式生产环境下的MessageSource实现。
   
   ![1552457356429](https://cdn.jsdelivr.net/gh/Clarencezero/poi/1552457356429.png)

2. MessageSourceAware和MessageSource的注入

   ApplicationContext启动的时候，会自动识别容器中类型为MessageSourceAware的bean定义,并将自身作为MessageSource注入相应对象实例中。如果某个业务对象需要国际化的信息支持,那么最简单的办法就是让它实现MessageSourceAware接口,然后注册到ApplicationContext容器。不过, 这样侵入性太强。
   
   实际上, 直接通过构造方法注入或者setter方法注入的方式声明依赖就可以了。

### 5.3 容器内部事件发布

#### 5.3.1 自定义事件发布

#### 5.3.2 Spring的容器内事件发布类结构分析

容器内部以ApplicationEvent的形式发布事件容器内注册的ApplicationListener类型的bean定义会被ApplicationContext容器自动识别,它们负责监听容器内发布的所有ApplicationContext类型的事件。也就是说,一旦容器内发布ApplicationEvent及其子类型的事件,注册到容器的ApplicationListener就会对这些事件进行处理。

- ApplicationEvent
- ApplicationListener
- ApplicationContext。转包给ApplicationEventMulticaster的接口,该接口定义了具体事件监听器的注册管理以及事件发布的方法,SimpleApplicationEventMulticaster是Spring提供的一个子类实现,添加了事件发布功能的实现。

Spring的ApplicationContext容器内的事件发布机制，主要用于单一容器内的简单消息通知和处理，并不适合分布式、多进程、多容器之间的事件通知。

可以通过两种方式为我们的业务对象注入ApplicationEventPublisher

- ApplicationEventPublisherAware接口。
- ApplicationContextAware接口。























