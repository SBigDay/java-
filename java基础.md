java基础

## javadoc注释

/**+enter

## 执行顺序

\1. 先执行父类的静态代码块，再执行当前类的静态代码块，再执行父类的构造器，最后执行当前类的构造器。百度查static关键字

\2. 构造器执行之前，先执行父类的成员变量，父类的执行完了，到自身，执行构造器之前，先执行成员变量。

## static关键字

\1. 不用new一个对象，可直接被调用

\2. 静态方法只能调用静态方法或变量。

\3. 静态的只需要被加载一次，所以可以在多次被加载的地方用静态修饰实现优化

\4. Static关键词不允许修饰局部变量

## this关键字 super

指代当前类下的成员变量，不指代局部变量

super 调用父类。

## return break continue

continue:结束单次循环

break:结束

return:结束当前方法

## 反射

### 1.简介

- RTTI，编译器在编译时打开和检查.class文件    也就是正射，直接new
- 反射，运行时打开和检查.class文件

Class类与java.lang.reflect类库一起对反射进行了支持，该类库包含Field、Method和Constructor类，这些类的对象由JVM在启动时创建，用以表示未知类里对应的成员。这样的话就可以使用Contructor创建新的对象，用get()和set()方法获取和修改类中与Field对象关联的字段，用invoke()方法调用与Method对象关联的方法。另外，还可以调用getFields()、getMethods()和getConstructors()等许多便利的方法，以返回表示字段、方法、以及构造器对象的数组，这样，对象信息可以在运行时被完全确定下来，而在编译时不需要知道关于类的任何事情。

正射

```
Apple apple = new Apple(); //直接初始化，「正射」
apple.setPrice(4);
```

反射

```
Class clz = Class.forName("com.chenshuyi.reflect.Apple");//获取类的 Class 对象实例  即引用
Method method = clz.getMethod("setPrice", int.class);//获得方法
Constructor constructor = clz.getConstructor();获得构造器
Object object = constructor.newInstance();//获得反射对象    即new一个
method.invoke(object, 4);
```

```
public class Apple {

    private int price;

    public int getPrice() {
        return price;
    }

    public void setPrice(int price) {
        this.price = price;
    }

    public static void main(String[] args) throws Exception{
        //正常的调用
        Apple apple = new Apple();
        apple.setPrice(5);
        System.out.println("Apple Price:" + apple.getPrice());
        //使用反射调用
        Class clz = Class.forName("com.chenshuyi.api.Apple");
        Method setPriceMethod = clz.getMethod("setPrice", int.class);
        Constructor appleConstructor = clz.getConstructor();
        Object appleObj = appleConstructor.newInstance();
        setPriceMethod.invoke(appleObj, 14);
        Method getPriceMethod = clz.getMethod("getPrice");
        System.out.println("Apple Price:" + getPriceMethod.invoke(appleObj));
    }
}
```

### 2.反射常用API

1.获取类的引用。

```
Class clz = Class.forName("java.lang.String");   知道类的路径名的情况下
```

```
Class clz = String.class;            编译前就知道class的情况
```

```
String str = new String("Hello");
Class clz = str.getClass();
```

2.通过反射创建类对象

```
Class clz = Apple.class;
Apple apple = (Apple)clz.newInstance();
```

```
Class clz = Apple.class;
Constructor constructor = clz.getConstructor();          无参
Apple apple = (Apple)constructor.newInstance();
```

```
Class clz = Apple.class;
Constructor constructor = clz.getConstructor(String.class, int.class);    有参
Apple apple = (Apple)constructor.newInstance("红富士", 15);
```

3.1反射获取类的属性，但不可以获得私有属性

```
Class clz = Apple.class;
Field[] fields = clz.getFields();
for (Field field : fields) {
    System.out.println(field.getName());
}
```

3.2反射获得所有属性，包括私有的。

```
Class clz = Apple.class;
Field[] fields = clz.getDeclaredFields();
for (Field field : fields) {
    System.out.println(field.getName());
}
```

4.获取方法和方法的使用

```
Method setPriceMethod = clz.getMethod("setPrice", int.class);
setPriceMethod.invoke(appleObj, 14);
```



## String Stringbuffer Stringbuilder

速度：Stringbuilder>Stringbuffer>String

String 定义一个对象不可以改变，如果改变就会生成一个新的对象。

StringBuffer的长度可以改变,线程安全

Stringbuilder 线程不安全

## Java.util.optional  是一个可以为null的容器，可以很好的解决空指针问题。

 List<String> alpha = Arrays.asList("a", "b", "c", "d");

List<String>collect=alpha.stream().map(String::toUpperCase).collect(Collectors.toList());  省略了for循环的操作

将alpha集合全都变成大写    toUpperCase是String的方法。

## Oracle insert all 把 一个表的数据同时插入到多个表中

## Insert all 

into t1(object_name,object_id)

Into t2(object_name,object_id)

Select * from t;

Commit;

insert first：对于每一行数据，只插入到第一个when条件成立的表，不继续检查其他条件。
insert all ： 对于每一行数据，对每一个when条件都进行检查，如果满足条件就执行插入操作。 

## Lambda表达式

说先了解函数式接口：我们把这些只拥有一个方法的接口称为*函数式接口*

lambda表达式是匿名方法，它提供了轻量级的语法，从而解决了匿名内部类带来的“高度问题”。

lambda表达式的语法由参数列表、箭头符号->和函数体组成。函数体既可以是一个表达式，也可以是一个语句块



## 双重for循环 用增强for循环会出错 报并发性异常

## Calender用于日历操作

```
public class CalendarTest {
    Calendar calendar = null;

    @Before
    public void test() {
        calendar = Calendar.getInstance();
    }

    // 基本用法，获取年月日时分秒星期
    @Test
    public void test1() {
        // 获取年
        int year = calendar.get(Calendar.YEAR);

        // 获取月，这里需要需要月份的范围为0~11，因此获取月份的时候需要+1才是当前月份值
        int month = calendar.get(Calendar.MONTH) + 1;

        // 获取日
        int day = calendar.get(Calendar.DAY_OF_MONTH);

        // 获取时
        int hour = calendar.get(Calendar.HOUR);
        // int hour = calendar.get(Calendar.HOUR_OF_DAY); // 24小时表示

        // 获取分
        int minute = calendar.get(Calendar.MINUTE);

        // 获取秒
        int second = calendar.get(Calendar.SECOND);

        // 星期，英语国家星期从星期日开始计算
        int weekday = calendar.get(Calendar.DAY_OF_WEEK);

        System.out.println("现在是" + year + "年" + month + "月" + day + "日" + hour
                + "时" + minute + "分" + second + "秒" + "星期" + weekday);
    }

    // 一年后的今天
    @Test
    public void test2() {
        // 同理换成下个月的今天calendar.add(Calendar.MONTH, 1);
        calendar.add(Calendar.YEAR, 1);

        // 获取年
        int year = calendar.get(Calendar.YEAR);

        // 获取月
        int month = calendar.get(Calendar.MONTH) + 1;

        // 获取日
        int day = calendar.get(Calendar.DAY_OF_MONTH);

        System.out.println("一年后的今天：" + year + "年" + month + "月" + day + "日");
    }

    // 获取任意一个月的最后一天
    @Test
    public void test3() {
        // 假设求6月的最后一天
        int currentMonth = 6;
        // 先求出7月份的第一天，实际中这里6为外部传递进来的currentMonth变量
        // 1
        calendar.set(calendar.get(Calendar.YEAR), currentMonth, 1);

        calendar.add(Calendar.DATE, -1);

        // 获取日
        int day = calendar.get(Calendar.DAY_OF_MONTH);

        System.out.println("6月份的最后一天为" + day + "号");
    }

    // 设置日期
    @Test
    public void test4() {
        calendar.set(Calendar.YEAR, 2000);
        System.out.println("现在是" + calendar.get(Calendar.YEAR) + "年");

        calendar.set(2008, 8, 8);
        // 获取年
        int year = calendar.get(Calendar.YEAR);

        // 获取月
        int month = calendar.get(Calendar.MONTH);

        // 获取日
        int day = calendar.get(Calendar.DAY_OF_MONTH);

        System.out.println("现在是" + year + "年" + month + "月" + day + "日");
    }
}

```

### 1.获取当前日期

```
 Date time = new Date();
        Calendar calendar = Calendar.getInstance();
        calendar.setTime(time);
        calendar.add(Calendar.DATE,0);
        time = calendar.getTime();
        SimpleDateFormat format= new SimpleDateFormat("yyyy-MM-dd");
        String dateString = format.format(time);
```

## 2.集合

### ArrayList

1.底层使用的是数组。

2.该集合是长度可变的数组，当进行扩容的时候，会将老数组的内容拷贝到新数组中，新数组的长度大约为老数组的1.5倍，代价还是很大的

3.采用Fail-fast机制，当存在并发情况时，迭代器会很快操作失败进行阻止。

4.remove操作会让下标从最后一位向前移一位。

不能存储相同的元素

```
List<NodeDemandVO> nodeDemandVOS =  new ArrayList<NodeDemandVO>(sortMap.values());
```

将map集合中的value全都存到list中；

1.利用for循环 插入list集合时，一定要在循环内部初始化，如果在外面初始化，就会只开辟一个内存空间，list.add()加的内容都为相同的

```
   List<User_class_grade> list=new ArrayList<>();
            User user =null;
            for(int i=0;i<user_classes.size();i++){
                User_class_grade ucg=new User_class_grade();
                User_class uc = user_classes.get(i);
                user=teacherService.selectUserByUserId(uc.getUser_id());
                Class aClass = teacherService.selectClassByUserId(uc.getClass_id());
                ucg.setUserName(user.getName());
                ucg.setGrade(uc.getGrade());
                ucg.setClassName(aClass.getName());
                list.add(ucg);
            }
```

### LinkedList

1.底层是双向链表

2.双向链表有3个属性 prev,next,item

3.查询的时候采用二分查找。

### HashMap

1.底层是数组，每一个数组对应一个单向链表，当链表到达一定的阈值时，链表转变为红黑树，减少链表查询时间

2.底层数组进行扩容是时候，需要重新计算在数组中的位置。很耗性能

红黑树属于平衡二叉树。它不严格是因为它不是严格控制左、右子树高度或节点数之差小于等于1，但红黑树高度依然是平均log(n)，且最坏情况高度不会超过2log(n)。红黑树能够以O(log2(N))的时间复杂度进行搜索、插入、删除操作。此外,任何不平衡都会在3次旋转之内解决。这一点是AVL所不具备的。

### HashTable

跟HashMap很像，但不允许使用null值和null键

Synchronized是锁住整张表的，让线程独占。

### ConcurrentHashmap

将HashMap分段，每一段都相当于一个hashtable，当在不同的段中进行操作时，可以并发进行。

### TreeMap

底层是红黑树

### set



## String 包

### subString方法 

   应有返回值  要不截取无效。

## Log日志

直接加@slf4j

在 try catch的时候尽量不要写e.printStackTrace(); 这样会影响系统性能  用log.info()的形式。

## el表达式

${c.id} 属性字段首字母一定要小写

## C/S B/S

C/S    客户端/服务器端              一般是局域网

B/S   浏览器/服务器端               一般是广域网 WWW

## JVM JRE JDK

JDK>JRE>JVM

JVM会将java程序转换为.class文件。但是无法运行.class文件。因为没有类库

jre是.class文件的运行环境，因为其包含jvm和其运行所需要的类库。

jdk中有jre还有一些java基础工具

# 项目实例

## 1.抽象类实现某个接口 

 可以先实现接口的部分方法，当子类继承这个抽象类时，必须实现剩余的方法。

```
public interface IMessage {
    public Map<String,Object> getMessageParam();//获取生产参数
    public void setMessageParam(Map<String, Object> messageParam);
    public void sendMessage()throws Exception;
}
```

```
public  abstract class MyAbstractMessage implements IMessage{
    private Map<String, Object> messageParam;// 这里可以理解为生产产品所需要的原材料库。最好是个自定义的对象，这里为了不引起误解使用Map。

    @Override
    public Map<String, Object> getMessageParam() {
        return messageParam;
    }

    @Override
    public void setMessageParam(Map<String, Object> messageParam) {
        this.messageParam = messageParam;
    }
}
```

```
public class MymessageEmail extends MyAbstractMessage{
   
    @Override
    public void sendMessage() throws Exception {
        if(getMessageParam()==null||getMessageParam().get("EMAIL")==null||"".equals(getMessageParam().get("EMAIL"))){
            throw new Exception("发送短信，需要传入email参数");
        }
        System.out.println("发送邮件给"+getMessageParam().get("EMAIL"));
    }
}

```

## 2.分页 

```
Page<ReclaimDemand> objects = PageHelper.startPage(page, limit);
demandBiz.findByCriteria(fromDate, toDate, procInstIdList,state);//分页查询
PageInfo<ReclaimDemand> pageInfo = new PageInfo<>(objects);
List<ReclaimDemand> demandList = pageInfo.getList();
```

## 3.任务调度（job schedule）

### 1.简介

资源有限(例如任务太多CPU处理不过来)且任务的重要性不一样，所以需要调度。

### 2.四种任务调度

Timer:基于起始时间和时间间隔的任务调度，Timer就足够了

SchaduledExecutor:同时调度多个任务，基于线程池的的SchaduledExecutor更合适

Quartz:更复杂的业务                         可以与spring结合

JcronTab:更复杂的(更适合与web服务器结合在一起)

### 3.学习Quartz(石英)

#### 1.简介：

Quartz是一个轻量级的java库，几乎包含了所有的定时功能，主要接口是Schadule。还有一些简单的操作：安排任务，取消任务，启动任务，停止任务。如果想在应用中使用Quartz，应该实现Job接口，实现execute接口，如果想要时间到了通知你的功能，实现TriggerListenner或者JobListener接口。Quartz任务可以在你的应用中启动和执行，可以作为一个独立的应用程序（通过RMI接口[即Remote Method Invoke 远程方法调用]），也可是在一个J2EE应用中执行。



在SSM框架中使用Quartz框架的时候，只用启动服务，根本不需要调用接口路径，Quartz就会自己执行，时间自己走

#### 2.基础结构

1.job接口:实现execute

2.jobdetail:存储job实例的状态信息，调度器需要借助jobdetail对象添加job实例

3.Trigger：描述触发job执行的触发规则，主要有SimpleTrigger(当只触发一次或者固定时间触发规则)和CronTrigger(可以通过cron表达式定义复杂的调度方案)实现类

4.Calander:日历，一个Trigger可以和多个Calander结合。

5.Schedule:一个 Quartz 的独立运行容器，Trigger 和 JobDetail 可以注册到 Scheduler 中，两者在 Scheduler 中拥有各自的组及名称。组及名称是 Scheduler 查找容器中某一对象的依据，Trigger 和 JobDetail 的组及名称的组合都必须唯一（但 Scheduler和 Trigger 的组合名称可以相同，因为他们是不同的类型，处于不同的容器中）。Scheduler 可以将 Trigger 绑定到某一个 JobDetails 中，这样当 Trigger 被触发时，对应的 Job 就被执行。一个 Job 可以对应多个 Trigger，但一个 Trigger 只能对应一个 Job。

#### 3.应用

##### 1.使用SimpleTrigger

步骤：

　　实现 Job 接口，可使 Java 类变为可调度的任务；

　　创建描述 Job 的 JobDetail 对象；

　　创建 SimpleTrigger 对象；

　　设置触发 Job 执行的时间规则；

　　通过 SchedulerFactory 获取 Scheduler 对象；

　　向 SchedulerFactory 中注册 JobDetail 和 Trigger；

　　启动调度任务。

```
	<dependency> <!--quartz依赖-->
			<groupId>org.quartz-scheduler</groupId>
			<artifactId>quartz</artifactId>
			<version>2.2.3</version>
	</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-context-support</artifactId>
	</dependency>
	dependency>
			<groupId>org.quartz-scheduler</groupId>
			<artifactId>quartz-jobs</artifactId>
			<version>2.2.3</version>
	</dependency>
```



```
public class SimpleJob implements Job {
    @Override
    public void execute(JobExecutionContext jobExecutionContext) throws JobExecutionException {
        System.out.println("执行任务"+new Date());
    }

```

```
  public static void main(String[] args) throws SchedulerException {
        //定义一个jobDetail
        JobDetailImpl jd=new JobDetailImpl();
        jd.setName("test-job");
        jd.setGroup("test-group");
        jd.setJobClass(SimpleJob.class);
        SimpleTriggerImpl simpleTrigger=new SimpleTriggerImpl();
        simpleTrigger.setName("test-trigger");
        simpleTrigger.setStartTime(new Date());
        simpleTrigger.setRepeatCount(10);//设置重复次数
        simpleTrigger.setRepeatInterval(2000);//设置重复间隔
        SchedulerFactory schedulerFactory=new StdSchedulerFactory();//必须是工厂模式，因为的一直生产，隔几秒就得生产一次
        Scheduler scheduler = schedulerFactory.getScheduler();
        scheduler.scheduleJob(jd,simpleTrigger);
        scheduler.start();
    }

```



##### 2.使用CronTrigger

1.Cron表达式

Quartz 使用类似于 Linux 下的 Cron 表达式定义时间规则，Cron 表达式由 6 或 7 个空格分隔的时间字段组成。

Cron 表达式对特殊字符的大小写不敏感.

2.Cron字符

*：每，每分钟、每秒钟

?：只能用在日期和星期字段中，表示一个毫无意义的值，相当于占位符

-：表示一个范围。例如：在小时字段中使用 10-12，表示从 10 点到 12 点。即 10，11, 12

,：表示一个列表值。例如：在星期字段中使用 MON,WED,FRI，表示星期一，星期三，星期五

/：x/y 表示一个等步长序列，x 为起始值，y 为增量步长。例如：在分钟字段中使用 0/15，则表示为 0，15，30 和 45 秒；而 5/15 在分钟字段表示 5，20，35，50 。*/y 等同于 0/y

