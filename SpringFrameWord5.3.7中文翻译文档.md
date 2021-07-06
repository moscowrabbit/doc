## Core Technologies

```
Version 5.3.7
```

本文档包括Spring FrameWork所有技术点。

其中最重要的是Spring框架的控制反转(IoC)容器。在对Spring框架的IoC容器进行了全面的处理之后，紧接着全面介绍了Spring的面向切面编程(AOP)技术。Spring框架有自己的AOP框架，它在概念上很容易理解，并且成功地解决了Java企业编程中80%的AOP需求。

还介绍了Spring与AspectJ的集成(就特性而言，目前AspectJ是最丰富的，当然也是Java企业空间中最成熟的AOP实现)。

## 1. The IoC Container

本章介绍Spring的控制反转(IoC)容器。

### 1.1. Introduction to the Spring IoC Container and Beans

本章介绍了控制反转(IoC)原理的Spring框架实现。IoC也称为依赖项注入(DI)。在这个过程中，对象只能通过构造函数参数、工厂方法的参数或在对象实例被构造或从工厂方法返回后设置的属性来定义它们的依赖关系(即它们使用的其他对象)。然后，容器在创建bean时注入这些依赖项。这个过程基本上是bean本身通过直接构造类或诸如Service Locator模式这样的机制来控制其依赖项的实例化或位置的逆过程(因此得名“控制反转”)。

`org.springframework.beans`和org.springframework.context包是Spring Framework控制反转(IoC)容器基本的包。BeanFactory接口提供了能够管理任何类型对象的高级配置机制。ApplicationContext是BeanFactory的一个子接口。增加了以下内容:

- 更容易与Spring的AOP特性集成
- 消息资源处理(用于国际化)
- 事件发布
- 应用层特定的上下文，如web应用中使用的WebApplicationContext。

简而言之，BeanFactory提供了配置框架和基本功能，而ApplicationContext添加了更多特定于企业的功能。ApplicationContext是BeanFactory的一个完整超集，在本章描述Spring的IoC容器时专门使用。有关使用BeanFactory而不是ApplicationContext的更多信息，请参见BeanFactory。

在Spring中，构成应用程序主干并由Spring IoC容器管理的对象称为bean。bean是由Spring IoC容器实例化、组装和管理的对象。否则，bean只是应用程序中的众多对象之一。bean及其之间的依赖关系反映在容器使用的配置元数据中。

### 1.2. Container Overview

`org.springframework.context.ApplicationContext`接口表示Spring IoC容器，负责实例化、配置和组装bean。容器通过读取配置元数据获得关于实例化、配置和组装哪些对象的指令。元数据用XML、Java注释或Java代码配置。它允许您表达组成应用程序的对象以及这些对象之间丰富的相互 依赖关系。

Spring提供了ApplicationContext接口的几个实现。在独立应用程序中，通常会创建ClassPathXmlApplicationContext或FileSystemXmlApplicationContext的实例。虽然XML是定义配置元数据的传统格式，但是您可以通过提供少量XML配置以声明方式支持这些额外的元数据格式，指示容器使用Java注释或代码作为元数据格式。

在大多数应用程序场景中，不需要显式的用户代码来实例化Spring IoC容器的一个或多个实例。例如，在一个web应用程序场景中，在应用程序的web. XML文件中简单的8行(大约)web描述符XML样本就足够了(参见方便的web应用程序上下文实例化)。如果您使用用于Eclipse的Spring Tools (Eclipse支持的开发环境)，那么只需点击几下鼠标或击几下键，就可以轻松创建这个样板配置。

下图显示了Spring如何工作的高级视图。应用程序类与配置元数据相结合，这样在ApplicationContext创建并初始化之后，您就拥有了一个配置完整且可执行的系统或应用程序。

![](C:\Users\dingbq\Desktop\spring文档\container-magic.png)

​                                                                                              **Figure 1. The Spring IoC container**

#### 1.2.1. Configuration Metadata

如上图所示，Spring IoC容器使用一种配置元数据形式。此配置元数据表示作为应用程序开发人员，您如何告诉Spring容器实例化、配置和组装应用程序中的对象。

配置元数据传统上以一种简单而直观的XML格式提供，本章的大部分内容都使用这种格式来传达Spring IoC容器的关键概念和特性。

```
基于xml的元数据并不是唯一允许的配置元数据形式。Spring IoC容器本身与配置元数据实际编写的格式完全分离。现在，许多开发人员为他们的Spring应用程序选择基于java的配置。
```

有关在Spring容器中使用其他形式的元数据配置方式，请参见:

- 基于注解的配置: Spring 2.5引入了对基于注解的配置元数据的支持。
- 基于java的配置:从Spring 3.0开始，Spring JavaConfig项目提供的许多特性成为了核心Spring框架的一部分。因此，您可以通过使用Java而不是XML文件来定义应用程序类外部的bean。要使用这些新特性，请参阅@Configuration、@Bean、@Import和@DependsOn注释。

Spring配置由容器必须管理的至少一个(通常是多个)bean定义组成。基于xml的配置元数据将这些bean配置为顶层元素中的元素。Java配置通常在@Configuration类中使用@ bean注释的方法。

这些bean定义对应于组成应用程序的实际对象。通常，您需要定义服务层对象、数据访问对象(dao)、表示对象(如Struts Action实例)、基础设施对象(如Hibernate SessionFactories、JMS队列)等等。通常，不需要在容器中配置细粒度的域对象，因为创建和加载域对象通常是dao和业务逻辑的责任。但是，您可以使用Spring与AspectJ的集成来配置在IoC容器控制之外创建的对象。请参阅使用AspectJ使用Spring依赖注入域对象。

下面的例子展示了基于xml的配置元数据的基本结构:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="..." class="...">  
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <bean id="..." class="...">
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions go here -->

</beans>
```

1. id属性是标识单个bean定义的字符串。
2. class属性定义了bean的类型，并使用完全限定类名。


id属性的值引用协作对象。引用协作对象的XML没有显示在这个示例中。有关更多信息，请参见Dependencies。

#### 1.2.2. Instantiating a Container

位置路径或提供给ApplicationContext构造函数的路径是资源字符串，这些资源字符串允许容器从各种外部资源(如本地文件系统、Java CLASSPATH等)加载配置元数据。

```java
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");
```

```
在了解了Spring的IoC容器之后，您可能想了解更多关于Spring的资源抽象(如参考资料中所述)的信息，它提供了一种方便的机制，用于从URI语法中定义的位置读取InputStream。特别是，资源路径用于构造应用程序上下文，如应用程序上下文和资源路径中所述。
```

以下示例显示了服务层对象(services.xml)配置文件:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- services -->

    <bean id="petStore"             class="org.springframework.samples.jpetstore.services.PetStoreServiceImpl">
        <property name="accountDao" ref="accountDao"/>
        <property name="itemDao" ref="itemDao"/>
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions for services go here -->

</beans>
```

下面的例子显示了数据访问对象dao .xml文件:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="accountDao"
        class="org.springframework.samples.jpetstore.dao.jpa.JpaAccountDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <bean id="itemDao" class="org.springframework.samples.jpetstore.dao.jpa.JpaItemDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions for data access objects go here -->

</beans>
```

在上面的示例中，服务层由PetStoreServiceImpl类和JpaAccountDao和JpaItemDao(基于JPA对象关系映射标准)类型的两个数据访问对 象组成。属性名元素引用JavaBean属性的名称，ref元素引用另一个bean定义的名称。id和ref元素之间的这种链接表示了协作对象之间的依赖关系。有关配置对象依赖关系的详细信息，请参见依赖关系。

#####  Composing XML-based Configuration Metadata

让bean定义跨越多个XML文件是很有用的。通常，每个单独的XML配置文件代表体系结构中的一个逻辑层或模块。

可以使用应用程序上下文构造函数从所有这些XML片段加载bean定义。此构造函数接受多个Resource位置，如前一节所示。或者，使用一次或多次元素从另一个或多个文件加载bean定义。下面的例子展示了如何做到这一点:

```xml
<beans>
    <import resource="services.xml"/>
    <import resource="resources/messageSource.xml"/>
    <import resource="/resources/themeSource.xml"/>

    <bean id="bean1" class="..."/>
    <bean id="bean2" class="..."/>
</beans>
```

在前面的示例中，外部bean定义从三个文件加载:services.xml、messageSource.xml和themeSource.xml。所有位置路径都相对于执行导入的定义文件，因此services.xml必须与执行导入的文件位于相同的目录或类路径位置，而messageSource.xml和themeSource.xml必须位于导入文件位置下面的资源位置。如您所见，前面的斜杠被忽略了。但是，考虑到这些路径是相对的，最好不要使用斜杠。根据Spring Schema，导入的文件的内容(包括顶级元素)必须是有效的XML bean定义。

```
可以使用相对的".. "来引用父目录中的文件，但不建议这样做。/”路径。这样做将创建对当前应用程序之外的文件的依赖关系。特别是，不建议对classpath: url(例如，classpath:../services.xml)使用此引用，运行时解析过程选择“最近的”类路径根，然后查看其父目录。类路径配置更改可能导致选择不同的、不正确的目录。
您可以始终使用完全限定的资源位置，而不是相对路径:例如，文件:C:/config/services.xml或类路径:/config/services.xml。但是，请注意，您是将应用程序的配置耦合到特定的绝对位置。通常，最好为这些绝对位置保留一个间接位置——例如，通过在运行时根据JVM系统属性解析的“${…}”占位符。
```

命名空间本身提供了导入指令特性。除了普通bean定义之外，Spring提供的XML名称空间中还提供了更多的配置特性——例如，上下文和util名称空间。

##### The Groovy Bean Definition DSL

作为外部化配置元数据的进一步例子，bean定义也可以在Spring的Groovy bean Definition DSL中表示，这从Grails框架中就可以知道。通常，这样的配置存在于。Groovy”文件的结构如下所示:

```java
beans {
    dataSource(BasicDataSource) {
        driverClassName = "org.hsqldb.jdbcDriver"
        url = "jdbc:hsqldb:mem:grailsDB"
        username = "sa"
        password = ""
        settings = [mynew:"setting"]
    }
    sessionFactory(SessionFactory) {
        dataSource = dataSource
    }
    myService(MyService) {
        nestedBean = { AnotherBean bean ->
            dataSource = dataSource
        }
    }
}
```

这种配置风格很大程度上等同于XML bean定义，甚至支持Spring的XML配置名称空间。它还允许通过importBeans指令导入XML bean定义文件。

#### 1.2.3. Using the Container

ApplicationContext是一个高级工厂的接口，该工厂能够维护不同bean及其依赖项的注册表。通过使用`T getBean(String name, Class requiredType)`方法，您可以检索bean的实例。

ApplicationContext允许你读取并访问bean定义，如下面的例子所示:

```java
// create and configure beans
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");

// retrieve configured instance
PetStoreService service = context.getBean("petStore", PetStoreService.class);

