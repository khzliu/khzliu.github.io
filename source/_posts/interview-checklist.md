---
title: iOS面试题总结
date: 2016-06-23 22:52:03
tags: [面试]
categories: 日常记事

---

# 常见的iOS面试题
由于空闲时间比较喜欢在网上看一些大牛的技术blog，今天刚好看到大牛sunnyxx的一个关于他面试别人时提到的一些常见问题（[原文地址](http://blog.sunnyxx.com/2015/07/04/ios-interview/)），所以自己也就试着回答了一遍，也算时对自己的一次面试吧，通过自己面试时间也发现了一些自己不足的一面，所以今天记录下来，加深一下印象吧。<br/>

<font color="red"><strong>另外这里有一份 [详细答案](https://github.com/ChenYilong/iOSInterviewQuestions/blob/master/01%E3%80%8A%E6%8B%9B%E8%81%98%E4%B8%80%E4%B8%AA%E9%9D%A0%E8%B0%B1%E7%9A%84iOS%E3%80%8B%E9%9D%A2%E8%AF%95%E9%A2%98%E5%8F%82%E8%80%83%E7%AD%94%E6%A1%88/%E3%80%8A%E6%8B%9B%E8%81%98%E4%B8%80%E4%B8%AA%E9%9D%A0%E8%B0%B1%E7%9A%84iOS%E3%80%8B%E9%9D%A2%E8%AF%95%E9%A2%98%E5%8F%82%E8%80%83%E7%AD%94%E6%A1%88%EF%BC%88%E4%B8%8A%EF%BC%89.md) 分析的很到位</strong></font>

### [※]@property中有哪些属性关键字？
在@property 中的关键词个人总结分为三类：RC语意修饰符，线程安全修饰符，读写属性修饰符。
#### RC语意修饰符
这类修饰符主要为了在ARC或者MRC下一个属性的内存管理方法，共如下几种：<br/>
`assign`： 简单的赋值，不会改变引用计数，共享内存；一般用于基本类型（c/c++：int float等，OC:NSInteger,BOOL等）<br/>
`retain`： 处理简单赋值外会对赋值对象的引用计数加1操作，用于OC对象；<br/>
`copy`：处理简单赋值，但是赋值对象引用计数不变，被赋值对象引用计数为1，也就是新生成一个与被赋值对象一样的数据（非绝对 如NSArray对象）<br/>
`unsafe_unretained`：简单赋值，与assign相似，多用于修饰OC对象，只是表显该对象是不安全的（当赋值对象变为nil的时候会引起crash 野指针问题）<br/>
`strong`：ARC中简单赋值，与非ARC下的retain相同作用<br/>
`weak`：ARC中简单赋值，与非ARC下的assign对应，但是又有不同，weak多用于修饰OC对象，且当改对象的引用计数为0的时候，改对象会被修改成nil，而assign则不会，好处时能有效的防止野指针<br/>

#### 线程安全修饰符
这类修饰符主要是指定属性赋值操作的线程安全性<br/>
`atomic`：线程安全的，多线程访问情况下只有一个线程能访问他，影响访问速度<br/>
`nonatomic`：非线程安全的，多线程可同时读写，容易造成混乱，但读写速度快<br/>

#### 读写属性修饰符
这类修饰符主要控制属性的读写权限<br/>
`readonly`: 编译器只会对该属性生成get方法<br/>
`readwrite`: 编译器能生成get/set方法，默认此属性<br/>
另外get方法可以在修饰符最后指定，比如：`@property(getter = isGirl) BOOL girl`

### [*]怎么用 copy 关键字？
1. NSString，NSArray，NSDictionary类型属性常用，因为它们有NSMutableString,NSMutableArray,NSMutableDictionary类型属性，当我们不想外界干扰我们正在操作的这类型属性的时候，我们会把她们设置成copy。
2. 在使用block的时候我们用copy，在MRC时代，block是创建在栈上的，所以当我们把block进行赋值时不进行copy操作的话，block内的变量会因为栈的pop而造成野指针，从而引起crash。当然在ARC下，block会自动进行copy，但是作为程序员团队协作开发，能明确让别人看到block是经过copy到堆上的对象则更加具有易读性。
### [*]这个写法会出什么问题： @property (copy) NSMutableArray *array;
1. 把NSMutableArray 属性设置成copy，无论赋值对象是NSArray还是NSMutableArray赋值后array指向的是NSArray对象，但我们往往会因为array的声明NSMutableArray＊ 而把它当成一个可变对象来操作，一旦调用了array的[array removeAtIndex:n],等不属于NSArray的方法后会引起Crash。
2. 属性修饰符没有加nonatomic，那么array会被编译器默认设置为atomic，atomic表示该属性是原则性操作，每次setter都会加锁来确保它的原子性，这将耗费大量运算。

### [*]如何让自己的类用 copy 修饰符？如何重写带 copy 关键字的 setter？

想要自己的类能够使用copy修饰符则必须让自己的类实现NSCopying协议，如果自己的类像NSString那样大小有可变版本和不可变版本，那么就是实现NSCopying协议和NSMutalbeCopying协议。

具体步骤：
1. 声明该类遵从NSCopying协议。
2. 在该类中实现下面方法,并且注意对象copy是否需要深拷贝操作。
	
	- (id)copyWithZone:(NSZone *)zone;
	
至于重写带copy关键字的setter的话一般如下：

	- (void)setCustomClass:(CustomClass *)customClass
	{
		_customClass = [customClass copy];
	}
但是由于遵循NSCopying协议的类一般都是用作只读属性，所以我们就不去实现setter方法，而是在类initializer的时候对其进行初始化。例如：
	
	- (instancetype)initWithCustomClass:(CustomClass *)cClass
	{
		if(self == [super init]){
			_cClass = [cClass copy];
		}
		return self;
	}
	

### [*]@property 的本质是什么？ivar、getter、setter 是如何生成并添加到这个类中的?
<font color="red" size="5px"><strong>此为盗取答案：</strong></font>
#### @property 的本质是什么？
> @property = (ivar + getter + setter)
> “属性” (property)有两大概念：ivar（实例变量）、存取方法（access method ＝ getter + setter）。

“属性” (property)作为 Objective-C 的一项特性，主要的作用就在于封装对象中的数据。 Objective-C 对象通常会把其所需要的数据保存为各种实例变量。实例变量一般通过“存取方法”(access method)来访问。其中，“获取方法” (getter)用于读取变量值，而“设置方法” (setter)用于写入变量值。这个概念已经定型，并且经由“属性”这一特性而成为 Objective-C 2.0 的一部分。 而在正规的 Objective-C 编码风格中，存取方法有着严格的命名规范。 正因为有了这种严格的命名规范，所以 Objective-C 这门语言才能根据名称自动创建出存取方法。其实也可以把属性当做一种关键字，其表示:

> 编译器会自动写出一套存取方法，用以访问给定类型中具有给定名称的变量。 所以你也可以这么说：
> @property = getter + setter;

#### ivar、getter、setter 是如何生成并添加到这个类中的?

> “自动合成”( autosynthesis)
> 完成属性定义后，编译器会自动编写访问这些属性所需的方法，此过程叫做“自动合成”(autosynthesis)。需要强调的是，这个过程由编译 器在编译期执行，所以编辑器里看不到这些“合成方法”(synthesized method)的源代码。除了生成方法代码 getter、setter 之外，编译器还要自动向类中添加适当类型的实例变量，并且在属性名前面加下划线，以此作为实例变量的名字。在前例中，会生成两个实例变量，其名称分别为 _firstName 与 _lastName。也可以在类的实现代码里通过 @synthesize 语法来指定实例变量的名字.

	@implementation Person
	@synthesize firstName = _myFirstName;
	@synthesize lastName = _myLastName;
	@end
	
> 我为了搞清属性是怎么实现的,曾经反编译过相关的代码,他大致生成了五个东西

> 1. OBJC_IVAR_$类名$属性名称 ：该属性的“偏移量” (offset)，这个偏移量是“硬编码” (hardcode)，
> 2. 表示该变量距离存放对象的内存区域的起始地址有多远。
> 3. setter 与 getter 方法对应的实现函数
> 4. ivar_list ：成员变量列表
> 5. method_list ：方法列表
> 6. prop_list ：属性列表
> 
> 也就是说我们每次在增加一个属性,系统都会在 ivar_list 中添加一个成员变量的描述,在 method_list 中增加 setter 与 getter 方法的描述,在属性列表中增加一个属性的描述,然后计算该属性在对象中的偏移量,然后给出 setter 与 getter 方法对应的实现,在 setter 方法中从偏移量的位置开始赋值,在 getter 方法中从偏移量开始取值,为了能够读取正确字节数,系统对象偏移量的指针类型进行了类型强转.

### [*]@protocol 和 category 中如何使用 @property ?

`@property`是封装数据的方式，我们一般不再`@protocol`和`category`中使用`@property`，而是在`class_continuation分类`中, 查看OC源码我们知道objc_struct结构中，成员变量的偏移量是放到`ivar_list`中的，是指针加偏移量来进行寻址操作的,OC把实例变量当作一种存储偏移量所用的“特殊变量”，并交由“类对象”来管理，偏移量会在运行时进行查找，如果类的定义变了，那么存储的偏移量也就变了，这样的话，无论何时访问实例变量，总能使用正确的偏移量，也就是说这些偏移量是是在编译时期生成的，所以不能在运行时期向类内加入实例变量。这和方法列表`method_list`不同的是，方法列表中结构中存的不是偏移量，而是方法指针的指针，也就是说每个方法我们可以修改其实现为任意实现地址。<br/>
虽然我们不能通过在`@protocol`和`category`中添加实例变量，但是我们可以通过`@property`+`@dynamic`+`关联`来达到新增实例相同的结果。

> 参考资料：https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/CustomizingExistingClasses/CustomizingExistingClasses.html



1. 在 protocol 中使用 property 只会生成 setter 和 getter 方法声明,我们使用属性的目的,是希望遵守我协议的对象能实现该属性
2. category 使用 @property 也是只会生成 setter 和 getter 方法的声明,如果我们真的需要给 
category 增加属性的实现,需要借助于运行时的两个函数：

	objc_setAssociatedObject
	objc_getAssociatedObject

Runtime给我们提供了这两个方法来达到这个目的，这些关联是通过一个`AssociationsManager`实例维护一个哈西列表`AssociationsHashMap`实现的。

### [※]weak属性需要在dealloc中置nil么？
不需要，因为被weak修饰符修饰的属性当其引用计数为0的时候，自动引用计数器会自动把属性设置为nil

### [※※]@synthesize和@dynamic分别有什么作用？

`@synthesize`: 该修饰符告诉编译器在编译期自动为该属性生成getter/setter方法，并可指明该属性对应的成员变量。
`@dynamic`: 告诉编译器，不自动生成getter/setter方法，避免编译期间产生警告,然后由自己实现存取方法或存取方法在运行时动态创建绑定：主要使用在CoreData的实现NSManagedObject子类时使用，由Core Data框架在程序运行的时动态生成子类属性
### [※※※]ARC下，不显式指定任何属性关键字时，默认的关键字都有哪些？
内存管理默认`strong`,  原子性默认`atomic`,读写性默认`readwrite`;
### [※※※]用@property声明的NSString（或NSArray，NSDictionary）经常使用copy关键字，为什么？如果改用strong关键字，可能造成什么问题？
对源头是NSMutableString的字符串，strong仅仅是指针引用，增加了引用计数器，这样源头改变的时候，用这种strong方式声明的变量（无论被赋值的变量是可变的还是不可变的），它也会跟着改变;而copy声明的变量，它不会跟着源头改变，它实际上是深拷贝的“备份”。<br/>
对源头是NSString的字符串，无论是strong声明的变量还是copy声明的变量，当第二次源头的字符串重新指向其它的地方的时候，它还是指向原来的最初的那个位置（因为常量字符串分配在一个常量内存区），也就是说其实二者都是指针引用，也就是浅拷贝。<br/>

而对于NSMutableArray和NSMutableDictionary情况是和NSMutableString是一样的，strong情况下是传引用，并且引用计数加1，copy情况下是产生一个深拷贝“备份”，并且这个备份变成了一个不可变的NSArray或NSDictionary<br/>
对于NSArray和NSDictionary在strong情况下传引用，并且引用计数加1，copy情况下是产生一个深拷贝“备份”，并且这个备份变成了一个不可变的NSArray或NSDictionary。<br/>

对于NSString，NSArray，NSDictionary我们经常用copy关键字是因为这些类型的对象不可变，我们只需要使用其一个备份就可以。另外，NSMutableString、NSMutableArray、NSMutableDictionary，他们之间可能进行赋值操作，为确保对象中的字符串值不会无意间变动，应该在设置新属性值时拷贝一份，所以我们也会用copy修饰符。<br/>
如果改用strong关键字修饰的话，试图去修改这些不可改变大小的对象而参数数据混乱（更改了一个原不想修改的数据）。
### [※※※]@synthesize合成实例变量的规则是什么？假如property名为foo，存在一个名为_foo的实例变量，那么还会自动合成新变量么？
> 引用答案的一句话：` 实例变量 == 成员变量 == ivar `

`@synthesize` 合成变量的规则是绑定成员属性getter/setter方法到某个成员变量，这个过程叫自动合成，你可以指定成员变量，也可以不指定，如果不指定，则这个成员变量的名字与属性名字相同，在没有`@synthesize`绑定情况下系统会自动生成一个“_属性名”的一个成员变量，并绑定getter/setter方法到“_属性名”成员变量。如果已经存在“_属性名”，则编译器不会再自动合成新变量。

例如.h文件,用`@property`声明了一个FooClass类型的成员属性 `foo`：<br/>
	
	class FooLcass : NSObject
	{
		FooClass* _foo; //实例变量
	}

	@property (strong, nonatomic) FooClass* foo; //成员属性
	
在.m代码如下：

	@synthesize foo = _foo;


### [※※]什么情况下不会autosynthesis（自动合成）?
1. 同时重写了 setter 和 getter 时
2. 重写了只读属性的 getter 时
3. 使用了 @dynamic 时
4. 在 @protocol 中定义的所有属性
5. 在 category 中定义的所有属性
6. 重载的属性当你在子类中重载了父类中的属性，你必须 使用 @synthesize 来手动合成ivar。

### [※※※※※]在有了自动合成属性实例变量之后，@synthesize还有哪些使用场景？

1. 手动合成ivar（手动管理property，不使用自动合成autosynthesis）
2. 重载父类的属性 当子类重载了父类的属性石，必须手动指定ivar
3. 以在类的实现代码里通过 @synthesize 语法来指定实例变量的名字

### [※※]objc中向一个nil对象发送消息将会发生什么？
之前理解如果objc是nil，那么向nil对象发生消息将什么也不会发生（对象方法没有正常执行并且也没有引起crash），这些虽然从表象上看是正确的，但是在运行时Runtime中却发生了一系列变化。首先Runtime会动态转化方法调用为消息发送，调用objc_msgSend(receiver,selecter,argv,...),objc_msgSend是不会返回值，他只是起到消息转发的作用，整个过程就是objc_Send根据reciver找到该实例对象的类对象，然后从该类对象的方法列表objc_method_list中找到对应的IMP，然后把参数传给IMP进行方法调用，最后把IMP函数指针指定的方法只需结果返回。由于receiver参数是nil，也就是该类对象的isa指针指向为nil(0)，objc_msgSend也就返回nil(0).

另外附上objc的一个结构定义

	// runtime.h（类在runtime中的定义）
	// http://weibo.com/luohanchenyilong/
	// https://github.com/ChenYilong
	
	struct objc_class {
	  Class isa OBJC_ISA_AVAILABILITY; //isa指针指向Meta Class，因为Objc的类的本身也是一个Object，为了处理这个关系，runtime就创造了Meta Class，当给类发送[NSObject alloc]这样消息时，实际上是把这个消息发给了Class Object
	  #if !__OBJC2__
	  Class super_class OBJC2_UNAVAILABLE; // 父类
	  const char *name OBJC2_UNAVAILABLE; // 类名
	  long version OBJC2_UNAVAILABLE; // 类的版本信息，默认为0
	  long info OBJC2_UNAVAILABLE; // 类信息，供运行期使用的一些位标识
	  long instance_size OBJC2_UNAVAILABLE; // 该类的实例变量大小
	  struct objc_ivar_list *ivars OBJC2_UNAVAILABLE; // 该类的成员变量链表
	  struct objc_method_list **methodLists OBJC2_UNAVAILABLE; // 方法定义的链表
	  struct objc_cache *cache OBJC2_UNAVAILABLE; // 方法缓存，对象接到一个消息会根据isa指针查找消息对象，这时会在method Lists中遍历，如果cache了，常用的方法调用时就能够提高调用的效率。
	  struct objc_protocol_list *protocols OBJC2_UNAVAILABLE; // 协议链表
	  #endif
	  } OBJC2_UNAVAILABLE;
### [※※※]objc中向一个对象发送消息[obj foo]和objc_msgSend()函数之间有什么关系？

Runtime运行原理就如上描述一致，消息发生会被转发objc_msgSend((id)obj,@selecter(foo));除此之外对于“边界情况”，如果消息是发给父类的例如：[super foo]那么消息会被转发objc_msgSendSuper((id)obj,@selecter(foo));如果foo消息返回的是结构体，那么消息转换会变成objc_msgSend_stret((id)obj,@selecter(foo));如果foo消息返回的是浮点型数据，那么消息转换成objc_msgSend_fpret((id)obj,@selecter(foo));
这些转换都是在动态编译的时候进行转换的。


### [※※※]什么时候会报unrecognized selector的异常？

“unrecognized selecter”中文翻译就是无法识别的方法，当实例对象或者类对象发送一个该对象objc_method_list/objc_protocol_list没有对应方法的时候，NSObject就是抛出异常。<br/>
在Runtime抛出异常之间还可做很多事情来处理这种异常，这要依赖于NSObject的消息转发机制。<br/>
首先在Class中的缓存查找IMP（没缓存则初始化缓存），如果没找到，则向父类的Class查找。如果一直查找到根类仍旧没有实现，则用_objc_msgForward函数指针代替IMP。最后，执行这个IMP。<br/>
_objc_msgForward是用于消息转发的。这个函数的实现并没有在objc-runtime的开源代码里面，而是在Foundation框架里面实现的。__CFInitialize这个方法会调用objc_setForwardHandler函数来注册一个实现。<br/>
_objc_msgForward消息转发做了如下几件事：

征询接受者看其是否能动态添加方法，以处理当前这个“未知的选择子”（这叫动态方法解析）即调用：

	+ (BOOL)resolveInstanceMethod:(SEL)selecter;//实例对象的处理方法
	+ (BOOL)resolveClassMethod:(SEL)selecter;//类对象的处理方法	
该方法的参数就是哪个未知的选择子，我们可以在这里处理哪些没有被objc_msgSend找到的方法。<br/>
假如在这个阶段还是没有处理这个未知的SEL，那么该SEL会被转发到备用接受者
	
	- (id)forwardingTargetForSelecter:(SEL)selecter;
	
这个方法返回指定的能够处理该消息的实例对象或者类对象。<br/>
如果没有备用接收者，那么就启用完整的转发机制，就是把尚未处理的那条消息的全部信息（选择子，目标，参数）封装成NSInvocation对象，并触发NSinvocation对象，让“消息派发系统”把消息指派给目标对象。即调用:
	
	- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector;//获取方法签名信息
	- (void)forwardInvocation:(NSInvocation*)invocaton;
	
消息派发系统依照类的继承关系逐一寻找能够处理该NSInvacation的对象，直到NSObject，如果最后调用了NSObject类方法，那么该方法还会继续调用`doesNotRecognizeSelecter:`以抛出异常。

最后陪一个能够理解的图：
![消息转发机制流程图](http://7xoo3c.com1.z0.glb.clouddn.com/blogmethod_forwarding.png "消息转发机制流程图")


### [※※※※]一个objc对象如何进行内存布局？（考虑有父类的情况）
* 一个objc对象的存储空间中保存着该对象的所有实例变量和该对象所有父类的所有实例变量
* 每个对象都有一个isa指针，这个指针指向该objc对象的类对象，此类对象在Runtime环境中只有一份儿，并且和其它类对象有着相同的结构，此类对象中存储的是：

typedef struct objc_class *Class;
	
	/*
	  这是由编译器为每个类产生的数据结构,这个结构定义了一个类.
	  这个结构是通过编译器在执行时产生,在运行时发送消息时使用.
	  因此,一些成员改变了类型.
	  编译器产生"char* const"类型的字符串指针替代了下面的成员变量"super_class"
	*/
	struct objc_class {
	  struct objc_class*  class_pointer;    /* 指向元类的指针. */
	  struct objc_class*  super_class;      /* 指向父类的指针. 对于NSObject来说是NULL.*/
	  const char*         name;             /* 类的名称. */
	  long                version;          /* 未知. */
	  unsigned long       info;             /* 比特蒙板.  参考下面类的蒙板定义. */
	  long                instance_size;    /* 类的字节数.包含类的定义和所有父类的定义 */
	#ifdef _WIN64
	  long pad;
	#endif
	  struct objc_ivar_list* ivars;         /* 指向类中定义的实例变量的列表结构. NULL代表没有实例变量.不包括父类的变量. */
	  struct objc_method_list*  methods;    /* 链接类中定义的实例方法. */
	  struct sarray *    dtable;            /* 指向实例方法分配表. */
	  struct objc_class* subclass_list;     /* 父类列表 */
	  struct objc_class* sibling_class;
	  struct objc_protocol_list *protocols; /* 要实现的原型列表 */
	  void* gc_object_type;
	};

该类也有一个isa指针，它指向该类的元类，元类的作用类似于类作用于类对象，保存了该类的类方法信息。

引用网上常列举的一张图：
![内存布局图](http://7xoo3c.com1.z0.glb.clouddn.com/blogmeta_calss.png "类内存布局图")

### [※※※※]一个objc对象的isa的指针指向什么？有什么作用？
1. 实例对象的isa指向该实例的类对象，类对象的isa指向该类的元类，元类的isa指向自己。
2. 类对象保存着该类的成员列表信息，方法列表信息等。元类保存着该类对象的类方法列表。


### [※※※※]下面的代码输出什么？

	@implementation Son : Father
	- (id)init
	{
	    self = [super init];
	    if (self) {
	        NSLog(@"%@", NSStringFromClass([self class]));
	        NSLog(@"%@", NSStringFromClass([super class]));
	    }
	    return self;
	}
	@end
	
都输出：Son，答案：http://chun.tips/blog/2014/11/05/bao-gen-wen-di-objective%5Bnil%5Dc-runtime(1)%5Bnil%5D-self-and-super/

	2014-11-05 11:06:18.060 Test[8566:568584] NSStringFromClass([self class]) = Son
	2014-11-05 11:06:18.061 Test[8566:568584] NSStringFromClass([super class]) = Son

解惑：这个题目主要是考察关于objc中对 self 和 super 的理解。

self 是类的隐藏参数，指向当前调用方法的这个类的实例。而 super 是一个 Magic Keyword， 它本质是一个编译器标示符，和 self 是指向的同一个消息接受者。上面的例子不管调用[self class]还是[super class]，接受消息的对象都是当前 Son ＊xxx 这个对象。而不同的是，super是告诉编译器，调用 class 这个方法时，要去父类的方法，而不是本类里的。

当使用 self 调用方法时，会从当前类的方法列表中开始找，如果没有，就从父类中再找；而当使用 super 时，则从父类的方法列表中开始找。然后调用父类的这个方法。

真的是这样吗？继续看：

使用clang重写命令:

	$ clang -rewrite-objc test.m
发现上述代码被转化为:

    NSLog((NSString *)&__NSConstantStringImpl__var_folders_gm_0jk35cwn1d3326x0061qym280000gn_T_main_a5cecc_mi_0, NSStringFromClass(((Class (*)(id, SEL))(void *)objc_msgSend)((id)self, sel_registerName("class"))));

    NSLog((NSString *)&__NSConstantStringImpl__var_folders_gm_0jk35cwn1d3326x0061qym280000gn_T_main_a5cecc_mi_1, NSStringFromClass(((Class (*)(__rw_objc_super *, SEL))(void *)objc_msgSendSuper)((__rw_objc_super){ (id)self, (id)class_getSuperclass(objc_getClass("Son")) }, sel_registerName("class"))));
    
从上面的代码中，我们可以发现在调用`[self class]`时，会转化成`objc_msgSend`函数。看下函数定义：

	id objc_msgSend(id self, SEL op, ...)
我们把`self`做为第一个参数传递进去。

而在调用`[super class]`时，会转化成`objc_msgSendSuper`函数。看下函数定义:

	id objc_msgSendSuper(struct objc_super *super, SEL op, ...)
第一个参数是`objc_super`这样一个结构体，其定义如下:

	struct objc_super {
	   __unsafe_unretained id receiver;
	   __unsafe_unretained Class super_class;
	};
结构体有两个成员，第一个成员是`receiver`, 类似于上面的`objc_msgSend`函数第一个参数`self`。第二个成员是记录当前类的父类是什么。

所以，当调用`[self class]`时，实际先调用的是`objc_msgSend`函数，第一个参数是`Son`当前的这个实例，然后在`Son`这个类里面去找 `- (Class)class`这个方法，没有，去父类 Father里找，也没有，最后在 NSObject类中发现这个方法。而 `- (Class)class`的实现就是返回self的类别，故上述输出结果为 Son。

objc Runtime开源代码对- (Class)class方法的实现:

	- (Class)class {
	    return object_getClass(self);
	}
而当调用`[super class]`时，会转换成`objc_msgSendSuper`函数。第一步先构造`objc_super`结构体，结构体第一个成员就是 self 。第二个成员是 `(id)class_getSuperclass(objc_getClass(“Son”)) `, 实际该函数输出结果为 Father。第二步是去 Father这个类里去找`- (Class)class`，没有，然后去NSObject类去找，找到了。最后内部是使用` objc_msgSend(objc_super->receiver, @selector(class))`去调用，此时已经和`[self class]`调用相同了，故上述输出结果仍然返回 `Son`。

> 参考：http://chun.tips/blog/2014/11/05/bao-gen-wen-di-objective%5Bnil%5Dc-runtime(1)%5Bnil%5D-self-and-super/
	
### [*]runtime 如何实现 weak 属性?

`weak`修饰符是ARC下引入的修饰符，其和`assign`表示相同，表示`nonowning relationship`(非拥有关系)，对于被`weak`修饰的对象，当其所引用的对象的引用计数为0的时候，该对象就为被值为nil。这个过程是依靠Runtime来实现的，当Runtime载入并注册类，会对内存布局，把类中每个被`weak`修饰的属性加入多一个与该类相关的哈希表结构中，此结构以该对象的地址为key，当该对象的引用计数为0的时候，就会调用dealloc方法，该方法会检索此哈希表，找到所有key为该地址的所有对象，并把它们设值成nil。

### [※※※※]runtime如何通过selector找到对应的IMP地址？（分别考虑类方法和实例方法）

我们看一下`@selector(methodName:)`返回一个`SEL`类型:

	typedef struct objc_selector *SEL;
	
	struct objc_selector
	{
	  void *sel_id;
	  const char *sel_types;
	};
	
上面的结构是在网上搜索到的，但是我在objc/runtime.h中没有找到objc_selector的定义。

> https://sourceware.org/svn/gcc/tags/stack-last-merge/libobjc/objc/objc.h

这里指明了objc.h是GCC中的一个文件`This file is part of GCC.`。

Objective-C在编译时，会依据每一个方法的名字、参数序列，生成一个唯一的整型标识(Int类型的地址)，这个标识就是SEL。所以在Objective-C同一个类(及类的继承体系)中，不能存在2个同名的方法，即使参数类型不同也不行。相同的方法只能对应一个SEL。这也就导致Objective-C在处理相同方法名且参数个数相同但类型不同的方法方面的能力很差。如在某个类中定义以下两个方法：
	
	- (void)setWidth:(int)width;
	
	- (void)setWidth:(double)width;
这样的定义被认为是一种编译错误，所以我们不能像C++, C#那样。而是需要像下面这样来声明：

	-(void)setWidthIntValue:(int)width;
	
	-(void)setWidthDoubleValue:(double)width;
当然，不同的类可以拥有相同的selector，这个没有问题。不同类的实例对象执行相同的selector时，会在各自的方法列表中去根据selector去寻找自己对应的IMP。<br/>

工程中的所有的SEL组成一个Set集合，Set的特点就是唯一，因此SEL是唯一的。因此，如果我们想到这个方法集合中查找某个方法时，只需要去找到这个方法对应的SEL就行了，SEL实际上就是根据方法名hash化了的一个字符串，而对于字符串的比较仅仅需要比较他们的地址就可以了，可以说速度上无语伦比！！但是，有一个问题，就是数量增多会增大hash冲突而导致的性能下降（或是没有冲突，因为也可能用的是perfect hash）。但是不管使用什么样的方法加速，如果能够将总量减少（多个方法可能对应同一个SEL），那将是最犀利的方法。那么，我们就不难理解，为什么SEL仅仅是函数名了。

本质上，SEL只是一个指向方法的指针（准确的说，只是一个根据方法名hash化了的KEY值，能唯一代表一个方法），它的存在只是为了加快方法的查询速度。这个查找过程我们将在下面讨论。

我们可以在运行时添加新的selector，也可以在运行时获取已存在的selector，我们可以通过下面三种方法来获取SEL:

1. sel_registerName函数
2. Objective-C编译器提供的@selector()
3. NSSelectorFromString()方法

IMP实际上是一个函数指针，指向方法实现的首地址。其定义如下：

	id (*IMP)(id, SEL, ...)
	
介绍完SEL和IMP，我们就可以来讲讲Method了。Method用于表示类定义中的方法，则定义如下：

	typedef struct objc_method *Method;
	
	
	
	struct objc_method {
	
	    SEL method_name                 OBJC2_UNAVAILABLE;  // 方法名
	
	    char *method_types                  OBJC2_UNAVAILABLE;
	
	    IMP method_imp                      OBJC2_UNAVAILABLE;  // 方法实现
	
	}  
我们可以看到该结构体中包含一个SEL和IMP，实际上相当于在SEL和IMP之间作了一个映射。有了SEL，我们便可以找到对应的IMP，从而调用方法的实现代码。
	
对于每一个实例对象它会有一个isa指针，该指针指向该实例对象的类对象，它是一个objc_class的结构体，该结构题中的objc_method_list内包含了该对象的所有实例方法，对于类方法，每个类对象中又有一个isa指针指向该类对象的元类，元类objc_class结构的objc_method_list结构中存储着该类的所有类方法。Runtime通过查找这两个类来获取方法的IMP。

> 参考：http://southpeak.github.io/blog/2014/11/03/objective-c-runtime-yun-xing-shi-zhi-san-:fang-fa-yu-xiao-xi-zhuan-fa/

### [※※※※]使用runtime Associate方法关联的对象，需要在主对象dealloc的时候释放么？
不需要（ARC下和MRC），`overfllow`的一句话`Associated objects are released after the dealloc method of the original object has finished.`。通过源码我们可以得知关联对象是通过一个单例类`AssociationsManager`实例维护一个哈希列表`AssociationsHashMap`实现的。而map的key是这个对象的指针地址（任意两个不同对象的指针地址一定是不同的），而这个map的value又是另外一个AssociationsHashMap，里面保存了关联对象的kv对。

而在对象的销毁逻辑里面，当对象的引用计数为0的时候，Runloop在清理对象的时候会调用对象的`dealloc`,该方法会调用`object_dispose`方法来清理对象，在`object_dispose`步会对关联对象进行清理。源代码见`objc-runtime-new.mm`:

	void *objc_destructInstance(id obj) 
	{
	    if (obj) {
	        Class isa_gen = _object_getClass(obj);
	        class_t *isa = newcls(isa_gen);
	
	        // Read all of the flags at once for performance.
	        bool cxx = hasCxxStructors(isa);
	        bool assoc = !UseGC && _class_instancesHaveAssociatedObjects(isa_gen);
	
	        // This order is important.
	        if (cxx) object_cxxDestruct(obj);
	        if (assoc) _object_remove_assocations(obj);
	
	        if (!UseGC) objc_clear_deallocating(obj);
	    }
	
	    return obj;
	}
	
Runtime的销毁对象函数objc_destructInstance里面会判断这个对象有没有关联对象，如果有，会调用_object_remove_assocations做关联对象的清理工作。

### [※※※※※]objc中的类方法和实例方法有什么本质区别和联系？

### [※※※※※]_objc_msgForward函数是做什么的，直接调用它将会发生什么？
### [※※※※※]runtime如何实现weak变量的自动置nil？

### [※※※※※]能否向编译后得到的类中增加实例变量？能否向运行时创建的类中添加实例变量？为什么？

不能向遍以后得到的类中增加实例变量，但是能够向运行时创建的类中添加实例变量。

	class_addIvar

> This function may only be called after objc_allocateClassPair and before objc_registerClassPair. 
> Adding an instance variable to an existing class is not supported.

> Objective-C不支持往已存在的类中添加实例变量，因此不管是系统库提供的提供的类，还是我们自定义的类，都无法动态添加成员变量。但如果我们通过运行时来创建一个类的话，又应该如何给它添加成员变量呢？这时我们就可以使用class_addIvar函数了。不过需要注意的是，这个方法只能在objc_allocateClassPair函数与objc_registerClassPair之间调用。另外，这个类也不能是元类。成员变量的按字节最小对齐量是1<<alignment。这取决于ivar的类型和机器的架构。如果变量的类型是指针类型，则传递log2(sizeof(pointer_type))。

> 文／西木（简书作者）
> 原文链接：http://www.jianshu.com/p/6b905584f536


### [※※※]runloop和线程有什么关系？

### [※※※]runloop的mode作用是什么？

### [※※※※]以+ scheduledTimerWithTimeInterval...的方式触发的timer，在滑动页面上的列表时，timer会暂定回调，为什么？如何解决？

### [※※※※※]猜想runloop内部是如何实现的？

### [※]objc使用什么机制管理对象内存？

### [※※※※]ARC通过什么方式帮助开发者管理内存？

### [※※※※]不手动指定autoreleasepool的前提下，一个autorealese对象在什么时刻释放？（比如在一个vc的viewDidLoad中创建）

### [※※※※]BAD_ACCESS在什么情况下出现？

### [※※※※※]苹果是如何实现autoreleasepool的？

### [※※]使用block时什么情况会发生引用循环，如何解决？

### [※※]在block内如何修改block外部变量？

### [※※※]使用系统的某些block api（如UIView的block版本写动画时），是否也考虑引用循环问题？

### [※※]GCD的队列（dispatch_queue_t）分哪两种类型？

### [※※※※]如何用GCD同步若干个异步调用？（如根据若干个url异步加载多张图片，然后在都下载完成后合成一张整图）

### [※※※※]dispatch_barrier_async的作用是什么？

### [※※※※※]苹果为什么要废弃dispatch_get_current_queue？

### [※※※※※]以下代码运行结果如何？

	- (void)viewDidLoad
	{
	    [super viewDidLoad];
	    NSLog(@"1");
	    dispatch_sync(dispatch_get_main_queue(), ^{
	        NSLog(@"2");
	    });
	    NSLog(@"3");
	}
### [※※]addObserver:forKeyPath:options:context:各个参数的作用分别是什么，observer中需要实现哪个方法才能获得KVO回调？

### [※※※]如何手动触发一个value的KVO?

### [※※※]若一个类有实例变量NSString *_foo，调用setValue:forKey:时，可以以foo还是_foo作为key？

### [※※※※]KVC的keyPath中的集合运算符如何使用？

### [※※※※]KVC和KVO的keyPath一定是属性么？

### [※※※※※]如何关闭默认的KVO的默认实现，并进入自定义的KVO实现？

### [※※※※※]apple用什么方式实现对一个对象的KVO？

### [※※]IBOutlet连出来的视图属性为什么可以被设置成weak?

### [※※※※※]IB中User Defined Runtime Attributes如何使用？

### [※※※]如何调试BAD_ACCESS错误?

### [※※※]lldb（gdb）常用的调试命令？