L：该字符只在日期和星期字段中使用，代表 Last。

L 用在日期字段中，代表这个月的最后一天

L 用在星期字段中，代表星期六
若 L 出现在星期字段里，且前面有一个数值 X，则表示 这个月的最后星期 X，例如 6L 表示该月的最后一个星期五

W：该字符只能出现在日期字段里，是对前导日期的修饰，表示离该日期最近的工作日。例如：15W 表示离该月 15 号最近的工作日：若 15 号是星期六，则匹配 14 号；若 15 号是星期日，则匹配 16 号；若 15 号是星期二，则匹配 15 号。
注意1：关联的匹配日期不能跨月。例如用户指定 1W，若 1 号是星期六，则匹配 3 号。
注意2：W 字符串只能指定单一日期，而不能指定日期范围。

LW：在日期字段可以组合使用 LW，意为当月的最后一个工作日

#：只能在星期字段中使用，表示当月某个工作日。例如：6#3 表示当月的第三个星期五（6表示星期五，#3表示当月的第三个），4#5 表示当月的第五个星期三，若没有第五个星期三，则忽略不触发

C：该字符只在日期和星期字段中使用，代表 Calendar。意为计划所关联的日期，如果日期没有被关联，则相当于日历中所有日期。例如：5C 在日期字段中就相当于 5 日以后的第一天。1C 在星期字段中相当于星期日后的第一天。

3.表达式实例

```
0 0 12 * * ?：每天中午12点触发

0 15 10 ? * *：每天上午10:15触发

0 15 10 * ? *： 每天上午10:15触发

0 15 10 * * ?： 每天上午10:15触发

0 15 10 * * ? 2005： 2005年的每天上午10:15触发

0 * 14 * * ?：在每天下午2点到下午2:59期间的每1分钟触发

0 0/5 14 * * ?：在每天下午2点到下午2:55期间的每5分钟触发

0 0/5 14, 18 * * ?：在每天下午2点到2:55期间和下午6点到6:55期间的每5分钟,触发

0 0-5 14 * * ?：在每天下午2点到下午2:05期间的每1分钟触发

0 10,44 14 ? 3 WED：每年三月的星期三的下午2:10和2:44触发

0 15 10 ? * MON-FRI：周一至周五的上午10:15 触发

0 15 10 15 * ?：每月15日上午10:15 触发

0 15 10 L * ?：每月最后一日的上午10:15 触发

0 15 10 ? * 6L：每月的最后一个星期五上午10:15 触发

0 15 10 ? * 6L 2002-2005：2002年至2005年的每月的最后一个星期五上午10:15触发

0 15 10 ? * 6#3：每月的第三个星期五上午10:15触发
```

4.cronTrigger使用

```
 public static void main(String[] args) {

        DateFormat df=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss:SSS");
        System.out.println("开始时间："+ df.format(Calendar.getInstance().getTime()));

        //通过schedulerFactory获取一个调度器
        SchedulerFactory schedulerfactory = new StdSchedulerFactory();
        Scheduler scheduler = null;
        try {
//      通过schedulerFactory获取一个调度器
            scheduler = schedulerfactory.getScheduler();

//       创建jobDetail实例，绑定Job实现类
//       指明job的名称，所在组的名称，以及绑定job类
            JobDetail job = JobBuilder.newJob(MyJob.class).withIdentity("job1", "jgroup1").build();


//       定义调度触发规则

//      使用simpleTrigger规则
//        Trigger trigger=TriggerBuilder.newTrigger().withIdentity("simpleTrigger", "triggerGroup")
//                        .withSchedule(SimpleScheduleBuilder.repeatSecondlyForever(1).withRepeatCount(8))
//                        .startNow().build();
//      使用cornTrigger规则  每天10点42分
            Trigger trigger = TriggerBuilder.newTrigger().withIdentity("cronTrigger", "triggerGroup")
                    .withSchedule(CronScheduleBuilder.cronSchedule("0/15 * * * * ? *"))
                    .startNow().build();

//       把作业和触发器注册到任务调度中
            scheduler.scheduleJob(job, trigger);

//       启动调度
            scheduler.start();

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```

```
public class MyJob implements Job {
    @Override
    //把要执行的操作，写在execute方法中
    public void execute(JobExecutionContext arg0) throws JobExecutionException {
        DateFormat df=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss:SSS");
        System.out.println("测试Quartz"+ df.format(Calendar.getInstance().getTime()));
    }
}

```