// use configured instance
List<String> userList = service.getUsernameList();
```

使用Groovy配置，引导看起来非常相似。它有一个不同的上下文实现类，它是groovy感知的(但也理解XML bean定义)。下面的例子展示了Groovy的配置:

```java
ApplicationContext context = new GenericGroovyApplicationContext("services.groovy", "daos.groovy");
```

最灵活的变体是GenericApplicationContext与阅读器委托结合使用——例如，XML文件使用XmlBeanDefinitionReader，如下例所示:

```java
GenericApplicationContext context = new GenericApplicationContext();
new XmlBeanDefinitionReader(context).loadBeanDefinitions("services.xml", "daos.xml");
context.refresh();
```

你也可以对Groovy文件使用GroovyBeanDefinitionReader，如下面的例子所示:

```java
GenericApplicationContext context = new GenericApplicationContext();
new GroovyBeanDefinitionReader(context).loadBeanDefinitions("services.groovy", "daos.groovy");
context.refresh();
```

您可以在相同的ApplicationContext上混合和匹配这样的读取器委托，从不同的配置源读取bean定义。

然后可以使用getBean来检索bean的实例。ApplicationContext接口有一些其他的方法用于检索bean，但是，理想情况下，应用程序代码不应该使用它们。实际上，您的应用程序代码根本不应该调用getBean()方法，因此根本不依赖于Spring api。例如，Spring与web框架的集成为各种web框架组件(如控制器和jsf管理的bean)提供了依赖注入，允许您通过元数据(如自动装配注释)声明对特定bean的依赖。

### 1.3. Bean Overview

Spring IoC容器管理一个或多个bean。这些bean是使用您提供给容器的配置元数据创建的(例如，以XML 定义的形式)。

在容器本身内，这些bean定义表示为BeanDefinition对象，其中包含(在其他信息中)以下元数据:

- 包限定的类名:通常是定义的bean的实际实现类。
- Bean行为配置元素，声明Bean在容器中应该如何行为(范围、生命周期回调，等等)。
- 对其他bean的引用是该bean完成其工作所必需的。这些引用也称为合作者或依赖关系。
- 要在新创建的对象中设置的其他配置设置—例如，池的大小限制或管理连接池的bean中要使用的连接数。

此元数据转换为组成每个bean定义的一组属性。下表描述了这些属性:

​                                                              **Table 1. The bean definition**

|         Property         | Explained in…                                                |
| :----------------------: | ------------------------------------------------------------ |
|          Class           | [Instantiating Beans](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-class) |
|           Name           | [Naming Beans](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-beanname) |
|          Scope           | [Bean Scopes](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes) |
|  Constructor arguments   | [ Dependency Injection](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-collaborators) |
|        Properties        | [Dependency Injection](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-collaborators) |
|     Autowiring mode      | [ Autowiring Collaborators](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-autowire) |
| Lazy initialization mode | [Lazy-initialized Beans](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lazy-init) |
|  Initialization method   | [Initialization Callbacks](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle-initializingbean) |
|    Destruction method    | [Destruction Callbacks](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle-disposablebean) |

除了包含关于如何创建特定bean的信息的bean定义外，`ApplicationContext`实现还允许注册(由用户)在容器外部创建的现有对象。这是通过`getBeanFactory()`方法访问`ApplicationContext`的`BeanFactory`来完成的，该方法返回`BeanFactory DefaultListableBeanFactory`实现。`DefaultListableBeanFactory`通过`registerSingleton(..)`和`registerBeanDefinition(..)`方法支持这种注册。但是，典型的应用程序只使用通过常规bean定义元数据定义的bean。

Bean元数据和手动提供的单例实例需要尽早注册，以便容器在自动装配和其他自省步骤中正确地对它们进行推理。虽然在某种程度上支持覆盖现有的元数据和现有的单例实例，但是在运行时注册新bean(与对工厂的实时访问同时进行)并没有得到官方支持，并且可能导致并发访问异常、bean容器中的不一致状态，或者两者都有。

#### 1.3.1. Naming Beans

每个bean都有一个或多个标识符。这些标识符在承载bean的容器中必须是唯一的。一个bean通常只有一个标识符。但是，如果需要一个以上的别名，则可以将额外的别名视为别名。

在基于xml的配置元数据中，可以使用id属性、name属性或两者来指定bean标识符。id属性允许您指定一个id。通常，这些名称是字母数字('myBean'， 'someService'等)，但它们也可以包含特殊字符。如果希望为bean引入其他别名，还可以在name属性中指定它们，用逗号(，)、分号(;)或空格分隔。作为历史记录，在Spring 3.1之前的版本中，id属性被定义为xsd: id类型，这限制了可能的字符。从3.1开始，它被定义为xsd:string类型。注意，bean id惟一性仍然由容器强制执行，但不再由XML解析器强制执行。

您不需要为bean提供名称或id。如果您没有显式地提供名称或id，容器将为该bean生成唯一的名称。但是，如果希望通过使用ref元素或Service Locator样式查找按名称引用该bean，则必须提供名称。不提供名称的动机与使用内部bean和自动装配协作者有关。

```
                                                     Bean Naming Conventions 
约定是在命名bean时对实例字段名使用标准Java约定。也就是说，bean名称以小写字母开头，并从那里开始采用驼峰大小写。这些名称的示例包括accountManager、accountService、userDao、loginController等。
一致地命名bean可以使您的配置更易于阅读和理解。另外，如果您使用Spring AOP，那么将通知应用到名称相关的一组bean时，它会有很大帮助。
```

```
通过在类路径中扫描组件，Spring为未命名组件生成bean名，遵循前面描述的规则:本质上，采用简单的类名并将其初始字符转换为小写。但是，在(不常见的)特殊情况下，当有多个字符且第一个和第二个字符都是大写时，保留原来的大小写。这些规则与java.beans.Introspector.decapitalize (Spring在这里使用的)定义的规则相同。
```

##### Aliasing a Bean outside the Bean Definition

在bean定义本身中，通过使用id属性指定的最多一个名称和name属性中任意数量的其他名称的组合，您可以为bean提供多个名称。这些名称可以是相同bean的等效别名，在某些情况下非常有用，例如通过使用特定于该组件本身的bean名称，让应用程序中的每个组件引用公共依赖项。

但是，指定bean实际定义的所有别名并不总是足够的。有时需要为在其他地方定义的bean引入别名。在大型系统中，配置在每个子系统之间分割，每个子系统都有自己的对象定义集，这种情况很常见。在基于xml的配置元数据中，可以使用元素来实现这一点。下面的例子展示了如何做到这一点:

```xml
<alias name="fromName" alias="toName"/>
```

在这种情况下，在使用这个别名定义之后，命名为fromName的bean(在同一个容器中)也可以称为toName。

例如，子系统A的配置元数据可以通过subsystemA-dataSource的名称引用一个数据源。子系统B的配置元数据可以通过subsystemB-dataSource的名称来引用一个数据源。当组合使用这两个子系统的主应用程序时，主应用程序通过myApp-dataSource的名称引用数据源。要让这三个名称都引用同一个对象，可以在配置元数据中添加以下别名定义:

```xml
<alias name="myApp-dataSource" alias="subsystemA-dataSource"/>
<alias name="myApp-dataSource" alias="subsystemB-dataSource"/>
```

现在，每个组件和主应用程序都可以通过一个惟一的名称来引用dataSource，这个名称保证不会与任何其他定义冲突(有效地创建了一个名称空间)，但它们引用的是同一个bean。

```
                                                    Java-configuration
如果使用java conconfiguration，可以使用@Bean注释提供别名。详细信息请参见使用@Bean注释。
```

#### 1.3.2. Instantiating Beans

bean定义本质上是创建一个或多个对象的方法。容器在被询问时查看命名bean的菜谱，并使用该bean定义封装的配置元数据创建(或获取)实际对象。

如果使用基于xml的配置元数据，则需要指定要在元素的class属性中实例化的对象的类型(或类)。这个类属性(在内部是BeanDefinition实例上的class属性)通常是强制性的。(对于异常，请参见使用实例工厂方法和Bean定义继承进行实例化。)你可以用以下两种方式之一来使用Class属性:

- 通常，在容器本身通过反射调用构造函数直接创建bean的情况下，指定要构造的bean类，这在某种程度上相当于带有new操作符的Java代码。
- 要指定包含用于创建对象的静态工厂方法的实际类，容器调用类上的静态工厂方法来创建bean的情况不太常见。调用静态工厂方法返回的对象类型可以是同一个类，也可以是另一个完全相同的类。

```
                                                     Nested class names
如果您想为嵌套类配置bean定义，您可以使用嵌套类的二进制名称或源名称。
例如，如果你在com中有一个叫做SomeThing的类。这个SomeThing类有一个名为OtherThing的静态嵌套类，它们可以用美元符号($)或点(.)分隔。因此，bean定义中的class属性的值应该是com.example。OtherThing或com.example.SomeThing.OtherThing美元的东西。
```

#####  Instantiation with a Constructor

当您通过构造函数方法创建一个bean时，所有普通的类都可以被Spring使用，并且与Spring兼容。也就是说，正在开发的类不需要实现任何特定的接口，也不需要以特定的方式编码。只需指定bean类就足够了。但是，根据您为特定bean使用的IoC类型，您可能需要一个默认(空)构造函数。

Spring IoC容器实际上可以管理您希望它管理的任何类。它并不局限于管理真正的javabean。大多数Spring用户更喜欢只有默认(无参数)构造函数和根据容器中的属性建模的适当setter和getter的实际javabean。您还可以在容器中添加更多非bean风格的类。例如，如果您需要使用绝对不遵守JavaBean规范的遗留连接池，Spring也可以管理它。

使用基于xml的配置元数据，你可以如下指定你的bean类:

```xml
<bean id="exampleBean" class="examples.ExampleBean"/>
<bean name="anotherExample" class="examples.ExampleBeanTwo"/>
```

有关向构造函数提供参数(如果需要)和在对象构造后设置对象实例属性的机制的详细信息，请参见注入依赖项。

##### Instantiation with a Static Factory Method

在定义使用静态工厂方法创建的bean时，使用class属性指定包含静态工厂方法的类，并使用名为factory-method的属性指定工厂方法本身的名称。您应该能够调用这个方法(带有可选参数，如后面所述)并返回一个活动对象，该对象随后被视为通过构造函数创建的。这种bean定义的一个用途是在遗留代码中调用静态工厂。

下面的bean定义指定通过调用工厂方法来创建bean。该定义不指定返回对象的类型(类)，只指定包含工厂方法的类。在这个例子中，createInstance()方法必须是一个静态方法。下面的例子展示了如何指定工厂方法:

```xml
<bean id="clientService"
    class="examples.ClientService"
    factory-method="createInstance"/>
```

下面的例子展示了一个可以使用前面的bean定义的类:

```java
public class ClientService {
    private static ClientService clientService = new ClientService();
    private ClientService() {}

    public static ClientService createInstance() {
        return clientService;
    }
}
```

有关为工厂方法提供(可选)参数和在从工厂返回对象后设置对象实例属性的机制的详细信息，请参见详细信息中的依赖关系和配置。

##### Instantiation by Using an Instance Factory Method

与通过静态工厂方法进行实例化类似，使用实例工厂方法的实例化从容器调用现有bean的非静态方法来创建新bean。要使用这种机制，请将class属性保留为空，并在工厂bean属性中，指定当前(或父或祖先)容器中bean的名称，该容器包含要被调用来创建对象的实例方法。使用工厂方法属性设置工厂方法本身的名称。下面的例子展示了如何配置这样一个bean:

```xml
<!-- the factory bean, which contains a method called createInstance() -->
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<!-- the bean to be created via the factory bean -->
<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>
```

下面的例子显示了相应的类:

```java
public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();

    public ClientService createClientServiceInstance() {
        return clientService;
    }
}
```

一个工厂类也可以包含多个工厂方法，如下面的例子所示:

```xml
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>

<bean id="accountService"
    factory-bean="serviceLocator"
    factory-method="createAccountServiceInstance"/>
```

下面的例子显示了相应的类:

```java
public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();

    private static AccountService accountService = new AccountServiceImpl();

    public ClientService createClientServiceInstance() {
        return clientService;
    }

    public AccountService createAccountServiceInstance() {
        return accountService;
    }
}
```

这种方法表明工厂bean本身可以通过依赖项注入(DI)进行管理和配置。请参阅详细的依赖关系和配置。

```
在Spring文档中，“工厂bean”指的是在Spring容器中配置的bean，它通过实例或静态工厂方法创建对象。相反，FactoryBean(注意大写)引用一个特定于spring的FactoryBean实现类。
```

##### Determining a Bean’s Runtime Type

确定特定bean的运行时类型是非常重要的。bean元数据定义中的指定类只是一个初始类引用，可能与声明的工厂方法或FactoryBean类相结合，这可能导致bean的不同运行时类型，或者在实例级工厂方法(而是通过指定的工厂bean名称解析)的情况下，根本没有设置。此外，AOP代理可以用一个基于接口的代理来包装一个bean实例，并有限地暴露目标bean的实际类型(仅仅是实现的接口)。

要了解特定bean的实际运行时类型，推荐使用BeanFactory。getType调用指定的bean名。这将考虑上述所有情况，并返回BeanFactory对象的类型。getBean调用将返回相同的bean名。

### 1.4. Dependencies

典型的企业应用程序不是由单个对象(或者用Spring的说法是bean)组成的。即使是最简单的应用程序也有一些共同工作的对象，以呈现终端用户所认为的一致的应用程序。下一节将解释如何从定义大量独立的bean定义过渡到一个完全实现的应用程序，其中对象通过协作实现目标。

#### 1.4.1. Dependency Injection

依赖注入(DI)是一个过程，对象通过构造函数参数、工厂方法的参数或在对象实例被构造或从工厂方法返回后设置的属性来定义它们的依赖项(即它们工作的其他对象)。然后，容器在创建bean时注入这些依赖项。这个过程基本上是bean本身通过直接构造类或Service Locator模式控制其依赖项的实例化或位置的逆过程(因此得名“控制反转”)。

使用DI原则，代码会更清晰，当对象与它们的依赖项一起提供时，解耦会更有效。对象不查找它的依赖项，也不知道依赖项的位置或类。因此，您的类变得更容易测试，特别是当依赖项在接口或抽象基类上时，这允许在单元测试中使用存根或模拟实现。

依赖注入主要有两种变体:基于构造函数的依赖注入和基于setter的依赖注入。

##### Constructor-based Dependency Injection

基于构造函数的DI是通过容器调用带有许多参数的构造函数来完成的，每个参数表示一个依赖项。调用带有特定参数的静态工厂方法来构造bean几乎是等价的，本文以类似的方式处理构造函数的参数和静态工厂方法的参数。下面的例子展示了一个只能通过构造函数注入进行依赖注入的类:

```java
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on a MovieFinder
    private final MovieFinder movieFinder;

    // a constructor so that the Spring container can inject a MovieFinder
    public SimpleMovieLister(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // business logic that actually uses the injected MovieFinder is omitted...
}
```

注意，这个类没有什么特别之处。它是一个不依赖于容器特定接口、基类或注释的POJO。

**Constructor Argument Resolution**

构造函数参数解析匹配是通过使用参数的类型来实现的。如果bean定义的构造函数参数中不存在潜在的歧义，那么在bean定义中定义构造函数参数的顺序就是在实例化bean时将这些参数提供给适当的构造函数的顺序。考虑以下类:

```java
package x.y;

public class ThingOne {