#### 3.Spring和Qurtz的整合

##### 1.,mvc

如果使用spring 可通过xml文件里的工厂bean依赖注入

```
<bean id="simpleTrigger" class="org.springframework.scheduling.quartz.SimpleTriggerBean"> 
<property name="jobDetail" ref="job1" /> 
<property name="startDelay" value="0" /><!– 调度工厂实例化后，经过0秒开始执行调度 –> 
<property name="repeatInterval" value="2000" /><!– 每2秒调度一次 –> 
</bean> 
<bean id="cronTrigger" class="org.springframework.scheduling.quartz.CronTriggerBean"> 
<property name="jobDetail" ref="job1" /> 
<!—每天12:00运行一次 —> 
<property name="cronExpression" value="0 0 12 * * ?" /> 
</bean> 
<bean class="org.springframework.scheduling.quartz.SchedulerFactoryBean"> 
<property name="triggers"> 
<list> 
<ref bean="cronTrigger" /> 
</list> 
</property> 
</bean> 
```

##### 2.spring自带的定时任务，并没整合Quartz 直接用Cron

直接使用注解，零配置。

```
@Component          //注入bean
@Configurable      //引入bean
@EnableScheduling         //开启Schedule
public class ScheduledTask {


    @Scheduled(fixedRate = 1000 * 30)//30秒执行一次
    public void reportCurrentTime(){
        System.out.println ("Scheduling Tasks Examples: The time is now " + dateFormat ().format (new Date()));
    }

    //每1分钟执行一次
    @Scheduled(cron = "0 */1 *  * * * ")
    public void reportCurrentByCron(){
        System.out.println ("Scheduling Tasks Examples By Cron: The time is now " + dateFormat ().format (new Date ()));
    }

    private SimpleDateFormat dateFormat(){
        return new SimpleDateFormat ("HH:mm:ss");
    }
}
```

3.

## 4.Execl表格的导入和导出

1. 

```
常用组件：
HSSFWorkbook                      excel的文档对象
HSSFSheet                         excel的表单
HSSFRow                           excel的行
HSSFCell                          excel的格子单元
HSSFFont                          excel字体
HSSFDataFormat                    日期格式
HSSFHeader                        sheet头
HSSFFooter                        sheet尾（只有打印的时候才能看到效果）
样式：
HSSFCellStyle                       cell样式
辅助操作包括：
HSSFDateUtil                        日期
HSSFPrintSetup                      打印
HSSFErrorConstants                  错误信息表
```

2.创建一个最基本的Excel

```
public static void main(String[] args) throws IOException {
        //创建HSSFWorkbook对象
        HSSFWorkbook wb = new HSSFWorkbook();
//创建HSSFSheet对象
        HSSFSheet sheet = wb.createSheet("sheet0");
//创建HSSFRow对象
        HSSFRow row = sheet.createRow(0);
//创建HSSFCell对象
        HSSFCell cell=row.createCell(0);
//设置单元格的值
        cell.setCellValue("单元格中的中文");
//输出Excel文件
        FileOutputStream output= null;
        try {
            output = new FileOutputStream("d:\\workbook.xls");
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }
        wb.write(output);
        output.flush();
    }
```

3.创建一个三行三列的Excel并直接返回浏览器(B/S形式)

```
//创建HSSFWorkbook对象(excel的文档对象)
      HSSFWorkbook wb = new HSSFWorkbook();
//建立新的sheet对象（excel的表单）
HSSFSheet sheet=wb.createSheet("成绩表");
//在sheet里创建第一行，参数为行索引(excel的行)，可以是0～65535之间的任何一个
HSSFRow row1=sheet.createRow(0);
//创建单元格（excel的单元格，参数为列索引，可以是0～255之间的任何一个
HSSFCell cell=row1.createCell(0);
      //设置单元格内容
cell.setCellValue("学员考试成绩一览表");
//合并单元格CellRangeAddress构造参数依次表示起始行，截至行，起始列， 截至列
sheet.addMergedRegion(new CellRangeAddress(0,0,0,3));
//在sheet里创建第二行
HSSFRow row2=sheet.createRow(1);    
      //创建单元格并设置单元格内容
      row2.createCell(0).setCellValue("姓名");
      row2.createCell(1).setCellValue("班级");    
      row2.createCell(2).setCellValue("笔试成绩");
row2.createCell(3).setCellValue("机试成绩");    
      //在sheet里创建第三行
      HSSFRow row3=sheet.createRow(2);
      row3.createCell(0).setCellValue("李明");
      row3.createCell(1).setCellValue("As178");
      row3.createCell(2).setCellValue(87);    
      row3.createCell(3).setCellValue(78);    
  //.....省略部分代码
 
 
//输出Excel文件
    OutputStream output=response.getOutputStream();
    response.reset();
    response.setHeader("Content-disposition", "attachment; filename=details.xls");
    response.setContentType("application/msexcel");        
    wkb.write(output);
    output.close();
retrun null;
```

该代码表示将details.xls的Excel文件通过应答实体（response）输出给请求的客户端浏览器，客户端可保存或直接打开。

4.对样式进行处理

```
合并单元格：sheet.addMergedRegion(new CellRangeAddress(0,0,0,2));
```

```
调整未使用的单元格尺寸：sheet.setDefaultRowHeightInPoints(50);
```

```
设置列宽：sheet.setDefaultColumnWidth(50);
```

```
设置指定列的宽度：sheet.setColumnWidth(1,80*256);    列的单位长度是1/256
```

5.导入Excel

行操作

```
  public static void main(String[] args) throws IOException {
        FileInputStream fileInputStream=new FileInputStream("d:/ExcelTest/5.xls");
        HSSFWorkbook hssfWorkbook=new HSSFWorkbook(fileInputStream);
        List<Student> list=new ArrayList<>();
         for (Row row:hssfWorkbook.getSheetAt(0)){
             if(row.getRowNum()<2){
                 continue;
             }
             Student student=new Student();
             student.setName(row.getCell(0).getStringCellValue());
             student.setClasses(row.getCell(1).getStringCellValue());
             student.setGrade(row.getCell(2).getNumericCellValue());
             list.add(student);
         }
        fileInputStream.close();
        System.out.println(list.toSring());
    }
```





## 5.邮件服务

### 1.两个协议

1.SMTP协议:用户给服务器（SMTP服务器）发送请求时候要遵守的协议

2.POP3协议:用户向服务器(POP3服务器)接收一个请求要遵守的协议

### 2.三大类

Message类       创建和解析邮件

Transport类     发送邮件

Store类              接收邮件

### 3.Session类

Session类用于定义整个应用程序所需的环境信息，以及收集客户端与邮件服务器建立网络连接的会话信息，如邮件服务器的主机名、端口号、采用的邮件发送和接收协议等。Session对象根据这些信息构建用于邮件收发的Transport和Store对象，以及为客户端创建Message对象时提供信息支持。

### 4.邮件结构的相关API

MimeMessage类表示整封邮件。

MimeBodyPart类表示邮件的一个MIME消息。

MimeMultipart类表示一个由多个MIME消息组合成的组合MIME消息。

```
<dependency>
			<groupId>javax.mail</groupId>
			<artifactId>mail</artifactId>
			<version>1.4.4</version>
</dependency>
```

例子一：简单邮件

```
public static void main(String[] args) throws MessagingException, GeneralSecurityException {
        Properties props = new Properties();

        // 开启debug调试
        props.setProperty("mail.debug", "true");
        // 发送服务器需要身份验证
        props.setProperty("mail.smtp.auth", "true");
        // 设置邮件服务器主机名
        props.setProperty("mail.host", "smtp.qq.com");
        // props.setProperty("mail.port", "465");
        // 发送邮件协议名称
        props.setProperty("mail.transport.protocol", "smtp");
        //设置一个安全协议
        MailSSLSocketFactory sf = new MailSSLSocketFactory();
        sf.setTrustAllHosts(true);
        props.put("mail.smtp.ssl.enable", "true");
        props.put("mail.smtp.ssl.socketFactory", sf);

        Session session = Session.getInstance(props);

        Message msg = new MimeMessage(session);
        //邮件名字
        msg.setSubject("seenews 错误");
        StringBuilder builder = new StringBuilder();
        builder.append("url = " + "http://blog.csdn.net/never_cxb/article/details/50524571");
        builder.append("\n页面爬虫错误第二次");
        builder.append("\n时间 " + System.currentTimeMillis());
        msg.setText(builder.toString());
        msg.setFrom(new InternetAddress("614880268@qq.com"));

        Transport transport = session.getTransport();
        transport.connect("smtp.qq.com", "614880268@qq.com", "jnbnuwiojwzbbcaf");

        transport.sendMessage(msg, new Address[] { new InternetAddress("tty_56318@163.com") });
        transport.close();
    }
```

例子二：包含附件，图片，正文的例子

```
 public static void main(String[] args) throws MessagingException, GeneralSecurityException, UnsupportedEncodingException {
        Properties props = new Properties();
        // 开启debug调试
        props.setProperty("mail.debug", "true");
        // 发送服务器需要身份验证
        props.setProperty("mail.smtp.auth", "true");
        // 设置邮件服务器主机名
        props.setProperty("mail.host", "smtp.qq.com");
        // props.setProperty("mail.port", "465");
        // 发送邮件协议名称
        props.setProperty("mail.transport.protocol", "smtp");
        //设置一个安全协议
        MailSSLSocketFactory sf = new MailSSLSocketFactory();
        sf.setTrustAllHosts(true);
        props.put("mail.smtp.ssl.enable", "true");
        props.put("mail.smtp.ssl.socketFactory", sf);
        Session session = Session.getInstance(props);
        MimeMessage message = CreateMessage(session);
        Transport transport = session.getTransport();
        transport.connect("smtp.qq.com", "614880268@qq.com", "jnbnuwiojwzbbcaf");

        transport.sendMessage(message, new Address[] { new InternetAddress("tty_56318@163.com") });
        transport.close();
    }
    public static MimeMessage CreateMessage(Session session) throws MessagingException, UnsupportedEncodingException {
        MimeMessage message=new MimeMessage(session);
        //设置邮件的基础信息
        message.setFrom(new InternetAddress("614880268@qq.com"));//从哪发
        message.setRecipient(Message.RecipientType.TO,new InternetAddress("tty_56318@163.com"));//发到那
        message.setSubject("test");
        //设置正文
        MimeBodyPart text=new MimeBodyPart();
        text.setContent("你好xxx，<img src='c:/dog.jpg' />第二次测试成功！<img src='cid:aaa.jpg' />", "text/html;charset=gbk");
        //设置图片
        MimeBodyPart image=new MimeBodyPart();
        image.setContentID("aaa.jpg");
        image.setDataHandler(new DataHandler(new FileDataSource("src/1.jpg")));
        //附件
        MimeBodyPart attach=new MimeBodyPart();
        DataHandler dh=new DataHandler(new FileDataSource("src/2.jar"));
        attach.setDataHandler(dh);
        //解决中文乱码问题
        attach.setFileName(MimeUtility.encodeText(dh.getName()));
        //描述正文和图片的关系
        MimeMultipart mimeMultipart1=new MimeMultipart();
        mimeMultipart1.setSubType("related");
        mimeMultipart1.addBodyPart(text);
        mimeMultipart1.addBodyPart(image);
        //将正文，图片，附件混合给message
        MimeMultipart mimeMultipart2=new MimeMultipart();
        mimeMultipart2.addBodyPart(attach);
        MimeBodyPart content=new MimeBodyPart();
        content.setContent(mimeMultipart1);
        mimeMultipart2.addBodyPart(content);
        mimeMultipart2.setSubType("mixed");
        message.setContent(mimeMultipart2);
        message.saveChanges();
        return  message;
    }
```

东一比东的密码：**jnbn****uwio****jwzb****bcaf**

# 测试

## Mock的用法

### 两大作用：1.验证方法的调用。

Mockito.verify(mockUserManager).performLogin("xiaochuang", "xiaochuang password");

验证mockUserManager里的performLogin方法是否使用 参数可以是anyString().

### 2指定某个方法的返回值或者执行特定的动作

PasswordValidator mockValidator = Mockito.mock(PasswordValidator.class);

 Mockito.when(validator.verifyPassword(anyString())).thenReturn(true);

## 当调用到verifyPassword时就返回true. 一般用在返回值是void的类型上

### 作用地点：

·  真实对象的行为是不确定的（例如，当前的时间或当前的温度）；

·  真实对象很难搭建起来；

·  真实对象的行为很难触发（例如，网络错误）；

·  真实对象速度很慢（例如，一个完整的数据库，在测试之前可能需要初始化）；

·  真实的对象是用户界面，或包括用户界面在内；

·  真实的对象使用了回调机制；

·  真实对象可能还不存在；

·  真实对象可能包含不能用作测试（而不是为实际工作）的信息和方法。

## 测试时有文件上传

**File** file=**new** File(**"D:****\\****app****\\****template.doc"**);

**FileInputStream** fileInputStream = **new** FileInputStream(file);
**MultipartFile** multipartFile = **new** MockMultipartFile(**"copy"**+file.getName(),file.getName(),**"a"**,fileInputStream);

 

## Mockmvc

应用于web层的测试

**private** **MockMvc** mockMvc;

 **@Before**

## **public void** setup(){

​     **MockitoAnnotations**.**initMocks**(**this**);
​     mockMvc = **MockMvcBuilders**.**standaloneSetup**(assemablevocontroller).build();
 }

 

   mockMvc.perform(**MockMvcRequestBuilders**.**get**(**"/selectAssemableNameByCode/{code}"**,**"A1132B"**)   路径和参数
            .accept(**MediaType**.**ALL**)  

指定请求的Accept头信息；     指定请求的contentType头信息；
            .contentType(**MediaType**.**APPLICATION_JSON**)
            .param(**"code"**,**"A1132B"**))
            .andDo(**MockMvcResultHandlers**.**print**())
            .andExpect(**MockMvcResultMatchers**.**status**().is(**200**))
            .andExpect(**MockMvcResultMatchers**.**handler**().handlerType(**AssemableVoController**.**class**))
            .andReturn();
} 



## 私有的测试

```
try {
    Class<?> clazz =ListTableMessageServiceImpl.class;
    Method method = clazz.getDeclaredMethod("getScopeByString",String.class);
    method.setAccessible(true);
    Object o = method.invoke(clazz.newInstance(), "11111");
    Assert.assertNotNull(o);
} catch (NoSuchMethodException e) {
    e.printStackTrace();
} catch (IllegalAccessException e) {
    e.printStackTrace();
} catch (InstantiationException e) {
    e.printStackTrace();
} catch (InvocationTargetException e) {
    e.printStackTrace();
}
```

# Spring框架

## 配置方式

1.传统的xml配置

2.spring2.5后新增注解配置

3.spring4.0新增java配置

### XML配置

### 注解配置

@Component 是spring容器的基本注解，表示一个bean组件，相当于一个xml文件中的<bean>元素

1. @Component：可以用于注册所有bean

2. @Repository：主要用于注册dao层的bean

3. @Controller：主要用于注册控制层的bean

4. @Service：主要用于注册服务层的bean

5. 上面的自动扫描而@Configration和@bean得用

   ```
   ApplicationContext context = new AnnotationConfigApplicationContext(TestConfiguration.class);
   TestBean tb = (TestBean) context.getBean("testBean");//方法名
           tb.sayHello();//注入类的方法名
   ```

   

```
package annotationConfig;

import org.springframework.stereotype.Component;

/**
 * 如果属性名称是value，value可以省略。
 * 如果不指定value，默认值是类名首先字母变为小写。
 * @Component(value="beanId") 就是把当前类实例化。相当于<bean id="beanId">
 */
@Component
public class HelloWorld {

    public void sayHello() {
        System.out.println("Hello World");
    }
}
```

在使用注解配置之前，必须开启扫描，这样才能扫描到注解

```
<!-- 开启注解扫描  -->
<context:component-scan base-package="com.example.annotationconfig"/>
```

### java配置

@Configuration和@bean注解是核心

@Configuration修饰类相当于一个xml配置文件，

@Bean作用在方法上相当于一个<bean>.

```
package cn.qlq;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration // 通过该注解来表明该类是一个Spring的配置，相当于一个xml文件
@ComponentScan(basePackages = "cn.qlq") // 配置扫描包
public class SpringConfig {

    @Bean // 通过该注解来表明是一个Bean对象，相当于xml中的<bean>
    public UserDao getUserDAO() {
        return new UserDao(); // 直接new对象做演示
    }

}
```

自己感觉没有注解配置好

## IOC控制反转

是spring的核心思想之一  即当需要创建一个对象时不需要自己new一个，而是通过容器自动注入。

控制：IOC控制对象，主要是控制外部获取资源(不只是对象，也有可能是文件等)

反转：哪里需要对象，提前用注解标记，需要的时候直接通过容器注入。

正传：需要的时候自己手动注入。

作用：松耦合，同时便于测试，由spring来负责对象的生命周期和对象间的关系。

那么 容器又是通过什么方式注入的呢：？

构造方法注入和setter注入 工厂方法注入      前提是xml文件配置

 还有通过配置@Autowired注入            通过注解配置      用的最多了

## AOP面向切面

springboot可直接通过@before或者@After等进行前值增强或后置增强等。

## Spring的5个作用域

1.singleton 

```
<bean id="userInfo" class="cn.lovepi.UserInfo" scope="singleton"></bean>  
```

单例的 顾名思义     只能存在一个实例

在默认情况下**ApplicationContext**容器在启动的时候自动实例化所有singleton的bean,虽然启动时有一些慢，但后期实例化的时候就不需要加载了。

2.prototype 

```
<bean id="userInfo" class="cn.lovepi.UserInfo" scope=" prototype "></bean>  
```

每次对bean请求的时候，都会创建一个新的作用域

对于有状态的bean应使用prototype 无状态的使用singleton

在使用**Web应用环境相关的Bean作用域**时，必须在Web容器中进行一些额外的配置：

使用http监听器进行监听

```
<listener> <listener-class>  
    org.springframework.web.context.request.RequestContextListener  
</listener-class></listener>  
```

3.request

```
<bean id="userInfo" class="cn.lovepi.UserInfo" scope=" request "></bean>  
```

针对每一次的http请求 ，该bean只在当前的http下有效。

4.session

```
<bean id="userInfo" class="cn.lovepi.UserInfo" scope=" session "></bean> 
```

准对session起作用

5.global session

```
<bean id="userInfo" class="cn.lovepi.UserInfo"scope=“globalSession"></bean> 
```

在全局的portlet session内都有效

# 线程

## 1.并行和并发

一个CPU可以同时处理一个线程

并行：系统有3个CPU，可以处理三个线程，就是并行

并发：系统只有一个CPU，只能处理一个线程，但是这些线程来回抢占CPU

同步：就是得排队，跟并发差不多。

同步方法的同步不是这个同步：同步方法跟加锁一个性质，串行执行。

## 2.线程的创建

3种：

继承Thread类

```
public class FirstThreadTest extends Thread{  
    int i = 0;  
    //重写run方法，run方法的方法体就是现场执行体  
    public void run()  
    {  
        for(;i<100;i++){  
        System.out.println(getName()+"  "+i);  
        }  
    }  
    public static void main(String[] args)  
    {  
        for(int i = 0;i< 100;i++)  
        {  
            System.out.println(Thread.currentThread().getName()+"  : "+i);  
            if(i==20)  
            {  
                new FirstThreadTest().start();  
                new FirstThreadTest().start();  
            }  
        }  
    }  
} 
```

实现runnable接口 

```
public class RunnableThreadTest implements Runnable  
{  
  
    private int i;  
    public void run()  
    {  
        for(i = 0;i <100;i++)  
        {  
            System.out.println(Thread.currentThread().getName()+" "+i);  
        }  
    }  
    public static void main(String[] args)  
    {  
        for(int i = 0;i < 100;i++)  
        {  
            System.out.println(Thread.currentThread().getName()+" "+i);  
            if(i==20)  
            {  
                RunnableThreadTest rtt = new RunnableThreadTest();  
                new Thread(rtt,"新线程1").start();  
                new Thread(rtt,"新线程2").start();  
            }  
        }  
    }   
}  
```

实现calable接口和future接口

```
public class CallableThreadTest implements Callable<Integer>  
{  
  
    public static void main(String[] args)  
    {  
        CallableThreadTest ctt = new CallableThreadTest();  
        FutureTask<Integer> ft = new FutureTask<>(ctt);  
        for(int i = 0;i < 100;i++)  
        {  
            System.out.println(Thread.currentThread().getName()+" 的循环变量i的值"+i);  
            if(i==20)  
            {  
                new Thread(ft,"有返回值的线程").start();  
            }  
        }  
        try  
        {  
            System.out.println("子线程的返回值："+ft.get());  
        } catch (InterruptedException e)  
        {  
            e.printStackTrace();  
        } catch (ExecutionException e)  
        {  
            e.printStackTrace();  
        }  
  
    }  
  
    @Override  
    public Integer call() throws Exception  
    {  
        int i = 0;  
        for(;i<100;i++)  
        {  
            System.out.println(Thread.currentThread().getName()+" "+i);  
        }  
        return i;  
    }  
  
}  
```

核心

```
 CallableThreadTest ctt = new CallableThreadTest();  
 FutureTask<Integer> ft = new FutureTask<>(ctt);  
  new Thread(ft,"有返回值的线程").start(); 
```

## 3.CountDownLoatch

提供的方法

```
1.public CountDownLatch(int count) {  };  //参数count为计数值
2.public void await() throws InterruptedException { };   //调用await()方法的线程会被挂起，它会等待直到count值为0才继续执行
3.public boolean await(long timeout, TimeUnit unit) throws InterruptedException { };  //和await()类似，只不过等待一定的时间后count值还没变为0的话就会继续执行
4.public void countDown() { };  //将count值减1
```