    public ThingOne(ThingTwo thingTwo, ThingThree thingThree) {
        // ...
    }
}
```

假设ThingTwo和ThingThree类没有继承关系，就不存在潜在的歧义。因此，以下配置可以正常工作，并且不需要在元素中显式指定构造函数参数索引或类型。

```xml
<beans>
    <bean id="beanOne" class="x.y.ThingOne">
        <constructor-arg ref="beanTwo"/>
        <constructor-arg ref="beanThree"/>
    </bean>
    <bean id="beanTwo" class="x.y.ThingTwo"/>
    <bean id="beanThree" class="x.y.ThingThree"/>
</beans>
```

当引用另一个bean时，类型是已知的，可以进行匹配(就像前面的示例一样)。当使用简单类型时，例如true， Spring不能确定值的类型，因此在没有帮助的情况下不能按类型匹配。考虑以下类:

```java
package examples;

public class ExampleBean {

    // Number of years to calculate the Ultimate Answer
    private final int years;

    // The Answer to Life, the Universe, and Everything
    private final String ultimateAnswer;

    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }
}
```

**Constructor argument type matching**

在前面的场景中，如果使用type属性显式指定构造函数参数的类型，容器可以使用简单类型的类型匹配，如下面的示例所示:

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg type="int" value="7500000"/>
    <constructor-arg type="java.lang.String" value="42"/>
</bean>
```

**Constructor argument index**

你可以使用index属性显式地指定构造函数参数的索引，如下面的例子所示:

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg index="0" value="7500000"/>
    <constructor-arg index="1" value="42"/>
</bean>
```

除了解决多个简单值的歧义外，指定索引还解决了构造函数具有两个相同类型参数的歧义。

```
The index is 0-based.
```

**Constructor argument name**

你也可以使用构造函数的参数名来消除值的歧义，如下面的例子所示:

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg name="years" value="7500000"/>
    <constructor-arg name="ultimateAnswer" value="42"/>
</bean>
```

请记住，要使其开箱即用，您的代码必须在启用调试标志的情况下编译，以便Spring可以从构造函数中查找参数名。如果不能或不想使用调试标志编译代码，可以使用@ConstructorProperties JDK注释显式地命名构造函数参数。样例类应该如下所示:

```java
package examples;

public class ExampleBean {

    // Fields omitted

    @ConstructorProperties({"years", "ultimateAnswer"})
    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }
}
```

#####  Setter-based Dependency Injection

基于setter的DI是通过容器在调用无参数构造函数或无参数静态工厂方法来实例化bean之后调用bean上的setter方法来完成的。

下面的例子展示了一个只能通过使用纯setter注入来进行依赖注入的类。这个类是传统的Java。它是一个不依赖于容器特定接口、基类或注释的POJO。

```java
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on the MovieFinder
    private MovieFinder movieFinder;

    // a setter method so that the Spring container can inject a MovieFinder
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // business logic that actually uses the injected MovieFinder is omitted...
}
```

ApplicationContext为它管理的bean支持基于构造函数和基于setter的DI。在通过构造函数方法注入了一些依赖项之后，它还支持基于setter的DI。您可以以BeanDefinition的形式配置依赖项，可以与PropertyEditor实例一起使用，将属性从一种格式转换为另一种格式。但是，大多数Spring用户并不是直接使用这些类(即通过编程方式)，而是使用XML bean定义、带注释的组件(即使用@Component、@Controller等带注释的类)，或者基于java的@Configuration类中的@Bean方法。然后，这些源在内部转换为BeanDefinition的实例，并用于加载整个Spring IoC容器实例。

```
                          Constructor-based or setter-based DI?
由于可以混合使用基于构造函数和基于setter的DI，因此对于强制依赖项使用构造函数，对于可选依赖项使用setter方法或配置方法是一条很好的经验法则。注意，在setter方法上使用@Required注释可以使属性成为必需的依赖项;然而，使用编程式验证参数的构造函数注入更可取。
Spring团队通常提倡构造函数注入，因为它允许您将应用程序组件实现为不可变对象，并确保所需的依赖项不为空。此外，构造函数注入的组件总是在完全初始化状态下返回给客户机(调用)代码。顺便说一下，大量构造函数参数是一种糟糕的代码味道，这意味着类可能有太多的职责，应该进行重构以更好地解决关注事项的适当分离。
Setter注入应该主要用于可选的依赖项，这些依赖项可以在类中分配合理的默认值。否则，必须在代码使用依赖项的任何地方执行非空检查。setter注入的一个好处是，setter方法使该类的对象能够在以后重新配置或重新注入。因此，通过JMX mbean进行管理是setter注入的一个引人注目的用例。
使用对特定类最有意义的DI风格。有时，当您处理没有源代码的第三方类时，您可以自行选择。例如，如果第三方类没有公开任何setter方法，那么构造函数注入可能是DI的唯一可用形式。
```

##### Dependency Resolution Process

容器执行bean依赖关系解析如下:

- 使用描述所有bean的配置元数据创建并初始化ApplicationContext。配置元数据可以通过XML、Java代码或注释指定。
- 对于每个bean，它的依赖关系以属性、构造函数参数或静态工厂方法的参数的形式表示(如果您使用静态工厂方法而不是普通构造函数)。当实际创建bean时，这些依赖项被提供给bean。
- 每个属性或构造函数参数都是要设置的值的实际定义，或对容器中另一个bean的引用。
- 作为值的每个属性或构造函数参数都将从其指定的格式转换为该属性或构造函数参数的实际类型。默认情况下，Spring可以将字符串格式提供的值转换为所有内置类型，比如int、long、string、boolean等等。

Spring容器在创建容器时验证每个bean的配置。但是，在实际创建bean之前，不会设置bean属性本身。在创建容器时将创建单例作用域且设置为预实例化(默认)的bean。作用域在Bean作用域中定义。否则，仅在请求bean时才创建它。创建一个bean可能会导致创建一个bean图，因为创建并分配了bean的依赖项及其依赖项的依赖项(等等)。请注意，这些依赖项之间的解析不匹配可能会在后期出现——即在受影响bean的第一次创建时出现。

```
                                                         Circular dependencies
如果主要使用构造函数注入，可能会创建不可解析的循环依赖项场景。
例如:类A通过构造函数注入需要类B的一个实例，类B通过构造函数注入需要类A的一个实例。如果您将类A和类B的bean配置为相互注入，那么Spring IoC容器将在运行时检测这个循环引用，并抛出beancurcurrentlyincreationexception。
一种可能的解决方案是编辑一些由setter而不是构造函数配置的类的源代码。或者，避免构造函数注入，只使用setter注入。换句话说，尽管不推荐这样做，但您可以使用setter注入配置循环依赖项。
与典型情况(没有循环依赖项)不同，bean a和bean B之间的循环依赖项迫使一个bean在完全初始化自己之前被注入到另一个bean中(典型的先有鸡还是先有蛋的场景)。
```

您通常可以相信Spring会做正确的事情。它在容器加载时检测配置问题，比如对不存在的bean和循环依赖项的引用。当bean实际创建时，Spring会尽可能晚地设置属性并解析依赖项。这意味着，如果在创建对象或其某个依赖项时出现问题，请求对象时，正确加载的Spring容器可以生成异常——例如，bean会由于缺少或无效的属性而抛出异常。这可能会延迟一些配置问题的可见性，这就是为什么ApplicationContext实现默认情况下会预先实例化单例bean。在实际需要这些bean之前创建这些bean需要花费一些前期时间和内存，因此在创建ApplicationContext时发现配置问题，而不是稍后。您仍然可以覆盖这个默认行为，以便单例bean可以延迟初始化，而不是急切地预先实例化。

如果不存在循环依赖项，那么当一个或多个协作bean被注入到依赖bean中时，每个协作bean在被注入到依赖bean之前都会被完全配置。这意味着，如果bean A依赖于bean B, Spring IoC容器在调用bean A的setter方法之前完全配置bean B。换句话说，bean被实例化(如果它不是一个预实例化的单例)，它的依赖项被设置，并且调用相关的生命周期方法(例如配置的init方法或InitializingBean回调方法)。

##### Examples of Dependency Injection

下面的示例为基于setter的DI使用基于xml的配置元数据。Spring XML配置文件的一小部分指定了一些bean定义，如下所示:

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <!-- setter injection using the nested ref element -->
    <property name="beanOne">
        <ref bean="anotherExampleBean"/>
    </property>
    <!-- setter injection using the neater ref attribute -->
    <property name="beanTwo" ref="yetAnotherBean"/>
    <property name="integerProperty" value="1"/>
</bean>
<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

下面的例子显示了相应的ExampleBean类:

```java
public class ExampleBean {

    private AnotherBean beanOne;

    private YetAnotherBean beanTwo;

    private int i;

    public void setBeanOne(AnotherBean beanOne) {
        this.beanOne = beanOne;
    }

    public void setBeanTwo(YetAnotherBean beanTwo) {
        this.beanTwo = beanTwo;
    }

    public void setIntegerProperty(int i) {
        this.i = i;
    }
}
```

在前面的示例中，setter被声明为与XML文件中指定的属性相匹配。下面的例子使用了基于构造函数的DI:

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <!-- constructor injection using the nested ref element -->
    <constructor-arg>
        <ref bean="anotherExampleBean"/>
    </constructor-arg>
    <!-- constructor injection using the neater ref attribute -->
    <constructor-arg ref="yetAnotherBean"/>
    <constructor-arg type="int" value="1"/>
</bean>
<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

下面的例子显示了相应的ExampleBean类:

```java
public class ExampleBean {

    private AnotherBean beanOne;

    private YetAnotherBean beanTwo;

    private int i;

    public ExampleBean(
        AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {
        this.beanOne = anotherBean;
        this.beanTwo = yetAnotherBean;
        this.i = i;
    }
}
```

在bean定义中指定的构造函数参数被用作ExampleBean的构造函数的参数。

现在考虑这个例子的一个变体，这里不使用构造函数，而是告诉Spring调用一个静态工厂方法来返回对象的实例:

```xml
<bean id="exampleBean" class="examples.ExampleBean" factory-method="createInstance">
    <constructor-arg ref="anotherExampleBean"/>
    <constructor-arg ref="yetAnotherBean"/>
    <constructor-arg value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

下面的例子显示了相应的ExampleBean类:

```java
public class ExampleBean {

    // a private constructor
    private ExampleBean(...) {
        ...
    }