```
public class ContDownLatchTest {
    public static void main(String[] args) {
        final CountDownLatch latch = new CountDownLatch(2);

        new Thread(){
            public void run() {
                try {
                    System.out.println("子线程"+Thread.currentThread().getName()+"正在执行");
                    Thread.sleep(3000);
                    System.out.println("子线程"+Thread.currentThread().getName()+"执行完毕");
                    latch.countDown();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            };
        }.start();

        new Thread(){
            public void run() {
                try {
                    System.out.println("子线程"+Thread.currentThread().getName()+"正在执行");
                    Thread.sleep(3000);
                    System.out.println("子线程"+Thread.currentThread().getName()+"执行完毕");
                    latch.countDown();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            };
        }.start();
        new Thread(){
            @Override
            public void run() {
                try {
                    latch.await();
                    System.out.println("子线程"+Thread.currentThread().getName()+"正在执行");
                    Thread.sleep(3000);
                    System.out.println("子线程"+Thread.currentThread().getName()+"执行完毕");

                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            };
        }.start();
        try {
            System.out.println("等待2个子线程执行完毕...");
            latch.await();
            System.out.println("2个子线程已经执行完毕");
            System.out.println("继续执行主线程");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```



## 4.CyclicBarrier

循环栅栏      能被重用        CountDownLoatch只能用一次

提供的方法

```
1.构造方法(n)    得传参数
2.await()方法，当=每个线程调用await()方法都会阻塞，直到n个线程await()后才整体解除阻塞
3 public void reset()
```

```
 public static void main(String[] args) {
        int N = 4;
        CyclicBarrier barrier  = new CyclicBarrier(N);
         
        for(int i=0;i<N;i++) {
            new Writer(barrier).start();
        }
         
        try {
            Thread.sleep(25000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
         
        System.out.println("CyclicBarrier重用");
         
        for(int i=0;i<N;i++) {
            new Writer(barrier).start();
        }
    }
    static class Writer extends Thread{
        private CyclicBarrier cyclicBarrier;
        public Writer(CyclicBarrier cyclicBarrier) {
            this.cyclicBarrier = cyclicBarrier;
        }
 
        @Override
        public void run() {
            System.out.println("线程"+Thread.currentThread().getName()+"正在写入数据...");
            try {
                Thread.sleep(5000);      //以睡眠来模拟写入数据操作
                System.out.println("线程"+Thread.currentThread().getName()+"写入数据完毕，等待其他线程写入完毕");
             
                cyclicBarrier.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }catch(BrokenBarrierException e){
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName()+"所有线程写入完毕，继续处理其他任务...");
        }
    }
```



## 5.Semaphore

控制信号量，主要控制运行的线程个数

主要方法：

```
Semaphore(int permits):构造方法，创建具有给定许可数的计数信号量并设置为非公平信号量。

Semaphore(int permits,boolean fair):构造方法，当fair等于true时，创建具有给定许可数的计数信号量并设置为公平信号量。

void acquire():从此信号量获取一个许可前线程将一直阻塞。相当于一辆车占了一个车位。

void acquire(int n):从此信号量获取给定数目许可，在提供这些许可前一直将线程阻塞。比如n=2，就相当于一辆车占了两个车位。

void release():释放一个许可，将其返回给信号量。就如同车开走返回一个车位。

void release(int n):释放n个许可。

int availablePermits()：当前可用的许可数。
```



```

public class Test {
    public static void main(String[] args) {
        int N = 8;            //工人数
        Semaphore semaphore = new Semaphore(5); //机器数目
        for(int i=0;i<N;i++)
            new Worker(i,semaphore).start();
    }
     
    static class Worker extends Thread{
        private int num;
        private Semaphore semaphore;
        public Worker(int num,Semaphore semaphore){
            this.num = num;
            this.semaphore = semaphore;
        }
         
        @Override
        public void run() {
            try {
                semaphore.acquire();
                System.out.println("工人"+this.num+"占用一个机器在生产...");
                Thread.sleep(2000);
                System.out.println("工人"+this.num+"释放出机器");
                semaphore.release();           
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
 
执行结果：
工人0占用一个机器在生产...
工人1占用一个机器在生产...
工人2占用一个机器在生产...
工人4占用一个机器在生产...
工人5占用一个机器在生产...
工人0释放出机器
工人2释放出机器
工人3占用一个机器在生产...
工人7占用一个机器在生产...
工人4释放出机器
工人5释放出机器
工人1释放出机器
工人6占用一个机器在生产...
工人3释放出机器
工人7释放出机器
工人6释放出机器
--------------------- 
作者：Lyzxii 
来源：CSDN 
原文：https://blog.csdn.net/weixin_37598682/article/details/81451270 
版权声明：本文为博主原创文章，转载请附上博文链接！
```



## 6.Exchanger

用于两个线程交换值

用到exchaer.exchange()方法。只需把需要交换的信息放入方法中，框架会自动进行交换，因为框架里面有两个卡槽，放入里面的东西像魔术一样，自己就交换了。

```

class ExchangerTest extends Thread {
    private Exchanger<String> exchanger;
    private String string;
    private String threadName;
 
    public ExchangerTest(Exchanger<String> exchanger, String string,
            String threadName) {
        this.exchanger = exchanger;
        this.string = string;
        this.threadName = threadName;
    }
 
    public void run() {
        try {
            System.out.println(threadName + ": " + exchanger.exchange(string));
        } catch (InterruptedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
}
 
public class Test {
    public static void main(String[] args) {
        Exchanger<String> exchanger = new Exchanger<>();
        ExchangerTest test1 = new ExchangerTest(exchanger, "string1",
                "thread-1");
        ExchangerTest test2 = new ExchangerTest(exchanger, "string2",
                "thread-2");
 
        test1.start();
        test2.start();
    }
}
```



## 7.ThreadLocal

1.简介：

1）线程本地变量，用于解决高并发问题，有时可以不用synchronized和lock锁，影响执行效率。

threadlocal会对本地资源的消耗较大

2）有四个方法

```
public T get() { }
public void set(T value) { }
public void remove() { }
protected T initialValue() { }
```

set()用来设置当前线程中变量的副本，remove()用来移除当前线程中变量的副本，initialValue()是一个protected方法，一般是用来在使用时进行重写的，它是一个延迟加载方法。

set()的时候键值就是当前ThreadLocal变量，value就是自己要传入的。

```
  ThreadLocal<Long> longLocal = new ThreadLocal<Long>(){
        protected Long initialValue() {
            return Thread.currentThread().getId();
        };
    };
    ThreadLocal<String> stringLocal = new ThreadLocal<String>(){;
        protected String initialValue() {
            return Thread.currentThread().getName();
        };
    };
```

里面用了intialValue()方法，就是对intialValue进行重写，正常set()就是给initialVlue里存值

3）应用场景

```
class ConnectionManager {
     
    private static Connection connect = null;
     
    public static Connection openConnection() {
        if(connect == null){
            connect = DriverManager.getConnection();
        }
        return connect;
    }
     
    public static void closeConnection() {
        if(connect!=null)
            connect.close();
    }
}
```

如果多线程操作时就有可能重复开通或者关闭数据库连接库，想要解决高并发问题，就得加synchronized或者锁，synchronized修饰静态方法就会把整个类锁住，但是性能会受一定影响。

用ThreadLocal解决：

```
private static ThreadLocal<Connection> connectionHolder
= new ThreadLocal<Connection>() {
public Connection initialValue() {
    return DriverManager.getConnection(DB_URL);
}
};//相当于set
 
public static Connection getConnection() {
return connectionHolder.get();
}
```

session管理

```
private static final ThreadLocal threadSession = new ThreadLocal();
 
public static Session getSession() throws InfrastructureException {
    Session s = (Session) threadSession.get();
    try {
        if (s == null) {
            s = getSessionFactory().openSession();
            threadSession.set(s);
        }
    } catch (HibernateException ex) {
        throw new InfrastructureException(ex);
    }
    return s;
}
```

在实际开放中先把ThreadLocal写在一个实体类中然后调用方法像下面这样

```
public class Test {
    ThreadLocal<Long> longLocal = new ThreadLocal<Long>(){
        protected Long initialValue() {
            return Thread.currentThread().getId();
        };
    };
    ThreadLocal<String> stringLocal = new ThreadLocal<String>(){;
        protected String initialValue() {
            return Thread.currentThread().getName();
        };
    };
 
     
    public void set() {
        longLocal.set(Thread.currentThread().getId());
        stringLocal.set(Thread.currentThread().getName());
    }
     
    public long getLong() {
        return longLocal.get();
    }
     
    public String getString() {
        return stringLocal.get();
    }
     
    public static void main(String[] args) throws InterruptedException {
        final Test test = new Test();
 
        System.out.println(test.getLong());
        System.out.println(test.getString());
     
         
        Thread thread1 = new Thread(){
            public void run() {
                System.out.println(test.getLong());
                System.out.println(test.getString());
            };
        };
        thread1.start();
        thread1.join();
         
        System.out.println(test.getLong());
        System.out.println(test.getString());
    }
}
```



## 8.线程池

### 1.线程池的构造方法

```
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              RejectedExecutionHandler handler)；
```

### 2.线程池原理和流程

![](E:\Download\2012121931.jpg)

### 3.饱和策略RejectedExecutionHandler

Java提供了4种策略：

 1、AbortPolicy：直接抛出异常            默认的

 2、CallerRunsPolicy：只用调用所在的线程运行任务

 3、DiscardOldestPolicy：丢弃队列里最近的一个任务，并执行当前任务。

 4、DiscardPolicy：不处理，丢弃掉。

接下来我们设置为策略4

```
 public static void main(String[] args)
     {
         LinkedBlockingQueue<Runnable> queue =
             new LinkedBlockingQueue<Runnable>(3);
          RejectedExecutionHandler handler = new ThreadPoolExecutor.DiscardPolicy();
  
          ThreadPoolExecutor threadPool = new ThreadPoolExecutor(2, 5, 60, TimeUnit.SECONDS, queue,handler);
         for (int i = 0; i < 9 ; i++)
         {
             threadPool.execute(
                 new Thread(new ThreadPoolTest(), "Thread".concat(i + "")));
             System.out.println("线程池中活跃的线程数： " + threadPool.getPoolSize());
             if (queue.size() > 0)
             {
                System.out.println("----------------队列中阻塞的线程数" + queue.size());
            }
         }
         threadPool.shutdown();
     }
```

### 4.线程池的四种种类ExecutorService

1.newCachedThreadPool 

底层：返回ThreadPoolExecutor实例，corePoolSize为0；maximumPoolSize为Integer.MAX_VALUE；keepAliveTime为60L；unit为TimeUnit.SECONDS；workQueue为SynchronousQueue(同步队列)

应用：当线程池中的线程空闲时可被回收，用于负载较轻的服务器。

```
public class newCachedThreadPoolTest {
    public static void main(String[] args) {

        ExecutorService cachedThreadPool = Executors.newCachedThreadPool();
        for (int i = 0; i < 10; i++) {
            final int index = i;
            try {
                Thread.sleep(index * 1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            cachedThreadPool.execute(new Runnable() {

                @Override
                public void run() {
                    System.out.println(index);
                }
            });
        }
    }
}
```

2.newFIxedThreadPool

底层：返回ThreadPoolExecutor实例，接收参数为所设定线程数量nThread，corePoolSize为nThread，maximumPoolSize为nThread；keepAliveTime为0L(不限时)；unit为：TimeUnit.MILLISECONDS；WorkQueue为：new LinkedBlockingQueue<Runnable>() 无界阻塞队列