    // a static factory method; the arguments to this method can be
    // considered the dependencies of the bean that is returned,
    // regardless of how those arguments are actually used.
    public static ExampleBean createInstance (
        AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {

        ExampleBean eb = new ExampleBean (...);
        // some other operations...
        return eb;
    }
}
```

静态工厂方法的参数是由元素提供的，就像实际使用了构造函数一样。工厂方法返回的类的类型不必与包含静态工厂方法的类的类型相同(尽管在本例中是相同的)。实例(非静态)工厂方法可以以本质上相同的方式使用(除了使用工厂bean属性而不是class属性)，所以我们在这里不讨论这些细节。

#### 1.4.2. Dependencies and Configuration in Detail

如前一节所述，可以将bean属性和构造函数参数定义为对其他托管bean(协作者)的引用，或内联定义的值。Spring基于xml的配置元数据为此目的支持和元素中的子元素类型。

#####  Straight Values (Primitives, Strings, and so on)

元素的value属性将属性或构造函数参数指定为人类可读的字符串表示形式。Spring的转换服务用于将这些值从String转换为属性或参数的实际类型。下面的示例显示了正在设置的各种值:

```xml
<bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
    <!-- results in a setDriverClassName(String) call -->
    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/mydb"/>
    <property name="username" value="root"/>
    <property name="password" value="misterkaoli"/>
</bean>
```

下面的例子使用p-namespace来进行更简洁的XML配置:

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource"
        destroy-method="close"
        p:driverClassName="com.mysql.jdbc.Driver"
        p:url="jdbc:mysql://localhost:3306/mydb"
        p:username="root"
        p:password="misterkaoli"/>

</beans>
```

前面的XML更简洁。然而，打字错误是在运行时而不是设计时发现的，除非您使用支持创建bean定义时自动完成属性的IDE(如IntelliJ IDEA或用于Eclipse的Spring Tools)。强烈推荐这样的IDE帮助。

您也可以配置一个java.util.Properties实例，如下所示:

```xml
<bean id="mappings"
    class="org.springframework.context.support.PropertySourcesPlaceholderConfigurer">

    <!-- typed as a java.util.Properties -->
    <property name="properties">
        <value>
            jdbc.driver.className=com.mysql.jdbc.Driver
            jdbc.url=jdbc:mysql://localhost:3306/mydb
        </value>
    </property>
</bean>
```

通过使用JavaBeans的PropertyEditor机制，Spring容器将元素中的文本转换为java.util.Properties实例。这是一个很好的快捷方式，也是Spring团队喜欢使用嵌套元素而不是value属性样式的少数地方之一。

**The** `idref` **element**

idref元素只是将容器中另一个bean的id(字符串值——而不是引用)传递给或元素的一种防止错误的方法。下面的例子展示了如何使用它:

```xml
<bean id="theTargetBean" class="..."/>

<bean id="theClientBean" class="...">
    <property name="targetName">
        <idref bean="theTargetBean"/>
    </property>
</bean>
```

前面的bean定义片段(在运行时)与下面的片段完全相同:

```xml
<bean id="theTargetBean" class="..." />

<bean id="client" class="...">
    <property name="targetName" value="theTargetBean"/>
</bean>
```

第一种形式比第二种形式更可取，因为使用idref标记可以让容器在部署时验证所引用的已命名bean实际存在。在第二个变体中，对传递给客户机bean的targetName属性的值不执行任何验证。只有在实际实例化客户端bean时才会发现打字错误(很可能导致致命的结果)。如果客户端bean是一个原型bean，那么这个输入错误和由此产生的异常可能要在容器部署很久之后才会被发现。

```
在4.0 bean XSD中不再支持idref元素的本地属性，因为它不再提供常规bean引用之上的值。升级到4.0模式时，将现有的idref本地引用更改为idref bean。
```

元素带来值的一个常见地方(至少在Spring 2.0之前的版本中)是在ProxyFactoryBean bean定义中的AOP拦截器配置中。在指定拦截器名称时使用元素可以防止你拼错拦截器ID。

##### References to Other Beans (Collaborators)

ref元素是或定义元素中的最后一个元素。在这里，您将一个bean的指定属性的值设置为对容器管理的另一个bean(合作者)的引用。被引用的bean是要设置其属性的bean的依赖项，在设置属性之前，根据需要对其进行初始化。(如果合作者是一个单例bean，它可能已经被容器初始化了。)所有引用最终都是对另一个对象的引用。范围和验证取决于您是通过bean还是父属性指定其他对象的ID或名称。

通过标记的bean属性指定目标bean是最通用的形式，允许在同一容器或父容器中创建对任何bean的引用，而不管它是否在同一XML文件中。bean属性的值可以与目标bean的id属性相同，也可以与目标bean的name属性中的某个值相同。下面的例子展示了如何使用ref元素:

```xml
<ref bean="someBean"/>
```

通过父属性指定目标bean将创建对当前容器父容器中的bean的引用。父属性的值可以与目标bean的id属性或目标bean的name属性中的一个值相同。目标bean必须位于当前bean的父容器中。当您有一个容器层次结构，并且您想用与父bean同名的代理将现有bean包装在父容器中时，您应该主要使用这个bean引用变体。下面的两个清单显示了如何使用父属性:

```xml
<!-- in the parent context -->
<bean id="accountService" class="com.something.SimpleAccountService">
    <!-- insert dependencies as required as here -->
</bean>
```

```xml
<!-- in the child (descendant) context -->
<bean id="accountService" <!-- bean name is the same as the parent bean -->
    class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="target">
        <ref parent="accountService"/> <!-- notice how we refer to the parent bean -->
    </property>
    <!-- insert other configuration and dependencies as required here -->
</bean>
```

```
4.0 bean XSD不再支持ref元素的local属性，因为它不再提供常规bean引用之上的值。升级到4.0模式时，将现有的ref本地引用更改为ref bean。
```

##### Inner Beans

或元素中的元素定义了一个内部bean，如下所示:

```xml
<bean id="outer" class="...">
    <!-- instead of using a reference to a target bean, simply define the target bean inline -->
    <property name="target">
        <bean class="com.example.Person"> <!-- this is the inner bean -->
            <property name="name" value="Fiona Apple"/>
            <property name="age" value="25"/>
        </bean>
    </property>
</bean>
```

内部bean定义不需要定义ID或名称。如果指定了该值，容器将不使用该值作为标识符。容器在创建时也会忽略范围标志，因为内部bean总是匿名的，并且总是与外部bean一起创建的。不可能单独访问内部bean，也不可能将它们注入协作bean中，而不是注入封闭bean中。

作为一种极端情况，可以从自定义范围接收销毁回调——例如，对于单例bean中包含的请求范围内的内部bean。内部bean实例的创建与包含它的bean绑定在一起，但是销毁回调让它参与请求范围的生命周期。这不是常见的情况。内部bean通常只是共享包含它们的bean的范围。

##### Collections

<list/>, <set/>, <map/>, and <props/>元素分别设置Java集合类型list、set、map和properties的属性和参数。下面的例子展示了如何使用它们:

```xml
<bean id="moreComplexObject" class="example.ComplexObject">
    <!-- results in a setAdminEmails(java.util.Properties) call -->
    <property name="adminEmails">
        <props>
            <prop key="administrator">administrator@example.org</prop>
            <prop key="support">support@example.org</prop>
            <prop key="development">development@example.org</prop>
        </props>
    </property>
    <!-- results in a setSomeList(java.util.List) call -->
    <property name="someList">
        <list>
            <value>a list element followed by a reference</value>
            <ref bean="myDataSource" />
        </list>
    </property>
    <!-- results in a setSomeMap(java.util.Map) call -->
    <property name="someMap">
        <map>
            <entry key="an entry" value="just some string"/>
            <entry key ="a ref" value-ref="myDataSource"/>
        </map>
    </property>
    <!-- results in a setSomeSet(java.util.Set) call -->
    <property name="someSet">
        <set>
            <value>just some string</value>
            <ref bean="myDataSource" />
        </set>
    </property>
</bean>
```

map键或值或set值的值也可以是以下元素中的任何一个:

```
bean | ref | idref | list | set | map | props | value | null
```

**Collection Merging**

Spring容器还支持合并集合。应用程序开发人员可以定义父元素<list/>`, `<map/>`, `<set/>` or `<props/>并让子元素<list/>`, `<map/>`, `<set/>` or `<props/>继承和覆盖父元素集合中的值。也就是说，子集合的值是父集合和子集合的元素合并的结果，子集合的元素覆盖父集合中指定的值。

关于合并的这一节讨论父-子bean机制。不熟悉父bean和子bean定义的读者可能希望在继续之前阅读相关部分。

下面的例子演示了集合合并:

```xml
<beans>
    <bean id="parent" abstract="true" class="example.ComplexObject">
        <property name="adminEmails">
            <props>
                <prop key="administrator">administrator@example.com</prop>
                <prop key="support">support@example.com</prop>
            </props>
        </property>
    </bean>
    <bean id="child" parent="parent">
        <property name="adminEmails">
            <!-- the merge is specified on the child collection definition -->
            <props merge="true">
                <prop key="sales">sales@example.com</prop>
                <prop key="support">support@example.co.uk</prop>
            </props>
        </property>
    </bean>
<beans>
```

注意在子bean定义的adminEmails属性的元素上使用了merge=true属性。当容器解析并实例化子bean时，结果实例有一个adminEmails Properties集合，该集合包含将子bean的adminEmails集合与父bean的adminEmails集合合并的结果。下面的清单显示了结果:

```
administrator=administrator@example.com
sales=sales@example.com
support=support@example.co.uk
```

子属性集合的值集继承了父属性的所有属性元素，并且子属性的支持值覆盖了父集合中的值。

这种合并行为同样适用于、和集合类型。在元素的特定情况下，将维护与list集合类型(即值的有序集合的概念)相关联的语义。父列表的值在所有子列表的值之前。对于Map、Set和Properties集合类型，不存在排序。因此，对于容器内部使用的关联Map、Set和Properties实现类型的集合类型，没有有效的排序语义。

**Limitations of Collection Merging**

您不能合并不同的集合类型(例如Map和List)。如果您试图这样做，则抛出一个适当的Exception。必须在较低的继承子定义上指定merge属性。在父集合定义上指定merge属性是多余的，并且不会导致所需的合并。

**Strongly-typed collection**

通过在Java 5中引入泛型类型，您可以使用强类型集合。也就是说，可以声明一个Collection类型，使它只能包含(例如)String元素。如果使用Spring将强类型Collection依赖注入到bean中，那么可以利用Spring的类型转换支持，在将强类型Collection实例的元素添加到Collection之前，将其转换为适当的类型。下面的Java类和bean定义展示了如何做到这一点:

```java
public class SomeClass {

    private Map<String, Float> accounts;

    public void setAccounts(Map<String, Float> accounts) {
        this.accounts = accounts;
    }
}
```

```xml
<beans>
    <bean id="something" class="x.y.SomeClass">
        <property name="accounts">
            <map>
                <entry key="one" value="9.99"/>
                <entry key="two" value="2.75"/>
                <entry key="six" value="3.99"/>
            </map>
        </property>
    </bean>
</beans>
```

当准备注入something bean的accounts属性时，关于强类型Map的元素类型的泛型信息可以通过反射获得。因此，Spring的类型转换基础结构将各种值元素识别为Float类型，并将字符串值(9.99、2.75和3.99)转换为实际的Float类型。

##### Null and Empty String Values

Spring将属性等的空参数视为空字符串。以下基于xml的配置元数据片段将email属性设置为空String值("")。

```xml
<bean class="ExampleBean">
    <property name="email" value=""/>
</bean>
```

上述示例相当于以下Java代码:

```java
exampleBean.setEmail("");
```

元素处理空值。下面的清单显示了一个示例:

```xml
<bean class="ExampleBean">
    <property name="email">
        <null/>
    </property>
</bean>
```

上述配置相当于以下Java代码:

```java
exampleBean.setEmail(null);
```

##### XML Shortcut with the p-namespace

p-namespace允许您使用bean元素的属性(而不是嵌套的元素)来描述协作bean的属性值，或者两者都使用。

Spring支持基于XML Schema定义的名称空间的可扩展配置格式。本章讨论的bean配置格式是在XML Schema文档中定义的。但是，p-namespace并没有在XSD文件中定义，它只存在于Spring的核心中。

下面的例子显示了两个XML片段(第一个使用标准XML格式，第二个使用p-namespace)，它们解析到相同的结果:

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean name="classic" class="com.example.ExampleBean">
        <property name="email" value="someone@somewhere.com"/>
    </bean>

    <bean name="p-namespace" class="com.example.ExampleBean"
        p:email="someone@somewhere.com"/>
</beans>
```

该示例显示了p-namespace中的一个属性，该属性在bean定义中称为email。这告诉Spring包含一个属性声明。如前所述，p-namespace没有模式定义，因此可以将属性的名称设置为属性名。

下一个示例包括另外两个bean定义，它们都有对另一个bean的引用:

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean name="john-classic" class="com.example.Person">
        <property name="name" value="John Doe"/>
        <property name="spouse" ref="jane"/>
    </bean>

    <bean name="john-modern"
        class="com.example.Person"
        p:name="John Doe"
        p:spouse-ref="jane"/>

    <bean name="jane" class="com.example.Person">
        <property name="name" value="Jane Doe"/>
    </bean>
</beans>
```

这个示例不仅包含使用p-namespace的属性值，而且还使用一种特殊的格式来声明属性引用。第一个bean定义使用来创建从bean john到bean jane的引用，第二个bean定义使用p:spouse-ref="jane"作为属性来完成完全相同的工作。在本例中，spouse是属性名，而-ref部分表示这不是一个直接值，而是对另一个bean的引用。

p-namespace不像标准XML格式那样灵活。例如，声明属性引用的格式与以Ref结尾的属性冲突，而标准XML格式不会。我们建议您谨慎地选择您的方法，并与您的团队成员进行沟通，以避免生成同时使用所有三种方法的XML文档。

##### XML Shortcut with the c-namespace

与带有p-namespace的XML快捷方式类似，Spring 3.1中引入的c-namespace允许内联属性来配置构造函数参数，而不是嵌套的构造函数-参数元素。

下面的例子使用c: namespace来做与基于构造函数的依赖注入相同的事情:

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:c="http://www.springframework.org/schema/c"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="beanTwo" class="x.y.ThingTwo"/>
    <bean id="beanThree" class="x.y.ThingThree"/>

    <!-- traditional declaration with optional argument names -->
    <bean id="beanOne" class="x.y.ThingOne">
        <constructor-arg name="thingTwo" ref="beanTwo"/>
        <constructor-arg name="thingThree" ref="beanThree"/>
        <constructor-arg name="email" value="something@somewhere.com"/>
    </bean>

    <!-- c-namespace declaration with argument names -->
    <bean id="beanOne" class="x.y.ThingOne" c:thingTwo-ref="beanTwo"
        c:thingThree-ref="beanThree" c:email="something@somewhere.com"/>

</beans>
```

命名空间使用与p: one (bean引用的末尾-ref)相同的约定，通过名称设置构造函数参数。类似地，它需要在XML文件中声明，即使它没有在XSD模式中定义(它存在于Spring核心中)。

对于构造函数参数名不可用的罕见情况(通常是在没有调试信息的情况下编译的字节码)，你可以使用回退参数索引，如下所示:

```xml
<!-- c-namespace index declaration -->
<bean id="beanOne" class="x.y.ThingOne" c:_0-ref="beanTwo" c:_1-ref="beanThree"
    c:_2="something@somewhere.com"/>
```

由于XML语法的原因，索引表示法要求出现前导_，因为XML属性名不能以数字开头(尽管有些ide允许)。对于元素也可以使用相应的索引表示法，但不常用，因为声明的简单顺序通常就足够了。

在实践中，构造函数解析机制在匹配参数方面非常有效，所以除非真的需要，否则我们建议在整个配置中使用名称表示法。

##### Compound Property Names

在设置bean属性时，可以使用复合或嵌套属性名，只要路径的所有组件(最终属性名除外)不为空。考虑以下bean定义:

```xml
<bean id="something" class="things.ThingOne">
    <property name="fred.bob.sammy" value="123" />
</bean>
```

这个bean有一个fred属性，它有一个bob属性，它有一个sammy属性，最后一个sammy属性被设置为123。为了使其工作，在构造bean之后，某物的fred属性和fred的bob属性不能为空。否则，抛出NullPointerException。

#### 1.4.3. Using `depends-on`

如果一个bean是另一个bean的依赖项，这通常意味着一个bean被设置为另一个bean的属性。通常使用基于xml的配置元数据中的元素来实现这一点。但是，有时bean之间的依赖关系不是那么直接。例如，当需要触发类中的静态初始化器时，例如数据库驱动程序注册时。依赖属性可以显式地在使用此元素的bean初始化之前强制初始化一个或多个bean。下面的例子使用depends-on属性来表示对单个bean的依赖:

```xml
<bean id="beanOne" class="ExampleBean" depends-on="manager"/>
<bean id="manager" class="ManagerBean" />
```

要表示对多个bean的依赖，请提供一个bean名列表作为依赖属性的值(逗号、空格和分号是有效的分隔符):

```xml
<bean id="beanOne" class="ExampleBean" depends-on="manager,accountDao">
    <property name="manager" ref="manager" />
</bean>

<bean id="manager" class="ManagerBean" />
<bean id="accountDao" class="x.y.jdbc.JdbcAccountDao" />
```

depends-on属性可以指定初始化时的依赖项，在单例bean的情况下，还可以指定相应的销毁时依赖项。定义了与给定bean的依赖关系的依赖bean首先被销毁，然后是给定bean本身被销毁。因此，依赖还可以控制关机顺序。

#### 1.4.4. Lazy-initialized Beans

默认情况下，ApplicationContext实现会在初始化过程中创建和配置所有单例bean。通常，这种预实例化是可取的，因为配置或周围环境中的错误是立即发现的，而不是几小时甚至几天后才发现的。当这种行为不可取时，您可以通过将bean定义标记为惰性初始化来防止单例bean的预实例化。延迟初始化bean告诉IoC容器在第一次请求时创建bean实例，而不是在启动时。

在XML中，这种行为由元素上的lazy-init属性控制，如下面的例子所示:

```xml
<bean id="lazy" class="com.something.ExpensiveToCreateBean" lazy-init="true"/>
<bean name="not.lazy" class="com.something.AnotherBean"/>
```

当上述配置被ApplicationContext使用时，当ApplicationContext启动时，惰性bean不会被急切地预实例化，而不是。惰性bean被急切地预先实例化。

然而，当一个惰性初始化的bean是一个非惰性初始化的单例bean的依赖项时，ApplicationContext会在启动时创建惰性初始化的bean，因为它必须满足单例bean的依赖项。惰性初始化的bean被注入到其他未惰性初始化的单例bean中。

你也可以通过在元素上使用default-lazy-init属性来控制容器级的惰性初始化，如下面的例子所示:

```xml
<beans default-lazy-init="true">
    <!-- no beans will be pre-instantiated... -->
</beans>
```

#### 1.4.5. Autowiring Collaborators

Spring容器可以自动装配协作bean之间的关系。通过检查ApplicationContext的内容，您可以让Spring自动为您的bean解析协作者(其他bean)。自动装配具有以下优点:

您不仅可以控制要插入到从特定bean定义创建的对象中的各种依赖项和配置值，还可以控制从特定bean定义创建的对象的范围。这种方法强大而灵活，因为您可以通过配置选择创建的对象的范围，而不必在Java类级别的对象范围内进行烘烤。可以将bean定义为部署在多个作用域中之一。Spring框架支持6个作用域，其中4个只有当你使用web感知的ApplicationContext时才可用。您还可以创建自定义范围。

支持的作用域如下表所示:

| **Scope**                                                    | **Description**                                              |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [singleton](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-singleton) | (Default) Scopes a single bean definition to a single object instance for each Spring IoC container. |
| [prototype](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-prototype) | Scopes a single bean definition to any number of object instances. |
| [request](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-request) | Scopes a single bean definition to the lifecycle of a single HTTP request. That is, each HTTP request has its own instance of a bean created off the back of a single bean definition. Only valid in the context of a web-aware Spring `ApplicationContext`. |
| [session](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-session) | Scopes a single bean definition to the lifecycle of an HTTP `Session`. Only valid in the context of a web-aware Spring `ApplicationContext`. |
| [application](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-application) | Scopes a single bean definition to the lifecycle of a `ServletContext`. Only valid in the context of a web-aware Spring `ApplicationContext`. |
| [websocket](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#websocket-stomp-websocket-scope) | Scopes a single bean definition to the lifecycle of a `WebSocket`. Only valid in the context of a web-aware Spring `ApplicationContext`. |

从spring3.0开始，线程范围是可用的，但默认情况下没有注册。有关更多信息，请参见SimpleThreadScope的文档。有关如何注册此或任何其他自定义作用域的说明，请参见使用自定义作用域。

####  1.5.1. The Singleton Scope

只管理一个单例bean的一个共享实例，所有对具有一个或多个ID匹配该bean定义的bean的请求都会导致Spring容器返回该特定的bean实例。

换句话说，当您定义一个bean定义并将其限定为一个单例对象时，Spring IoC容器恰好创建该bean定义定义的对象的一个实例。此单一实例存储在此类单例bean的缓存中，对该命名bean的所有后续请求和引用都返回缓存的对象。下面的图片显示了单例作用域是如何工作的:

![](C:\Users\dingbq\Desktop\spring文档\singleton.png)

Spring的单例bean概念不同于四人组模式书中定义的单例模式。GoF单例对对象的作用域进行了硬编码，这样每个ClassLoader只创建一个特定类的实例。Spring单例对象的作用域最好描述为每个容器和每个bean。这意味着，如果您在单个Spring容器中为特定类定义一个bean，那么Spring容器将创建由该bean定义的类的一个且仅一个实例。单例作用域是Spring的默认作用域。要在XML中将一个bean定义为一个单例，你可以如下所示定义一个bean:

```xml
<bean id="accountService" class="com.something.DefaultAccountService"/>

<!-- the following is equivalent, though redundant (singleton scope is the default) -->
<bean id="accountService" class="com.something.DefaultAccountService" scope="singleton"/>
```

####  1.5.2. The Prototype Scope

bean部署的非单例原型范围导致每次对特定bean发出请求时都创建一个新的bean实例。也就是说，该bean被注入到另一个bean中，或者通过容器上的getBean()方法调用请求它。作为一个规则，您应该对所有有状态bean使用原型作用域，而对无状态bean使用单例作用域。

下图说明了Spring原型的作用域:

![](C:\Users\dingbq\Desktop\spring文档\prototype.png)

数据访问对象(DAO)通常不被配置为原型，因为典型的DAO不保存任何会话状态。我们更容易重用单例图的核心。)

下面的例子用XML将bean定义为原型:

```xml
<bean id="accountService" class="com.something.DefaultAccountService" scope="prototype"/>
```

与其他作用域相比，Spring不管理原型bean的完整生命周期。容器实例化、配置和组装一个原型对象，并将其交给客户端，不再有该原型实例的进一步记录。因此，尽管对所有对象调用初始化生命周期回调方法而不考虑范围，但在原型的情况下，不调用配置的销毁生命周期回调。客户端代码必须清理原型作用域的对象，并释放原型bean持有的昂贵资源。要让Spring容器释放由原型作用域bean持有的资源，请尝试使用自定义bean后处理程序，它保存了对需要清理的bean的引用。

在某些方面，Spring容器相对于原型作用域bean的角色是Java new操作符的替代。超过这个时间点的所有生命周期管理都必须由客户端处理。(有关Spring容器中bean生命周期的详细信息，请参见生命周期回调。)

#### 1.5.3. Singleton Beans with Prototype-bean Dependencies

当您使用依赖于原型bean的单例作用域bean时，请注意依赖项是在实例化时解析的。因此，如果您将一个原型作用域的bean注入到单例作用域的bean中，那么将实例化一个新的原型bean，然后将依赖注入到单例bean中。原型实例是提供给单例作用域bean的唯一实例。

然而，假设您希望单例作用域bean在运行时反复获得原型作用域bean的一个新实例。不能将原型作用域的bean依赖注入到单例bean中，因为这种注入只发生一次，即Spring容器实例化单例bean并解析并注入它的依赖项时。如果你在运行时不止一次需要一个原型bean的新实例，请参阅方法注入

#### 1.5.4. Request, Session, Application, and WebSocket Scopes

request、session、application和websocket作用域只有在你使用web感知的Spring ApplicationContext实现(比如XmlWebApplicationContext)时才可用。如果您将这些作用域与常规的Spring IoC容器(如ClassPathXmlApplicationContext)一起使用，将抛出一个抱怨未知bean作用域的IllegalStateException。

##### Initial Web Configuration

为了在请求、会话、应用程序和websocket级别(web作用域bean)支持bean的作用域，在定义bean之前需要进行一些次要的初始配置。(标准作用域:singleton和prototype不需要这个初始设置。)

如何完成这个初始设置取决于特定的Servlet环境。

如果您在Spring Web MVC中访问作用域bean，实际上是在Spring DispatcherServlet处理的请求中访问，那么不需要任何特殊设置。DispatcherServlet已经公开了所有相关的状态。

如果你使用Servlet 2.5 web容器，在Spring的DispatcherServlet之外处理请求(例如，当使用JSF或Struts时)，你需要注册org.springframework.web.context.request.RequestContextListener ServletRequestListener。对于Servlet 3.0+，这可以通过使用WebApplicationInitializer接口以编程方式完成。或者，对于较旧的容器，在web应用程序的web.xml文件中添加以下声明:

```xml
<web-app>
    ...
    <listener>
        <listener-class>
            org.springframework.web.context.request.RequestContextListener
        </listener-class>
    </listener>
    ...
</web-app>
```

另外，如果您的侦听器设置有问题，可以考虑使用Spring的RequestContextFilter。过滤器映射取决于周围的web应用程序配置，因此您必须适当地更改它。下面的清单显示了一个web应用程序的过滤器部分:

```xml
<web-app>
    ...
    <filter>
        <filter-name>requestContextFilter</filter-name>
        <filter-class>org.springframework.web.filter.RequestContextFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>requestContextFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    ...
</web-app>
```

DispatcherServlet、RequestContextListener和RequestContextFilter都做同样的事情，即将HTTP请求对象绑定到服务该请求的线程。这使得请求作用域和会话作用域的bean在调用链的更下游可用。

#####  Request scope

考虑以下bean定义的XML配置:

```xml
<bean id="loginAction" class="com.something.LoginAction" scope="request"/>
```

Spring容器通过为每个HTTP请求使用LoginAction bean定义来创建LoginAction bean的新实例。也就是说，loginAction bean的作用域在HTTP请求级别。您可以随心所欲地更改已创建实例的内部状态，因为从相同的loginAction bean定义创建的其他实例不会看到这些状态更改。它们特别适合个人的要求。当请求完成处理后，将丢弃作用域为请求的bean。

当使用注释驱动组件或Java配置时，可以使用@RequestScope注释将组件分配给请求范围。下面的例子展示了如何做到这一点:

```java
@RequestScope
@Component
public class LoginAction {
    // ...
}
```

#####  Session Scope

考虑以下bean定义的XML配置:

```xml
<bean id="userPreferences" class="com.something.UserPreferences" scope="session"/>
```

Spring容器通过在单个HTTP会话的生命周期中使用UserPreferences bean定义来创建UserPreferences bean的新实例。换句话说，userPreferences bean有效地限定在HTTP Session级别。与请求范围内bean一样,你可以改变内部状态的实例创建尽可能多的你想要的,知道其他HTTP会话实例也使用相同的实例创建userPreferences bean定义看不到这些变化状态,因为他们是特定于一个单独的HTTP会话。当最终丢弃HTTP会话时，作用域为该特定HTTP会话的bean也将被丢弃。

在使用注释驱动组件或Java配置时，可以使用@SessionScope注释将组件分配给会话作用域。

```java
@SessionScope
@Component
public class UserPreferences {
    // ...
}
```

##### Application Scope

考虑以下bean定义的XML配置:

```xml
<bean id="appPreferences" class="com.something.AppPreferences" scope="application"/>
```

Spring容器通过对整个web应用程序使用一次AppPreferences bean定义来创建一个AppPreferences bean的新实例。也就是说，appPreferences bean的作用域在ServletContext级别，并存储为常规ServletContext属性。这有点类似于弹簧单例bean,但在两个重要方面不同:它是一个单例每ServletContext不是每春天ApplicationContext的(可能有几个在任何给定的web应用程序),它实际上是暴露,因此可见ServletContext属性。

当使用注释驱动组件或Java配置时，您可以使用@ApplicationScope注释将组件分配给应用程序范围。下面的例子展示了如何做到这一点:

```java
@ApplicationScope
@Component
public class AppPreferences {
    // ...
}
```

#####  Scoped Beans as Dependencies

Spring IoC容器不仅管理对象(bean)的实例化，还管理协作者(或依赖项)的连接。如果您想将(例如)一个HTTP请求作用域的bean注入到另一个长期作用域的bean中，您可以选择注入一个AOP代理来代替这个作用域bean。也就是说，您需要注入一个代理对象，该对象公开与作用域对象相同的公共接口，但也可以从相关作用域(如HTTP请求)检索实际目标对象，并对实际对象进行委托方法调用。

```
您也可以在作用域为单例的bean之间使用，然后引用通过一个可序列化的中间代理，因此能够在反序列化时重新获得目标单例bean。

当针对作用域原型bean声明时，对共享代理的每个方法调用都会导致创建一个新的目标实例，然后将调用转发给该实例。

此外，作用域代理并不是以生命周期安全的方式从较短的作用域访问bean的唯一方法。你也可以声明你的注射点(也就是构造函数或setter参数或autowired的字段)作为ObjectFactory < MyTargetBean >,允许getObject()调用来检索当前实例对需求每次需要——没有分别持有实例或存储它。

作为扩展的变体，您可以声明ObjectProvider，它提供了几个额外的访问变体，包括getIfAvailable和getIfUnique。

该方法的JSR-330变体称为Provider，它与Provider声明以及每次检索尝试对应的get()调用一起使用。关于JSR-330的更多详细信息，请参阅这里。
```

以下示例中的配置只有一行，但是理解其中的“为什么”和“如何”非常重要:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">

    <!-- an HTTP Session-scoped bean exposed as a proxy -->
    <bean id="userPreferences" class="com.something.UserPreferences" scope="session">
        <!-- instructs the container to proxy the surrounding bean -->
        <aop:scoped-proxy/> 
    </bean>

    <!-- a singleton-scoped bean injected with a proxy to the above bean -->
    <bean id="userService" class="com.something.SimpleUserService">
        <!-- a reference to the proxied userPreferences bean -->
        <property name="userPreferences" ref="userPreferences"/>
    </bean>
</beans>
```

要创建这样的代理，需要将一个子元素插入到一个scoped bean定义中(请参阅选择要创建的代理类型和基于XML模式的配置)。为什么在请求、会话和定制作用域级别定义bean需要元素?考虑以下单例bean定义，并将其与您需要为上述范围定义的内容进行对比(注意以下userPreferences bean定义是不完整的):

```xml
<bean id="userPreferences" class="com.something.UserPreferences" scope="session"/>

<bean id="userManager" class="com.something.UserManager">
    <property name="userPreferences" ref="userPreferences"/>
</bean>
```

在前面的示例中，单例bean (userManager)被注入一个对HTTP会话范围bean (userPreferences)的引用。这里突出的一点是userManager bean是一个单例:每个容器只实例化它一次，并且它的依赖项(在本例中只有一个，即userPreferences bean)也只被注入一次。这意味着userManager bean只对完全相同的userPreferences对象(即最初注入它的对象)进行操作。

当您将一个较短作用域的bean注入到较长作用域的bean中时，这不是您想要的行为(例如，将一个HTTP会话作用域的协作bean作为依赖项注入到单例bean中)。相反，您需要一个userManager对象，并且在HTTP会话的生命周期内，您需要一个特定于HTTP会话的userPreferences对象。因此，容器创建一个对象，该对象公开与UserPreferences类完全相同的公共接口(理想情况下，该对象是UserPreferences实例)，它可以从作用域机制(HTTP请求、会话等)获取真正的UserPreferences对象。容器将这个代理对象注入到userManager bean中，该bean并不知道这个UserPreferences引用是一个代理。在本例中，当UserManager实例调用依赖注入的UserPreferences对象上的方法时，它实际上是在调用代理上的方法。然后代理从HTTP Session(在本例中)获取真实的UserPreferences对象，并将方法调用委托给检索到的真实UserPreferences对象。

因此，在将请求和会话范围的bean注入协作对象时，您需要以下(正确和完整的)配置，如下面的示例所示:

```xml
<bean id="userPreferences" class="com.something.UserPreferences" scope="session">
    <aop:scoped-proxy/>
</bean>

<bean id="userManager" class="com.something.UserManager">
    <property name="userPreferences" ref="userPreferences"/>
</bean>
```

**Choosing the Type of Proxy to Create**

默认情况下，当Spring容器为用元素标记的bean创建代理时，将创建一个基于cglib的类代理。

CGLIB代理只拦截公共方法调用!不要在这样的代理上调用非公共方法。它们没有被委托给实际的作用域目标对象。

或者，通过为元素的proxy-target-class属性的值指定false，您可以配置Spring容器为这种作用域bean创建基于JDK接口的标准代理。使用基于JDK接口的代理意味着您不需要在应用程序类路径中添加其他库来影响这种代理。然而，这也意味着作用域bean的类必须实现至少一个接口，所有被注入该作用域bean的协作者必须通过它的一个接口引用该bean。下面的例子显示了一个基于接口的代理:

```xml
<!-- DefaultUserPreferences implements the UserPreferences interface -->
<bean id="userPreferences" class="com.stuff.DefaultUserPreferences" scope="session">
    <aop:scoped-proxy proxy-target-class="false"/>
</bean>

<bean id="userManager" class="com.stuff.UserManager">
    <property name="userPreferences" ref="userPreferences"/>
</bean>
```

有关选择基于类的代理或基于接口的代理的详细信息，请参见代理机制。

#### 1.5.5. Custom Scopes

bean范围机制是可扩展的。您可以定义自己的作用域，甚至可以重新定义现有的作用域，尽管后者被认为是不好的做法，并且您不能覆盖内置的单例作用域和原型作用域。

##### Creating a Custom Scope

要将您的自定义作用域集成到Spring容器中，您需要实现org.springframework.beans.factory.config.Scope接口，该接口将在本节中描述。要了解如何实现自己的作用域，请参阅随Spring框架本身提供的Scope实现和Scope javadoc，它将更详细地解释需要实现的方法。

Scope接口有四个方法，用于从作用域获取对象、从作用域移除对象并销毁它们。

例如，会话作用域实现返回会话作用域bean(如果它不存在，该方法将bean绑定到会话以备将来引用后返回该bean的新实例)。下面的方法从底层作用域返回对象:

```java
Object get(String name, ObjectFactory<?> objectFactory)
```

例如，会话作用域实现从底层会话中删除会话作用域bean。应该返回该对象，但如果没有找到指定名称的对象，则可以返回null。以下方法将对象从基础作用域中移除:

```java
Object remove(String name)
```

以下方法注册了一个回调函数，当该范围被撤销时，或者当该范围中的指定对象被撤销时，该回调函数应该被调用:

```java
void registerDestructionCallback(String name, Runnable destructionCallback)
```

有关销毁回调的更多信息，请参见javadoc或Spring作用域实现。

以下方法获取底层作用域的会话标识符:

```java
String getConversationId()
```

这个标识符对于每个作用域都是不同的。对于会话范围的实现，此标识符可以是会话标识符。

##### Using a Custom Scope

在编写和测试一个或多个自定义Scope实现之后，您需要让Spring容器知道您的新作用域。下面的方法是向Spring容器注册一个新的Scope的中心方法:

```java
void registerScope(String scopeName, Scope scope);
```

此方法在ConfigurableBeanFactory接口上声明，该接口可通过Spring附带的大多数具体ApplicationContext实现上的BeanFactory属性获得。

registerScope(..)方法的第一个参数是与作用域关联的唯一名称。Spring容器本身中的此类名称的例子有singleton和prototype。registerScope(..)方法的第二个参数是您希望注册和使用的自定义Scope实现的实际实例。

假设您编写了自定义Scope实现，然后如下一个示例所示注册它。

```
下一个示例使用SimpleThreadScope，它包含在Spring中，但默认情况下没有注册。对于您自己的自定义Scope实现，说明将是相同的。
```

```java
Scope threadScope = new SimpleThreadScope();
beanFactory.registerScope("thread", threadScope);
```

然后，您可以创建符合自定义Scope范围规则的bean定义，如下所示:

```xml
<bean id="..." class="..." scope="thread">
```

有了自定义的Scope实现，您就不限于该范围的编程注册。您也可以通过使用CustomScopeConfigurer类以声明的方式进行范围注册，如下面的示例所示:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">