应用：线程的存活时间是没有限制的，当新任务来了，如果线程都在忙，就进入队列，一直等待机会。

用于服务器好的，长期执行的任务，因为线程不会被销毁

```
public class newFixedThreadPool {
    public static void main(String[] args) {
        ExecutorService executorService= Executors.newFixedThreadPool(3);
        for(int i=0;i<10;i++){
            final int index=i;
          executorService.execute(new Runnable() {
              @Override
              public void run() {
                  System.out.println("线程"+index+"正在巡行");
                  try {
                      Thread.sleep(3000);
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  }
              }
          });
        }
    }

}
```

3.底层：创建ScheduledThreadPoolExecutor实例，corePoolSize为传递来的参数，maximumPoolSize为Integer.MAX_VALUE；keepAliveTime为0；unit为：TimeUnit.NANOSECONDS；workQueue为：new DelayedWorkQueue() 一个按超时时间升序排序的队列

应用：跟newFixedThreadPool差不多，就是队列会按时间排序。更加有序

 支持定时及周期性任务执行。延迟执行示例代码如下

```
 ScheduledExecutorService scheduledThreadPool = Executors.newScheduledThreadPool(5);
        scheduledThreadPool.schedule(new Runnable() {

            @Override
            public void run() {
                System.out.println("//隔3S后执行");
            }
        }, 3, TimeUnit.SECONDS);
    }
```

```

scheduledThreadPool.scheduleAtFixedRate(new Runnable() {
 
@Override
public void run() {
System.out.println("隔1S每3S创建一个");
}
}, 1, 3, TimeUnit.SECONDS); 
```

4.newSingleThreadExecutor

底层：FinalizableDelegatedExecutorService包装的ThreadPoolExecutor实例，corePoolSize为1；maximumPoolSize为1；keepAliveTime为0L；unit为：TimeUnit.MILLISECONDS；workQueue为：new LinkedBlockingQueue<Runnable>() 无界阻塞队列

应用：只能存活一个线程，一个任务一个任务的执行

```
  ExecutorService executorService= Executors.newSingleThreadExecutor();
        for(int i=0;i<10;i++){
            final int index=i;
          executorService.execute(new Runnable() {
              @Override
              public void run() {
                  System.out.println(index);
                  try {
                      Thread.sleep(3000);
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  }
              }
          });
        }
```

### 5.线程的生命周期

创建（等待），运行，挂起，死亡。四个状态。

sleep() 睡一会儿，并且不会释放锁，睡够了之后进入就绪状态，开始竞争。                  Thread的方法

wait() 需要notify()之后才能竞争   只能在同步方法或同步代码块中使用  ，wait()后会释放锁资源                                      Object的方法

join() 一般用于阻塞，一般阻塞主线程     t1线程执行完之后，再执行主线程

```
public static void main(String[] args) throws InterruptedException
    {
        System.out.println("main start");

        Thread t1 = new Thread(new Worker("thread-1"));
        t1.start();
        t1.join();
        System.out.println("main end");
    }
```

平行的线程一般不会阻塞 比如 t1 t2同时start()

## 9.重入锁

**定义**：重进入是指任意线程在获取到锁之后，再次获取该锁而不会被该锁所阻塞。

如果一个线程获取了锁，当执行之后，正在这时，恰巧这个线程又请求锁，就不需要去排队，直接上锁，state+1 最开始是0 如果两个相同的线程获取了这个锁，state就是2,如果去排队就会发生死锁。  

当锁递归访问一个线程的时候，不发生死锁，这个锁就是重入锁。（也可以理解为一个线程和他的子程序同时运行）。

ReentrantLock 和 sychronized都是可重入锁

```
 public class LoggingWidget extends Widget {
2     public synchronized void Widget.doSomething() {
3         // do something
4     }   // 父类的doSomething方法
5 
6     public synchronized void doSomething() {
7         super.doSomething();
8     }
9 }

```

super调用父类的doSomething方法时，并不会产生父类对象，调用的还是当前类，所以当调用当前类里的doSomething()方法时就顶算获取了两次锁，当第二次调用时，如果不是重入锁，第一次锁没释放，就无法获得锁，第一次的doSomeThing()方法还需要super.doSomething()，所以就会发生死锁。

## 10.死锁

当一组并发进程相互等待的现象。

死锁的四个条件：

1.互斥：一个资源只能被一个进程所占有。

2.不可剥脱条件：进程享受资源时不能被别的进程抢走。

3.请求和保持条件：请求获取资源的时候，不释放手里的资源

4.循环等待条件：若干进程之间形成一种头尾相连的等待状态。

## 11.3个特性

1.内存模型的相关概念：

CPU运行速度较快，对应内存的存储却很慢，就出现了高速缓存。先高速缓存之后，再存到内存中。

当是多核CPU的时候,机会出现线程不安全的问题。

2.多线程高并发下必须满足3个特性

原子性：整个操作要不全部完成，要不就整体失败。 通过synchronized和Reenttrotlock解决

可见性：一个线程对内存中数据更改时，其他线程可见。  通过voliatile关键字   synchronized和Reenttrotlock也可以保证可见性，因为只有一个线程

有序性：代码按照顺序执行，并且不能受JVM的重排序影响 vloliatile,synchronized,Reenttrotlock



## 12.volatile关键字

1.被volatile修饰的变量 有两点：

 1）保证了多个线程对这个变量操作时的可见性，不会有高速缓存中数据和内存中数据对应不上的问题。

 2）禁止进行指令重排序

2.volatile不能保证原子性

3.底层原理：

强制把缓存中的数据写到内存中，还有一个栅栏保证有序性。

4.应用场景：不需要控制原子性的场景。

## 13.synchronized关键字

是一种同步锁，是可重入锁，他修饰的对象有如下几种：

1.修饰同步代码块：其作用的范围是大括号{}括起来的代码，作用的对象是调用这个代码块的对象；

2.同步方法：其作用的范围是整个方法，作用的对象是调用这个方法的对象；

3.修饰一个静态方法：其作用的范围是整个静态方法，作用的对象是这个类的所有对象

4.修饰一个类：其作用的范围是synchronized后面括号括起来的部分，作用主的对象是这个类的所有对象。

例子：

### 1.修饰一个代码块

#### 1.一个线程访问一个对象的同步代码块时，其他线程将会被堵塞。

```
class SyncThread implements Runnable {
   private static int count;
 
   public SyncThread() {
      count = 0;
   }
 
   public  void run() {
      synchronized(this) {
         for (int i = 0; i < 5; i++) {
            try {
               System.out.println(Thread.currentThread().getName() + ":" + (count++));
               Thread.sleep(100);
            } catch (InterruptedException e) {
               e.printStackTrace();
            }
         }
      }
   }
 
   public int getCount() {
      return count;
   }
}
```

```
SyncThread syncThread = new SyncThread();
Thread thread1 = new Thread(syncThread, "SyncThread1");
Thread thread2 = new Thread(syncThread, "SyncThread2");
thread1.start();
thread2.start();  两个线程使用一个对象
```

```
Thread thread1 = new Thread(new SyncThread(), "SyncThread1");
Thread thread2 = new Thread(new SyncThread(), "SyncThread2");
thread1.start();
thread2.start(); 两个线程使用两个对象  分别给两个对象上锁互不影响
```

#### 2.一个线程访问一个对象的同步代码块时，另一个线程可以访问该对象的非同步代码块

```
class Counter implements Runnable{
   private int count;
 
   public Counter() {
      count = 0;
   }
 
   public void countAdd() {
      synchronized(this) {
         for (int i = 0; i < 5; i ++) {
            try {
               System.out.println(Thread.currentThread().getName() + ":" + (count++));
               Thread.sleep(100);
            } catch (InterruptedException e) {
               e.printStackTrace();
            }
         }
      }
   }
 
   //非synchronized代码块，未对count进行读写操作，所以可以不用synchronized
   public void printCount() {
      for (int i = 0; i < 5; i ++) {
         try {
            System.out.println(Thread.currentThread().getName() + " count:" + count);
            Thread.sleep(100);
         } catch (InterruptedException e) {
            e.printStackTrace();
         }
      }
   }
 
   public void run() {
      String threadName = Thread.currentThread().getName();
      if (threadName.equals("A")) {
         countAdd();
      } else if (threadName.equals("B")) {
         printCount();
      }
   }
}

psvm{Counter counter = new Counter();
Thread thread1 = new Thread(counter, "A");
Thread thread2 = new Thread(counter, "B");
thread1.start();
thread2.start();
}


```

### 2.修饰方法

跟修饰代码块类似，但应注意以下几点：

1.synchronized关键字不能继承

2.定义接口时不能用synchronized关键字。

3.构造方法不能使用synchronized关键字同步，但可以使用同步代码块。

4.当两个线程同时访问一个对象里被synchronized修饰的两个方法时，不能同步访问，整个对象都上了锁，必须等第一个方法释放锁之后，第二个方法才能获取锁。

```
public class SychronizedMethod {

    public synchronized void test1() {

        System.out.println("test1 entry");
        System.out.println("test1 out");
    }

    public synchronized void test2() {

        System.out.println("test2 entry");
        try {
            Thread.sleep(10000);
        } catch (InterruptedException ex) {
            ex.printStackTrace();
        }
        System.out.println("test2 out");
    }

    public static void main(String[] args) {

        SychronizedMethod sychronizedMethod = new SychronizedMethod();
        WorkHandler handler = new WorkHandler(sychronizedMethod);
        WorkHandler1 handler1 = new WorkHandler1(sychronizedMethod);
        Thread t1 = new Thread(handler);
        Thread t2 = new Thread(handler1);
        t1.start();
        t2.start();

    }
}


public class WorkHandler1 implements Runnable {

    private SychronizedMethod syn;

    WorkHandler1(SychronizedMethod syn) {
        this.syn = syn;
    }

    @Override
    public void run() {

        syn.test1();
    }
}

public class WorkHandler implements Runnable {

    private SychronizedMethod syn;

    WorkHandler(SychronizedMethod syn) {
        this.syn = syn;
    }

    @Override
    public void run() {

        syn.test2();
    }
}
```

```
test2 entry
test2 out
test1 entry
test1 out
```

### 3.修饰静态方法

这个类的所有对象都会上锁

### 4.修饰一个类

就是给这个类上锁

## 14.synchronized关键字与Reentranklock的区别

1.synchrinized是java内置关键字，是jvm层面的，Reentranklock是java类

2.Reentranklock可以获取锁的状态，synchronized不能

```
Lock lock = new ReentrantLock()
lock.tryLock();//尝试获取锁，如果获取到了就返回true;
```

3.synchronized会自动释放锁，Reentranklock得调用unlock()方法手动释放。

4.用synchronized关键字的两个线程1和线程2，如果当前线程1获得锁，线程2线程等待。如果线程1阻塞，线程2则会一直等待下去，而Lock锁就不一定会等待下去，如果尝试获取不到锁，线程可以不用一直等待就结束了；

```
lock.tryLock(3000, TimeUnit.MILLISECONDS)//尝试获取锁，获取不到就返回false.
```

5.synchronized可重入，不可中断，非公平。RennttrankLock不可重入，可中断，可公平。