    <bean class="org.springframework.beans.factory.config.CustomScopeConfigurer">
        <property name="scopes">
            <map>
                <entry key="thread">
                    <bean class="org.springframework.context.support.SimpleThreadScope"/>
                </entry>
            </map>
        </property>
    </bean>

    <bean id="thing2" class="x.y.Thing2" scope="thread">
        <property name="name" value="Rick"/>
        <aop:scoped-proxy/>
    </bean>

    <bean id="thing1" class="x.y.Thing1">
        <property name="thing2" ref="thing2"/>
    </bean>

</beans>
```

当您将放在FactoryBean实现的声明中时，作用域是工厂bean本身，而不是getObject()返回的对象。

###  1.6. Customizing the Nature of a Bean

Spring框架提供了许多接口，您可以使用它们来定制bean的性质。本节将它们分组如下:

- [Lifecycle Callbacks](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle)
- [`ApplicationContextAware` and `BeanNameAware`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-aware)
- [Other `Aware` Interfaces](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aware-list)

#### 1.6.1. Lifecycle Callbacks

为了与容器对bean生命周期的管理交互，您可以实现Spring InitializingBean和DisposableBean接口。容器对前者调用afterPropertiesSet()，对后者调用destroy()，让bean在初始化和销毁bean时执行某些操作。

```
JSR-250 @PostConstruct和@PreDestroy注释通常被认为是现代Spring应用程序中接收生命周期回调的最佳实践。使用这些注释意味着您的bean没有耦合到特定于spring的接口。详细信息请参见使用@PostConstruct和@PreDestroy。

如果您不想使用JSR-250注释，但仍然希望删除耦合，可以考虑init-method和destroy-method bean定义元数据。
```

在内部，Spring框架使用BeanPostProcessor实现来处理它可以找到的任何回调接口，并调用适当的方法。如果您需要自定义特性或Spring默认不提供的其他生命周期行为，您可以自己实现BeanPostProcessor，更多信息请参考[Container Extension Points](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-extension).

除了初始化和销毁回调，spring管理的对象也可以实现Lifecycle接口，以便这些对象可以参与由容器自己的生命周期驱动的启动和关闭过程。

本节描述生命周期回调接口。

#####  Initialization Callbacks

`org.springframework.beans.factory.InitializingBean`接口允许bean在容器设置了bean上的所有必要属性后执行初始化工作。InitializingBean接口指定了一个方法:

```java
void afterPropertiesSet() throws Exception;
```

我们建议您不要使用InitializingBean接口，因为它不必要地将代码耦合到Spring。另外，我们建议使用@PostConstruct注释或指定POJO初始化方法。对于基于xml的配置元数据，可以使用init-method属性指定具有空无参数签名的方法的名称。通过Java配置，您可以使用@Bean的initMethod属性。参见接收生命周期回调。考虑以下例子:

```xml
<bean id="exampleInitBean" class="examples.ExampleBean" init-method="init"/>
```

```java
public class ExampleBean {

    public void init() {
        // do some initialization work
    }
}
```

上面的示例与下面的示例(包含两个清单)的效果几乎完全相同:

```xml
<bean id="exampleInitBean" class="examples.AnotherExampleBean"/>
```

```
public class AnotherExampleBean implements InitializingBean {

    @Override
    public void afterPropertiesSet() {
        // do some initialization work
    }
}
```

但是，前面两个示例中的第一个没有将代码与Spring耦合。

#####  Destruction Callbacks

实现`org.springframework. factory. disposablebean`接口可以让一个bean在包含它的容器被销毁时获得一个回调。DisposableBean接口指定了一个方法:

```java
void destroy() throws Exception;
```

我们建议您不要使用DisposableBean回调接口，因为它不必要地将代码与Spring耦合在一起。另外，我们建议使用@PreDestroy注释或指定由bean定义支持的通用方法。使用基于xml的配置元数据，您可以在上使用destroy-method属性。在Java配置中，可以使用@Bean的destroyMethod属性。参见接收生命周期回调。考虑以下定义:

```xml
<bean id="exampleInitBean" class="examples.ExampleBean" destroy-method="cleanup"/>
```

```java
public class ExampleBean {

    public void cleanup() {
        // do some destruction work (like releasing pooled connections)
    }
}
```

上面的定义与下面的定义几乎完全相同:

```xml
<bean id="exampleInitBean" class="examples.AnotherExampleBean"/>
```

```java
public class AnotherExampleBean implements DisposableBean {

    @Override
    public void destroy() {
        // do some destruction work (like releasing pooled connections)
    }
}
```

但是，前面两个定义中的第一个没有将代码耦合到Spring。

您可以为元素的destroy-method属性指定一个特殊的(推断的)值，它指示Spring自动检测特定bean类上的公共关闭或关闭方法。(因此，任何实现java.lang.AutoCloseable或java.io.Closeable的类都会匹配。)您还可以在元素的Default - Destroy -method属性上设置这个特殊的(推断的)值，以便将此行为应用到整个bean集合(请参阅默认初始化和销毁方法)。注意，这是Java配置的默认行为。

##### Default Initialization and Destroy Methods

当您编写不使用特定于spring的InitializingBean和DisposableBean回调接口的初始化和销毁方法回调时，您通常会编写init()、initialize()、dispose()等名称的方法。理想情况下，生命周期回调方法的名称在整个项目中是标准化的，以便所有开发人员使用相同的方法名称并确保一致性。

您可以将Spring容器配置为“查找”已命名初始化并销毁每个bean上的回调方法名。这意味着，作为应用程序开发人员，您可以编写应用程序类并使用名为init()的初始化回调函数，而不必为每个bean定义配置init-method="init"属性。Spring IoC容器在创建bean时调用该方法(并且按照前面描述的标准生命周期回调契约)。该特性还强制初始化和销毁方法回调的一致命名约定。

假设初始化回调方法名为init()，销毁回调方法名为destroy()。你的类就像下面例子中的类:

```java
public class DefaultBlogService implements BlogService {

    private BlogDao blogDao;

    public void setBlogDao(BlogDao blogDao) {
        this.blogDao = blogDao;
    }

    // this is (unsurprisingly) the initialization callback method
    public void init() {
        if (this.blogDao == null) {
            throw new IllegalStateException("The [blogDao] property must be set.");
        }
    }
}
```

然后你可以像下面这样在bean中使用这个类:

```xml
<beans default-init-method="init">

    <bean id="blogService" class="com.something.DefaultBlogService">
        <property name="blogDao" ref="blogDao" />
    </bean>

</beans>
```

顶级元素属性上的default-init-method属性导致Spring IoC容器将bean类上名为init的方法识别为初始化方法回调。在创建和组装一个bean时，如果这个bean类有这样的方法，就会在适当的时候调用它。

通过在顶级元素上使用default-destroy-method属性，您可以类似地配置销毁方法回调(即在XML中)。

如果现有的bean类已经有了与约定不同的回调方法，那么您可以通过使用本身的init-method和destroy-method属性指定(即在XML中)方法名来覆盖默认值。

Spring容器保证在提供了包含所有依赖项的bean之后立即调用配置好的初始化回调。因此，对原始bean引用调用初始化回调，这意味着AOP拦截器等尚未应用于bean。首先完全创建目标bean，然后应用带有拦截器链的AOP代理(例如)。如果目标bean和代理是分开定义的，那么您的代码甚至可以绕过代理与原始目标bean交互。因此，将拦截器应用到init方法是不一致的，因为这样做会将目标bean的生命周期与其代理或拦截器耦合在一起，并且在代码直接与原始目标bean交互时留下奇怪的语义。

##### Combining Lifecycle Mechanisms

从spring2.5开始，你有三个选项来控制bean的生命周期行为:

- The [`InitializingBean`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle-initializingbean) and [`DisposableBean`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle-disposablebean) callback interfaces
- Custom `init()` and `destroy()` methods
- The [`@PostConstruct` and `@PreDestroy` annotations](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-postconstruct-and-predestroy-annotations). You can combine these mechanisms to control a given bean.

如果为一个bean配置了多个生命周期机制，并且每个机制都配置了不同的方法名，那么每个已配置的方法将按照本文后面列出的顺序运行。但是，如果为多个生命周期机制配置了相同的方法名——例如，为一个初始化方法配置init()——该方法将运行一次，如上一节所述。

为同一个bean配置的具有不同初始化方法的多个生命周期机制的调用如下:

1. Methods annotated with `@PostConstruct`
2. `afterPropertiesSet()` as defined by the `InitializingBean` callback interface
3. A custom configured `init()` method

Destroy方法的调用顺序相同:

1. Methods annotated with `@PreDestroy`
2. `destroy()` as defined by the `DisposableBean` callback interface
3. A custom configured `destroy()` method

##### Startup and Shutdown Callbacks

生命周期接口定义了任何有自己生命周期需求的对象的基本方法(例如启动和停止某些后台进程):

```java
public interface Lifecycle {

    void start();

    void stop();

    boolean isRunning();
}
```

任何spring管理的对象都可以实现Lifecycle接口。然后，当ApplicationContext本身接收到启动和停止信号(例如，对于运行时的停止/重启场景)时，它将这些调用级联到该上下文中定义的所有生命周期实现。它通过委托给LifecycleProcessor来做到这一点，如下所示:

```java
public interface LifecycleProcessor extends Lifecycle {

    void onRefresh();

    void onClose();
}
```

请注意，LifecycleProcessor本身就是生命周期接口的扩展。它还添加了另外两种方法来响应正在刷新和关闭的上下文。

```
请注意，常规的org.springframework.context.Lifecycle接口是显式启动和停止通知的简单约定，并不意味着在上下文刷新时自动启动。对于特定bean的自动启动(包括启动阶段)的细粒度控制，可以考虑实现org.springframework.context.SmartLifecycle。
另外，请注意，停止通知不能保证在销毁之前出现。在常规关闭时，在传播常规销毁回调之前，所有Lifecycle bean首先收到一个停止通知。但是，在上下文生命周期内的热刷新或停止刷新尝试时，只调用destroy方法。
```

启动和关闭调用的顺序可能很重要。如果任意两个对象之间存在“依赖”关系，依赖方在其依赖项之后启动，在其依赖项之前停止。然而，有时，直接依赖关系是未知的。您可能只知道某种类型的对象应该在另一种类型的对象之前启动。在这些情况下，SmartLifecycle接口定义了另一个选项，即在其超级接口phase上定义的getPhase()方法。phase接口的定义如下所示:

```java
public interface Phased {

    int getPhase();
}
```

下面的列表显示了SmartLifecycle接口的定义:

```java
public interface SmartLifecycle extends Lifecycle, Phased {

    boolean isAutoStartup();

    void stop(Runnable callback);
}
```

当启动时，具有最低阶段的对象首先启动。当停止时，则遵循相反的顺序。因此，一个实现SmartLifecycle的对象，其getPhase()方法返回Integer。MIN_VALUE将是最先开始和最后停止的值之一。在频谱的另一端，相位值为Integer。MAX_VALUE将指示对象应该最后启动，首先停止(可能是因为它依赖于其他正在运行的进程)。当考虑阶段值时，同样重要的是要知道任何“正常”生命周期对象的默认阶段是0，但它没有实现SmartLifecycle。因此，任何负相位值都表明对象应该在这些标准组件之前开始(并在它们之后停止)。对于任何正相位值，则相反。

SmartLifecycle定义的停止方法接受回调。任何实现都必须在实现的关闭过程完成后调用回调的run()方法。这在必要时支持异步关闭，因为LifecycleProcessor接口的默认实现DefaultLifecycleProcessor将等待每个阶段中的对象组的超时值来调用该回调。每个阶段的默认超时时间是30秒。您可以通过在上下文中定义一个名为lifecycleProcessor的bean来覆盖默认的生命周期处理器实例。如果你只想修改超时时间，定义以下内容就足够了:

```xml
<bean id="lifecycleProcessor" class="org.springframework.context.support.DefaultLifecycleProcessor">
    <!-- timeout value in milliseconds -->
    <property name="timeoutPerShutdownPhase" value="10000"/>
</bean>
```

如前所述，LifecycleProcessor接口还定义了用于刷新和关闭上下文的回调方法。后者驱动关机进程，就像显式调用了stop()一样，但它发生在上下文关闭时。另一方面，'refresh'回调启用了SmartLifecycle bean的另一个特性。当上下文被刷新时(在所有对象被实例化和初始化之后)，该回调将被调用。此时，默认的生命周期处理器会检查每个SmartLifecycle对象的isAutoStartup()方法返回的布尔值。如果为true，则该对象将在此点启动，而不是等待上下文的显式调用或它自己的start()方法(不同于上下文刷新，对于标准上下文实现，上下文启动不会自动发生)。如前所述，阶段值和任何“依赖”关系决定启动顺序。

#####  Shutting Down the Spring IoC Container Gracefully in Non-Web Applications

```
	
This section applies only to non-web applications. Spring’s web-based ApplicationContext implementations already have code in place to gracefully shut down the Spring IoC container when the relevant web application is shut down.
```

如果您在非web应用程序环境中使用Spring的IoC容器(例如，在富客户端桌面环境中)，请向JVM注册一个关机挂钩。这样做可以确保正常关闭，并调用单例bean上的相关destroy方法，以便释放所有资源。您仍然必须正确配置和实现这些销毁回调。

要注册一个shutdown钩子，调用在ConfigurableApplicationContext接口上声明的registerShutdownHook()方法，示例如下:

```java
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public final class Boot {

    public static void main(final String[] args) throws Exception {
        ConfigurableApplicationContext ctx = new ClassPathXmlApplicationContext("beans.xml");

        // add a shutdown hook for the above context...
        ctx.registerShutdownHook();

        // app runs here...

        // main method exits, hook is called prior to the app shutting down...
    }
}
```

####  1.6.2. `ApplicationContextAware` and `BeanNameAware`

当ApplicationContext创建一个实现org.springframework.context.ApplicationContextAware接口的对象实例时，该实例会提供对该ApplicationContext的引用。下面的清单显示了ApplicationContextAware接口的定义:

```java
public interface ApplicationContextAware {

    void setApplicationContext(ApplicationContext applicationContext) throws BeansException;
}
```

因此，bean可以通过程序操作创建它们的ApplicationContext，通过ApplicationContext接口，或者通过将引用转换到该接口的已知子类(例如ConfigurableApplicationContext，它公开了额外的功能)。一种用途是对其他bean进行编程检索。有时这种能力是有用的。但是，通常应该避免使用它，因为它将代码耦合到Spring，并且不遵循控制反转(Inversion of Control)样式，在这种样式中协作者作为属性提供给bean。ApplicationContext的其他方法提供了对文件资源的访问、发布应用程序事件和访问MessageSource。这些附加特性在ApplicationContext的附加功能中进行了描述。

自动装配是另一种获取ApplicationContext引用的方法。传统的构造函数和byType自动装配模式(如autowiring合作者所述)可以分别为构造函数参数或setter方法参数提供ApplicationContext类型的依赖关系。为了获得更多的灵活性，包括自动装配字段和多个参数方法的能力，可以使用基于注释的自动装配特性。如果您这样做，ApplicationContext将自动连接到一个字段、构造函数参数或方法参数，如果相关字段、构造函数或方法带有@Autowired注释，则该字段、构造函数或方法参数需要ApplicationContext类型。有关更多信息，请参见使用@Autowired。

当ApplicationContext创建一个实现org.springframework.beans.factory.BeanNameAware接口的类时，会为该类提供一个对其关联对象定义中定义的名称的引用。下面的清单显示了BeanNameAware接口的定义:

```java
public interface BeanNameAware {