6.synchronized适合小量同步代码，ReentrankLock适合大量同步代码

## 15.AQS

AbstractQueueSynchronizer抽象同步队列，由共享资源和FIFO线程等待队列构成，定义了多线程访问共享资源的框架，Semaphore,CountDounLoatch,CyclicBarrier中都有使用。(就是对共享资源进行原子操作)

### 1.共享资源共享方式：

1.独占：（ReentrantLock）

2.共享：(CountDownLoatch,Semaphore)

### 2.自定义同步器：

1.自定义同步器只需要实现共享资源state的获取和释放即可,队列的维护AQS已经封装好了，自定义同步器只需要实现下面方法即可。

```
isHeldExclusively()：该线程是否正在独占资源。只有用到condition才需要去实现它。
tryAcquire(int)：独占方式。尝试获取资源，成功则返回true，失败则返回false。
tryRelease(int)：独占方式。尝试释放资源，成功则返回true，失败则返回false。
tryAcquireShared(int)：共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
tryReleaseShared(int)：共享方式。尝试释放资源，如果释放后允许唤醒后续等待结点返回true，否则返回false。
```

## 16Cas算法

1.Cas保证无锁状态下的安全，属于乐观锁，总觉得没事儿

```
CAS(V,E,N)     v,变量      E,预期值         N，新值
```

 如果v=e,则v=n;如果v!=e,则说明其他线程已经做了更新，当前线程什么也不做。

当多个线程操作一个变量时，只有一个线程会胜出，相当于加锁。

2.原子操作类,在atomic包下

```
AtomicBoolean  原子更新
AtomicInteger
AtomicLong
```

3.ABA问题