    void setBeanName(String name) throws BeansException;
}
```

回调在填充普通bean属性之后，但在初始化回调(如InitializingBean、propertiesset之后或自定义初始化方法之前)之前被调用。

####  1.6.3. Other `Aware` Interfaces

除了ApplicationContextAware和BeanNameAware(前面讨论过)之外，Spring还提供了一系列Aware回调接口，让bean向容器表明它们需要特定的基础设施依赖关系。作为一般规则，名称表示依赖类型。下表总结了最重要的Aware接口:

| **Name**                       | **Injected Dependency**                                      | **Explained in…**                                            |
| ------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| ApplicationContextAware        | Declaring `ApplicationContext`.                              | [`ApplicationContextAware` and `BeanNameAware`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-aware) |
| ApplicationEventPublisherAware | Event publisher of the enclosing `ApplicationContext`.       | [Additional Capabilities of the `ApplicationContext`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#context-introduction) |
| BeanClassLoaderAware           | Class loader used to load the bean classes.                  | [Instantiating Beans](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-class) |
| BeanFactoryAware               | Declaring `BeanFactory`.                                     | [`ApplicationContextAware` and `BeanNameAware`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-aware) |
| BeanNameAware                  | Name of the declaring bean.                                  | [`ApplicationContextAware` and `BeanNameAware`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-aware) |
| LoadTimeWeaverAware            | Defined weaver for processing class definition at load time. | [Load-time Weaving with AspectJ in the Spring Framework](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-aj-ltw) |
| MessageSourceAware             | Configured strategy for resolving messages (with support for parametrization and internationalization). | [Additional Capabilities of the `ApplicationContext`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#context-introduction) |
| NotificationPublisherAware     | Spring JMX notification publisher.                           | [Notifications](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#jmx-notifications) |
| ResourceLoaderAware            | Configured loader for low-level access to resources.         | [Resources](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#resources) |
| ServletConfigAware             | Current `ServletConfig` the container runs in. Valid only in a web-aware Spring `ApplicationContext`. | [Spring MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc) |
| ServletContextAware            | Current `ServletContext` the container runs in. Valid only in a web-aware Spring `ApplicationContext`. | [Spring MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc) |

请再次注意，使用这些接口将您的代码绑定到Spring API，并且不遵循控制反转风格。因此，对于需要对容器进行编程访问的基础设施bean，我们建议使用它们。

###  1.7. Bean Definition Inheritance

bean定义可以包含大量配置信息，包括构造函数参数、属性值和特定于容器的信息，如初始化方法、静态工厂方法名，等等。子bean定义从父bean定义继承配置数据。子定义可以重写一些值或根据需要添加其他值。使用父bean和子bean定义可以节省大量的类型工作。实际上，这是一种模板形式。

如果您以编程方式使用ApplicationContext接口，则子bean定义由ChildBeanDefinition类表示。大多数用户不会在这个级别上使用它们。相反，它们在类(如ClassPathXmlApplicationContext)中以声明的方式配置bean定义。当您使用基于xml的配置元数据时，您可以通过使用父属性指定父bean作为该属性的值来指示子bean定义。下面的例子展示了如何做到这一点:

```xml
<bean id="inheritedTestBean" abstract="true"
        class="org.springframework.beans.TestBean">
    <property name="name" value="parent"/>
    <property name="age" value="1"/>
</bean>

<bean id="inheritsWithDifferentClass"
        class="org.springframework.beans.DerivedTestBean"
        parent="inheritedTestBean" init-method="initialize">  
    <property name="name" value="override"/>
    <!-- the age property value of 1 will be inherited from parent -->
</bean>
```

如果没有指定，则子bean定义使用父bean定义中的bean类，但也可以覆盖它。在后一种情况下，子bean类必须与父类兼容(也就是说，它必须接受父类的属性值)。

子bean定义继承父bean的作用域、构造函数参数值、属性值和方法覆盖，并具有添加新值的选项。您指定的任何范围、初始化方法、销毁方法或静态工厂方法设置都会覆盖相应的父设置。

其余的设置总是从子定义中获取:依赖、自动装配模式、依赖检查、单例模式和惰性初始化。

前面的示例通过使用抽象属性显式地将父bean定义标记为抽象。如果父bean定义没有指定一个类，则需要显式地将父bean定义标记为抽象，如下面的例子所示:

```xml
<bean id="inheritedTestBeanWithoutClass" abstract="true">
    <property name="name" value="parent"/>
    <property name="age" value="1"/>
</bean>

<bean id="inheritsWithClass" class="org.springframework.beans.DerivedTestBean"
        parent="inheritedTestBeanWithoutClass" init-method="initialize">
    <property name="name" value="override"/>
    <!-- age will inherit the value of 1 from the parent bean definition-->
</bean>
```

父bean不能单独实例化，因为它是不完整的，而且它也被显式地标记为抽象。当定义是抽象的时，它只能作为作为子定义的父定义的纯模板bean定义使用。试图单独使用这样一个抽象的父bean，通过引用它作为另一个bean的ref属性，或者使用父bean ID显式地调用getBean()将返回一个错误。类似地，容器的内部preInstantiateSingletons()方法忽略了定义为抽象的bean定义。

ApplicationContext默认预先实例化所有单例。因此,它是重要的(至少对单例bean),如果你有一个(父)bean定义你只打算使用作为模板,这个定义指定了一个类,您必须确保设置抽象属性为true,否则应用程序上下文会(试图)pre-instantiate抽象的bean。

###  1.8. Container Extension Points

通常，应用程序开发人员不需要继承ApplicationContext实现类的子类。相反，可以通过插入特殊集成接口的实现来扩展Spring IoC容器。接下来的几节将描述这些集成接口。

####  1.8.1. Customizing Beans by Using a `BeanPostProcessor`

BeanPostProcessor接口定义了可以实现的回调方法，以提供您自己的(或覆盖容器的默认)实例化逻辑、依赖关系解析逻辑等等。如果您希望在Spring容器完成bean的实例化、配置和初始化之后实现一些自定义逻辑，您可以插入一个或多个自定义BeanPostProcessor实现。

您可以配置多个BeanPostProcessor实例，并且可以通过设置order属性来控制这些BeanPostProcessor实例运行的顺序。只有当BeanPostProcessor实现了Ordered接口时，才能设置此属性。如果您编写自己的BeanPostProcessor，您也应该考虑实现Ordered接口。有关详细信息，请参见BeanPostProcessor和Ordered接口的javadoc。另请参阅有关BeanPostProcessor实例的程序化注册的说明。

```
BeanPostProcessor实例操作bean(或对象)实例。也就是说，Spring IoC容器实例化一个bean实例，然后BeanPostProcessor实例执行它们的工作。

BeanPostProcessor实例的作用域为每个容器。这只有在使用容器层次结构时才相关。如果您在一个容器中定义一个BeanPostProcessor，它只对该容器中的bean进行后处理。换句话说，在一个容器中定义的bean不会被另一个容器中定义的BeanPostProcessor进行后处理，即使两个容器都属于相同的层次结构。

要更改实际的bean定义(即定义bean的蓝图)，您需要使用BeanFactoryPostProcessor，如使用BeanFactoryPostProcessor定制配置元数据中所述。
```

beanpostprocessor接口恰好由两个回调方法组成。当这样一个类注册为后处理器的容器,每个容器创建bean实例,后处理器从容器之前得到一个回调容器初始化方法(如InitializingBean.afterPropertiesSet()或任何宣布init方法),在任何bean初始化回调之后。后处理器可以对bean实例执行任何操作，包括完全忽略回调。bean后处理程序通常检查回调接口，或者它可以用代理来包装bean。为了提供代理包装逻辑，一些Spring AOP基础设施类被实现为bean后处理器。

ApplicationContext自动检测在实现BeanPostProcessor接口的配置元数据中定义的任何bean。ApplicationContext将这些bean注册为后处理器，以便以后在创建bean时可以调用它们。Bean后处理程序可以像其他Bean一样部署在容器中。

注意,当宣布BeanPostProcessor @ bean配置类工厂方法,使用一个工厂方法的返回类型必须实现类本身或者至少org.springframework.beans.factory.config.BeanPostProcessor接口,明确指示后处理器的特性,bean。否则，ApplicationContext不能在完全创建它之前根据类型自动检测它。因为需要尽早实例化BeanPostProcessor，以便应用于上下文中其他bean的初始化，所以这种早期类型检测非常关键。

```
以编程方式注册BeanPostProcessor实例
虽然推荐的BeanPostProcessor注册方法是通过ApplicationContext自动检测(如前所述)，但是您可以通过使用addBeanPostProcessor方法以编程方式在ConfigurableBeanFactory上注册它们。当您需要在注册之前评估条件逻辑时，或者甚至在跨层次结构中的上下文复制bean后处理程序时，这将非常有用。但是请注意，以编程方式添加的BeanPostProcessor实例并不尊重Ordered接口。在这里，登记的顺序决定了执行的顺序。还请注意，以编程方式注册的BeanPostProcessor实例总是在通过自动检测注册的实例之前被处理，而不考虑任何显式排序。
```

```
                                BeanPostProcessor实例和AOP自动代理
实现BeanPostProcessor接口的类是特殊的，容器以不同的方式对待它们。作为ApplicationContext特殊启动阶段的一部分，所有BeanPostProcessor实例和它们直接引用的bean都在启动时实例化。接下来，所有BeanPostProcessor实例都以排序的方式注册，并应用于容器中的所有其他bean。因为AOP的自动代理是作为BeanPostProcessor本身实现的，所以BeanPostProcessor实例和它们直接引用的bean都没有资格进行自动代理，因此，它们没有被编入方面。
对于任何这样的bean，您应该看到一条信息日志消息:bean someBean不适合被所有BeanPostProcessor接口处理(例如:不适合自动代理)。
如果您通过使用autowiring或@Resource(这可能会退回到autowiring)将bean连接到BeanPostProcessor中，Spring可能会在搜索类型匹配依赖项候选时访问意料之外的bean，因此，使它们不适合进行自动代理或其他类型的bean后处理。例如，如果您有一个用@Resource注释的依赖项，其中字段或setter名称并不直接对应于bean声明的名称，并且没有使用name属性，那么Spring将访问其他bean以根据类型匹配它们。
```

下面的例子展示了如何在ApplicationContext中编写、注册和使用BeanPostProcessor实例。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       

#####  Example: Hello World, `BeanPostProcessor`-style

第一个示例演示了基本用法。该示例显示了一个自定义BeanPostProcessor实现，它在容器创建每个bean时调用toString()方法，并将结果字符串打印到系统控制台。 

下面的清单显示了自定义BeanPostProcessor实现类定义:

```java
package scripting;

import org.springframework.beans.factory.config.BeanPostProcessor;

public class InstantiationTracingBeanPostProcessor implements BeanPostProcessor {

    // simply return the instantiated bean as-is
    public Object postProcessBeforeInitialization(Object bean, String beanName) {
        return bean; // we could potentially return any object reference here...
    }

    public Object postProcessAfterInitialization(Object bean, String beanName) {
        System.out.println("Bean '" + beanName + "' created : " + bean.toString());
        return bean;
    }
}
```

下面的beans元素使用了InstantiationTracingBeanPostProcessor:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:lang="http://www.springframework.org/schema/lang"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/lang
        https://www.springframework.org/schema/lang/spring-lang.xsd">

    <lang:groovy id="messenger"
            script-source="classpath:org/springframework/scripting/groovy/Messenger.groovy">
        <lang:property name="message" value="Fiona Apple Is Just So Dreamy."/>
    </lang:groovy>

    <!--
    when the above bean (messenger) is instantiated, this custom
    BeanPostProcessor implementation will output the fact to the system console
    -->
    <bean class="scripting.InstantiationTracingBeanPostProcessor"/>

</beans>
```

注意InstantiationTracingBeanPostProcessor是如何定义的。它甚至没有名称，而且，因为它是一个bean，所以可以像对任何其他bean一样进行依赖注入。(前面的配置还定义了一个由Groovy脚本支持的bean。Spring动态语言支持在动态语言支持一章中有详细介绍。)

以下Java应用程序运行上述代码和配置:

```xml
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.scripting.Messenger;

public final class Boot {

    public static void main(final String[] args) throws Exception {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("scripting/beans.xml");
        Messenger messenger = ctx.getBean("messenger", Messenger.class);
        System.out.println(messenger);
    }

}
```

上述应用程序的输出类似如下:

```
Bean 'messenger' created : org.springframework.scripting.groovy.GroovyMessenger@272961
org.springframework.scripting.groovy.GroovyMessenger@272961
```

##### Example: The `AutowiredAnnotationBeanPostProcessor`

将回调接口或注释与自定义BeanPostProcessor实现结合使用是扩展Spring IoC容器的一种常见方法。一个例子是Spring的AutowiredAnnotationBeanPostProcessor -一个BeanPostProcessor实现，它随Spring分发和自动连接带注释的字段、setter方法和任意配置方法一起提供。

####  1.8.2. Customizing Configuration Metadata with a `BeanFactoryPostProcessor`

我们看到的下一个扩展点是`org.springframework.beans.factory.config.BeanFactoryPostProcessor`。这个接口的语义类似于BeanPostProcessor的语义，但有一个主要的区别:BeanFactoryPostProcessor操作bean配置元数据。也就是说，Spring IoC容器允许BeanFactoryPostProcessor读取配置元数据，并可能在容器实例化除BeanFactoryPostProcessor实例之外的任何bean之前更改它。

您可以配置多个BeanFactoryPostProcessor实例，并且可以通过设置order属性来控制这些BeanFactoryPostProcessor实例运行的顺序。但是，只有当BeanFactoryPostProcessor实现了Ordered接口时，才能设置此属性。如果您编写自己的BeanFactoryPostProcessor，您也应该考虑实现Ordered接口。有关更多细节，请参阅BeanFactoryPostProcessor和Ordered接口的javadoc。

```
如果您想要更改实际的bean实例(即，从配置元数据创建的对象)，那么您需要使用BeanPostProcessor(在前面通过使用BeanPostProcessor定制bean中描述过)。虽然在BeanFactoryPostProcessor中使用bean实例在技术上是可能的(例如，通过使用BeanFactory.getBean())，但是这样做会导致过早的bean实例化，违反标准的容器生命周期。这可能会导致负面的副作用，比如绕过bean后处理。
另外，BeanFactoryPostProcessor实例是按容器限定范围的。这只有在使用容器层次结构时才相关。如果您在一个容器中定义了BeanFactoryPostProcessor，那么它只应用于该容器中的bean定义。一个容器中的Bean定义不会被另一个容器中的BeanFactoryPostProcessor实例进行后处理，即使两个容器都属于相同的层次结构。
```

当在ApplicationContext中声明bean工厂后处理器时，将自动运行它，以便对定义容器的配置元数据应用更改。Spring包括许多预定义的bean工厂后处理程序，比如PropertyOverrideConfigurer和PropertySourcesPlaceholderConfigurer。您还可以使用自定义BeanFactoryPostProcessor—例如，注册自定义属性编辑器。

ApplicationContext自动检测部署到它中的任何实现BeanFactoryPostProcessor接口的bean。它在适当的时候使用这些豆子作为豆子工厂的后处理器。您可以像部署任何其他bean一样部署这些后处理器bean。

```
与BeanPostProcessors一样，您通常不希望将BeanFactoryPostProcessors配置为惰性初始化。如果没有其他bean引用bean(工厂)PostProcessor，则该后处理器将根本不会被实例化。因此，将其标记为惰性初始化将被忽略，并且即使您在元素的声明中将default-lazy-init属性设置为true, Bean(Factory)PostProcessor也将被急切地实例化。
```

#####  Example: The Class Name Substitution `PropertySourcesPlaceholderConfigurer`

您可以使用PropertySourcesPlaceholderConfigurer通过使用标准Java Properties格式将bean定义中的属性值外化到一个单独的文件中。这样做可以使部署应用程序的人员自定义特定于环境的属性，如数据库url和密码，而无需修改容器的主XML定义文件或其他文件的复杂性或风险。

考虑以下基于xml的配置元数据片段，其中定义了一个带有占位符值的数据源:

```xml
<bean class="org.springframework.context.support.PropertySourcesPlaceholderConfigurer">
    <property name="locations" value="classpath:com/something/jdbc.properties"/>
</bean>

<bean id="dataSource" destroy-method="close"
        class="org.apache.commons.dbcp.BasicDataSource">
    <property name="driverClassName" value="${jdbc.driverClassName}"/>
    <property name="url" value="${jdbc.url}"/>
    <property name="username" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
</bean>
```