![img](https://img-blog.csdn.net/20170702223304481?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvamF2YXplamlhbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

虽然无伤大雅但也可以防止的

**AtomicStampedReference**类每次修改后都有一个时间戳

## 17.自旋锁，轻量级锁，重量级锁，偏向锁

# 数据库

# 数据与信息传输

## OSI网络七层协议

OSI是一个[开放性](https://baike.baidu.com/item/%E5%BC%80%E6%94%BE%E6%80%A7/3129237)的通信系统互连参考模型，他是一个定义得非常好的协议规范。OSI模型有7层结构，每层都可以有几个子层。 OSI的7层从上到下分别是 7 [应用层](https://baike.baidu.com/item/%E5%BA%94%E7%94%A8%E5%B1%82/4329788) 6 [表示层](https://baike.baidu.com/item/%E8%A1%A8%E7%A4%BA%E5%B1%82/4329716) 5 会话层 4 [传输层](https://baike.baidu.com/item/%E4%BC%A0%E8%BE%93%E5%B1%82/4329536) 3 网络层 2 [数据链路层](https://baike.baidu.com/item/%E6%95%B0%E6%8D%AE%E9%93%BE%E8%B7%AF%E5%B1%82/4329290) 1 [物理层](https://baike.baidu.com/item/%E7%89%A9%E7%90%86%E5%B1%82/4329158) ；其中高层（即7、6、5、4层）定义了应用程序的功能，下面3层（即3、2、1层）主要面向通过网络的端到端的[数据流](https://baike.baidu.com/item/%E6%95%B0%E6%8D%AE%E6%B5%81/3002243)。

1.物理层：将数据转化为物理信号，负责计算机和网络媒体的互通。例：电压，网卡。

2.数据链路层：决定网络介质的方式，在此层将数据分帧，并处理流的控制。例如：以太网，WI-FI

3.网络层：通过IP寻址。

4.传输层：可靠的传输服务，实现端到端的连接。例：TCP,UDP协议

5.会话层：允许使用简单易记的名称建立连接。

6.表示层：协商系统数据交换格式，因为系统与系统之间数据格式不同

7.应用层：软件与软件之间的通信：例如Http,SMTP协议。

## TCP

![img](https://www.centos.bz/wp-content/uploads/2012/08/100327002629.png)

```
syn ack  seq

1                 x

1      x+1     y

​         y+1    z
```

连接的时候使用三次握手，关闭的时候使用4次挥手

![img](http://www.centos.bz/wp-content/uploads/2012/08/100327022731.jpg)

```
fin      ack           seq

1           z                x

​              x+1            z

1            x                y

​             y                 x
```

TCP和UDP的区别：

```
1.基于连接与无连接；
2.对系统资源的要求（TCP较多，UDP少）；
3.UDP程序结构较简单；
4.流模式与数据报模式 ；

5.TCP保证数据正确性，UDP可能丢包，TCP保证数据顺序，UDP不保证。
```

## RPC(Remote Procedure Call)远程过程调用

在OSI[网络通信](https://baike.baidu.com/item/%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1/9636548)模型中，RPC跨越了[传输层](https://baike.baidu.com/item/%E4%BC%A0%E8%BE%93%E5%B1%82/4329536)和[应用层](https://baike.baidu.com/item/%E5%BA%94%E7%94%A8%E5%B1%82/4329788)。

RPC采用客户端/服务端模式，请求程序就是一个客户机，客户机调用进程发送参数到服务器，服务器对其计算处理，然后发送给客户端。

## web service

跨编程语言和操作系统的远程调用技术，慢慢摒弃RPC

## Restful

每个服务都有一个URI，各服务直接通过URI调用，并不把客户端，服务端分的那么细。

## WebSocket

WebSocket是一种网络通讯协议，是长连接。

http协议有一个缺陷，只能是客户端调用服务端。

WebSocket协议的服务器可以主动向客户端推送消息，比如聊天室。

## MQ

### 1.为什么使用mq?

1.在高并发情况下，来不及进行同步处理，容易进行堵塞，导致错误。使用同步队列，可以进行异步处理，环节系统压力。

### 2.消息中间件的组成

1.Broker(消息服务器)

2.producer

3.consumer

4.Topic 订阅模式下消息的统一汇集地，不同的生产者向Topic发送消息，由MQ服务器发送到不同的订阅者。

5.queue  队列，PTP模式下，生产者指定特定的queue发送消息，消费者订阅特定的queue订阅消息。

5.Message 消息体 根据不同统一协议进行编码的数据包。

### 3.模式分类

#### 1.点对点

![img](https://leanote.com/api/file/getImage?fileId=5ad56d7cab64411333000bb0)

1.使用queue作为通信载体。

2.消息被消费后，queue中不会存储，所以一个消息只能被一个消费者所消费。消费者接收到的每一条消息都必须进行确认。如果消息接受了，在确认之前从rabbitmq断开连接，队列中将不会删除此消息。

#### 2.发布/订阅

![img](https://leanote.com/api/file/getImage?fileId=5ad56d8bab6441153c000ab3)

订阅该消息的消费者都会得到该消息

### 4.消息中间件的优势

1.系统解耦：系统之间没有调用关系，只是消息发送

2.解决高并发问题。

3.异步通信

4.顺序执行

5.消息会被持久化

### 5.几种MQ

![img](https://img-blog.csdn.net/20170816171523564?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvb01hdmVyaWNrMQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

综合选择rabbitmq

### 6.RabbitMQ

用Erlang语言编写

#### 1.为什么使用RabbitMQ

```
除了Qpid，RabbitMQ是唯一一个实现了AMQP标准的消息服务器；
可靠性，RabbitMQ的持久化支持，保证了消息的稳定性；
高并发，RabbitMQ使用了Erlang开发语言，Erlang是为电话交换机开发的语言，天生自带高并发光环，和高可用特性；
集群部署简单，正是应为Erlang使得RabbitMQ集群部署变的超级简单；
社区活跃度高，根据网上资料来看，RabbitMQ也是首选；
```

#### 2.消息发送原理

首先你必须连接到Rabbit才能发布和消费消息，那怎么连接和发送消息的呢？

你的应用程序和Rabbit Server之间会创建一个TCP连接，一旦认证通过你的应用程序和Rabbit就创建了一条AMQP信道（Channel）。信道是创建在“真实”TCP上的虚拟连接(因为用TCP太费资源)，AMQP命令都是通过信道发送出去的，每个信道都会有一个唯一的ID，不论是发布消息，订阅队列或者介绍消息都是通过信道完成的。

AMQP，即 Advanced Message Queuing Protocol，高级消息队列协议,AMQP 的主要特征是面向消息、队列、路由（包括点对点和发布/订阅）、可靠性、安全。

#### 3.RabbitMQ的名词

1.**ConnectionFactory(连接管理器)** ：应用程序与RabbitMQ建立连接，程序中用。

2.**Channel(信道)**:消息推送使用的通道。

3.**Exchange(交换器)**:用于接收和交换信息。

4.**Queue（队列）**：用于存储生产者的消息；

5**.虚拟主机**：一个虚拟主机持有一组交换机、队列和绑定。为什么需要多个虚拟主机呢？很简单， RabbitMQ 当中，*用户只能在虚拟主机的粒度进行权限控制。* 因此，如果需要禁止A组访问B组的交换机/队列/绑定，必须为A和B分别创建一个虚拟主机。每一个 RabbitMQ 服务器都有一个默认的虚拟主机“/”。

6**.交换机**：*Exchange 用于转发消息，但是它不会做存储* ，如果没有 Queue bind 到 Exchange 的话，它会直接丢弃掉 Producer 发送过来的消息。 这里有一个比较重要的概念：**路由键** 。消息到交换机的时候，交互机会转发到对应的队列中，那么究竟转发到哪个队列，就要根据该路由键。

 1.Direct Exchange 

是rabbitMQ默认的根据key去匹配队列**

![img](http://www.itmind.net/assets/images/2016/rabbitMq_direct.png)

​     当消息的路由键=orange时就会被绑定到Q1

2.Topic Exchange

```
转发消息主要是根据通配符。* 在这种交换机下，队列和交换机的绑定会定义一种路由模式，那么，通配符就要在这种路由模式和路由键之间匹配后交换机才能转发消息。

在这种交换机模式下：

- 路由键必须是一串字符，用句号（`.`） 隔开，比如说 agreements.us，或者 agreements.eu.stockholm 等。
- 路由模式必须包含一个 星号（`*`），主要用于匹配路由键指定位置的一个单词，比如说，一个路由模式是这样子：agreements..b.*，那么就只能匹配路由键是这样子的：第一个单词是 agreements，第四个单词是 b。 井号（#）就表示相当于一个或者多个单词，例如一个匹配模式是 agreements.eu.berlin.#，那么，以agreements.eu.berlin 开头的路由键都是可以的。
opic 和 direct 类似**, 只是匹配上支持了”模式”, 在”点分”的 routing_key 形式中, 可以使用两个通配符:

- `*`表示一个词.
- `#`表示零个或多个词.
```

具体代码发送的时候还是一样，第一个参数表示交换机，第二个参数表示 routing key，第三个参数即消息。如下：

```
rabbitTemplate.convertAndSend("testTopicExchange","key1.a.c.key2", " this is  RabbitMQ!");
```

3.Headers Exchange

```
headers 也是根据规则匹配, 相较于 direct 和 topic 固定地使用 routing_key , headers 则是一个自定义匹配规则的类型. 在队列与交换器绑定时, 会设定一组键值对规则, 消息中也包括一组键值对( headers 属性), 当这些键值对有一对, 或全部匹配时, 消息被投送到对应队列.
```

4.Fanout Exchange

```
Fanout Exchange 消息广播的模式，不管路由键或者是路由模式，*会把消息发给绑定给它的全部队列*，如果配置了 routing_key 会被忽略。
```

```
*实际上，用三种 Exchange 都可以实现点对点与发布订阅模型。**

**点对点模型：**

- **direct Exchange：创建队列A，通过任意绑定键绑定到 Exchange，消息发送使用相同的绑定键**
- **fanout Exchange：创建队列A，绑定到 Exchange**
- **topic Exchange：创建队列A，通过任意绑定键绑定到 Exchange，消息发送使用相同的绑定键**

**发布订阅模型：**

- **direct Exchange：创建多个队列，通过为每个队列设定多个绑定，也能实现相对复杂的发布订阅模型**
- **fanout Exchange：创建多个队列，绑定到 Exchange，这是简化的发布订阅模型**
- **topic Exchange：创建多个队列，通过带通配符的绑定键实现复杂而又灵活的发布订阅模型**
```

**AMDQ中实现的都是点对点通信，发布订阅实际上是exchange将信息复制到个队列来实现的**

7.**绑定**：也就是交换机需要和队列相绑定，这其中如上图所示，是多对多的关系。

![img](http://www.itmind.net/assets/images/2016/RabbitMQ01.png)

#### 4.消息持久化

默认情况下，重启RabbitMQ会使消息丢失。

##### 1.设置持久化

```
投递消息的时候durable设置为true，消息持久化，代码：channel.queueDeclare(x, true, false, false, null)，参数2设置为true持久化；
设置投递模式deliveryMode设置为2（持久），代码：channel.basicPublish(x, x, MessageProperties.PERSISTENT_TEXT_PLAIN,x)，参数3设置为存储纯文本到磁盘；
消息已经到达持久化交换器上；
消息已经到达持久化的队列；
```

##### 将消息写入到磁盘中的持久化日志文件，当消息被消费后，rabbir会把消息作为等待垃圾回收。

但是消息写入磁盘会降低性能。

#### 5.虚拟主机vhost

每个rabbit会产生多个vhost

1. RabbitMQ默认的vhost是“/”开箱即用；
2. 多个vhost是隔离的，多个vhost无法通讯，并且不用担心命名冲突（队列和交换器和绑定），实现了多层分离；
3. 创建用户的时候必须指定vhost；

```
rabbitmqctl工具命令创建：rabbitmqctl add_vhost[vhost_name]
删除:rabbitmqctl delete_vhost[vhost_name]
查看所有:rabbitmqctl list_vhosts
```

#### 6.使用RabbitMQ

##### 1.简单使用

1.启动rabbitMQ:直接登录网址localhost:15672输入用户名密码guest即可开启。

2.默认基础配置

```
spring:
  application:
    name: Spring-boot-rabbitmq
  rabbitmq:
    password: guest
    port: 5672
    host: localhost
    username: guest
```

3.pom

```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

4.队列配置

```
@Configuration
public class RabbitConfig {

    @Bean
    public Queue Queue() {
        return new Queue("hello");
    }
}
```

这是初始化配置,Springboot启动就会加载。

4.发送者与接收者

```
@component    将类注入到容器
public class HelloSender {

	@Autowired
	private AmqpTemplate rabbitTemplate; //Springboot提供的rabbitMQ支持

	public void send() {
		String context = "hello " + new Date();
		System.out.println("Sender : " + context);
		this.rabbitTemplate.convertAndSend("hello", context);
	}                                      队列名称    消息内容

}
```

```
@Component  //接收者也必须注入   ，可能rabbit框架会调用这个类。
@RabbitListener(queues = "hello")      //队列名称
public class HelloReceiver {

    @RabbitHandler
    public void process(String hello) {
        System.out.println("Receiver  : " + hello);
    }

}
```

5.测试

```
@RunWith(SpringRunner.class)
@SpringBootTest
public class RabbitMqHelloTest {

	@Autowired
	private HelloSender helloSender;

	@Test
	public void hello() throws Exception {
		helloSender.send();
	}
}
```

注意：

1.发送者和接收者的 queue name 必须一致，不然不能接收

2.一个发送者，N个接受者,经过测试会均匀的将消息发送到N个接收者中

3.N个发送者，N个接受者,经过测试会均匀的将消息发送到N个接收者中

4.Spring Boot 以及完美的支持对象的发送和接收，不需要格外的配置

##### 2.使用TopicExchange

```
@Configuration
public class TopicRabbitConfig {

    final static String message = "topic.message";
    final static String messages = "topic.messages";

    @Bean
    public Queue queueMessage() {
        return new Queue(TopicRabbitConfig.message);
    }

    @Bean
    public Queue queueMessages() {
        return new Queue(TopicRabbitConfig.messages);
    }

    @Bean
    TopicExchange exchange() {
        return new TopicExchange("exchange");
    }

    @Bean
    Binding bindingExchangeMessage(Queue queueMessage, TopicExchange exchange) {
        return BindingBuilder.bind(queueMessage).to(exchange).with("topic.message");
    }

    @Bean
    Binding bindingExchangeMessages(Queue queueMessages, TopicExchange exchange) {
        return BindingBuilder.bind(queueMessages).to(exchange).with("topic.#");
    }
}
```

```
public void send1() {
	String context = "hi, i am message 1";
	System.out.println("Sender : " + context);
	this.rabbitTemplate.convertAndSend("exchange", "topic.message", context);
}

public void send2() {
	String context = "hi, i am messages 2";
	System.out.println("Sender : " + context);
	this.rabbitTemplate.convertAndSend("exchange", "topic.messages", context);
}
```

```
@Component
@RabbitListener(queues = "topic.message")
public class HellpRecerver3 {
    @RabbitHandler
    public void process(String ss){
        System.out.println("接收者33333接收到了"+ss);
    }
}
```

```
@Component
@RabbitListener(queues = "topic.messages")
public class HelloReceiver4 {
    @RabbitHandler
    public void process(String ss){
        System.out.println("接收者444444接收到了"+ss);
    }
}
```

用这种方法使用DirectExchange不知道为啥不好使

##### 3.使用FanoutExchange

熟悉的广播模式和订阅模式。给 Fanout 交换机发送消息，绑定了这个交换机的所有队列都收到这个消息。

```
@Configuration
public class FanoutRabbitConfig {

    @Bean
    public Queue AMessage() {
        return new Queue("fanout.A");
    }

    @Bean
    public Queue BMessage() {
        return new Queue("fanout.B");
    }

    @Bean
    public Queue CMessage() {
        return new Queue("fanout.C");
    }

    @Bean
    FanoutExchange fanoutExchange() {
        return new FanoutExchange("fanoutExchange");
    }

    @Bean
    Binding bindingExchangeA(Queue AMessage,FanoutExchange fanoutExchange) {
        return BindingBuilder.bind(AMessage).to(fanoutExchange);
    }

    @Bean
    Binding bindingExchangeB(Queue BMessage, FanoutExchange fanoutExchange) {
        return BindingBuilder.bind(BMessage).to(fanoutExchange);
    }

    @Bean
    Binding bindingExchangeC(Queue CMessage, FanoutExchange fanoutExchange) {
        return BindingBuilder.bind(CMessage).to(fanoutExchange);
    }

}
```

```
public void send() {
	String context = "hi, fanout msg ";
	System.out.println("Sender : " + context);
	this.rabbitTemplate.convertAndSend("fanoutExchange","", context);
}
```

这里使用了 A、B、C 三个队列绑定到 Fanout 交换机上面，发送端的 routing_key 写任何字符都会被忽略：

```
@Component
@RabbitListener(queues = "fanout.A")
public class HellpRecerver7 {
    @RabbitHandler
    public void process(String ss){
        System.out.println("接收者fanout.A接收到了"+ss);
    }
}
@Component
@RabbitListener(queues = "fanout.B")
public class HellpRecerver7 {
    @RabbitHandler
    public void process(String ss){
        System.out.println("接收者fanout.A接收到了"+ss);
    }
}
@Component
@RabbitListener(queues = "fanout.C")
public class HellpRecerver7 {
    @RabbitHandler
    public void process(String ss){
        System.out.println("接收者fanout.A接收到了"+ss);
    }
}

```

#### 7.使用的大致步骤

1.注入 队列和交换器，路由键的关系     

2.写发送者 注明路由键和交换器

3.标清队列名

#### 8.问题

##### 1.使用DirectExchange时候没好使

##### 2.消息持久化的简易操作

##### 3.如何确保rabbitmq接收到消息，确保消费者接收到消息？

发送方确认模式：将信道设置成confirm模式(发送方确认方式)，所有在信道上发送的消息会被指定一个唯一ID,

当消息进入队列，或者存入磁盘，信道就会发送一个确认(包括生成的唯一ID)给生产者。如果发生错误，回产生一个nack信息。

接收方确认机制：消费者得到每一条消息后都会做一个确认，只有消费者确认了消息，rabbitmq才会把队列中的消息清除。

下面罗列几种特殊情况：
如果消费者接收到消息，在确认之前断开了连接或取消订阅，RabbitMQ会认为消息没有被分发，然后重新分发给下一个订阅的消费者。（可能存在消息重复消费的隐患，需要去重）

如果消费者接收到消息却没有确认消息，连接也未断开，则RabbitMQ认为该消费者繁忙，将不会给该消费者分发更多的消息。

##### 4.如何保证消息不被重复消费？

在消息生产时，rabbitmq会为每一条消息产生一个inner-msg-id,作为去重的依据。

在消息消费时，每一个消息体中应有一个bizID,作为去重的依据。

##### 5.rabbitmq集群

集群镜像模式：无论是元数据还是queue里的消息都会存在于多个势力上，防止一个宕机，消息丢失。

坏处就是，开销太大。

##### 6.保证消息100%投递。

方案一：消息入库，对消息进行打标。

​              将消息存入数据库,然后通过轮询操作，固定时间访问数据库，重新发送没发送的消息，

方案二：消息进行延迟投递，进行二次确认，回调检查。(表示不懂)

整个流程需要三个消息队列来完成。

step1：先业务数据入库，然后发送业务消息。
step2：延迟发送第二条check消息。（延迟时间根据业务来确定）
step3：消费端监听队列并消费消息。
step4：消费端发送confirm消息。（此消息为从新发送的新消息在不同的队列中，非ack签收）
step5：回调服务监听消费端的confirm消息队列，并将消息进行持久化存储。

step6：回调服务监听check消息队列，查询数据库，如果业务消息没有消费，回调服务将告诉producer从新发送业务消息作者重新发送消息

#### 9.Broker

做消息存储。

1、commitLog的存储是producer发送消息给broker端broker同步处理的

2、consumeQueue和index两者存储其实是一个定时任务从commitLog中获取偏移量然后存储过去的

3、consumeQueue和index的存储 与 commitLog的存储是隔离开的，非同步的

Rocketmq的消息的存储是由consume queue和 commitLog 配合完成的

# 包结构

![1555664689336](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1555664689336.png)

# 公共类

## BeanUtils

 1.两个相同类型的类，把第一个的内容赋值给第二个

```
BeanUtils.copyProperties(securityHoldingInfoVO,securityHoldingInfo);
```

