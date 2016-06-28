---
title: Objective-C底层数据结构
date: 2016-06-27 13:46:34
tags: [Objective-C]
categories: Objective-C语言

---

# [转]Objective-C底层数据结构

<br/>
<br/>
> 原文地址：http://iostour.diandian.com/post/2013-01-16/40047469315

<br/>
<br/>
<br/>
## 类的数据结构

### Class(指针)

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
	
### Method(指针)

	typedef struct objc_method *Method;
	       
	/* 编译器依据类中定义的方法为该类产生一个或更多这种这种结构.
	    一个类的实现可以分散在一个文件中不同部分,同时类别可以分散在不同的模块中.为了处理这个问题,使用一个单独的方法链表 */
	struct objc_method
	{
	  SEL         method_name;  /* 这个变量就是方法的名称.编译器使用在这里使用一个`char*`,当一个方法被注册,名称在运行时被使用真正的SEL替代  */
	  const char* method_types; /* 描述方法的参数列表. 在运行时注册选择器时使用.那时候方法名就会包含方法的参数列表.*/
	  IMP         method_imp;   /* 方法执行时候的地址. */
	};

### Ivar(指针)

	typedef struct objc_ivar *Ivar;
	       
	/* 编译器依据类中定义的实例变量为该类产生一个或更多这种这种结构  */
	struct objc_ivar
	{
	  const char* ivar_name;  /* 类中定义的变量名. */
	  const char* ivar_type;  /* 描述变量的类型.调试时非常有用. */
	  int        ivar_offset; /* 实例结构的基地址偏移字节 */
	};

### Category(指针)

	typedef struct objc_category *Category;
	       
	/* 编译器为每个类别产生一个这样的结构.一个类可以具有多个类别同时既包括实例方法,也可以包括类方法*/
	struct objc_category
	{
	  const char*   category_name;                /* 类别名.定义在类别后面的括号内*/
	  const char*   class_name;                   /* 类名 */
	  struct objc_method_list  *instance_methods; /* 链接类中定义的实例方法. NULL表示没有实例方法. */
	  struct objc_method_list *class_methods;     /* 链接类中定义的类方法. NULL表示没有类方法. */
	  struct objc_protocol_list *protocols;       /* 遵循的协议表  */
	};

### objc_property_t

	typedef struct objc_property *objc_property_t;

### IMP

	id (*IMP)(id, SEL, ...)

### SEL

	typedef struct objc_selector *SEL;
	       
	struct objc_selector
	{
	  void *sel_id;
	  const char *sel_types;
	};

### objc_method_list

	struct objc_method_list
	{
	  struct objc_method_list*  method_next; /* 这个变量用来链接另一个单独的方法链表 */
	  int            method_count;            /* 结构中定义的方法数量 */
	  struct objc_method method_list[1];      /* 可变长度的结构 */
	};

### objc_cache

	struct objc_cache
	{
	    unsigned int mask;
	    unsigned int occupied;
	    Method buckets[1];
	};

### objc_protocol_list

	struct objc_protocol_list
	{
	  struct objc_protocol_list *next;
	  size_t count;
	  struct objc_protocol *list[1];
	};

## 实例的数据结构

### id

	typedef struct objc_object *id;

### objc_object

	struct objc_object
	{
	  /* 类的指针是对象相关的类.如果是一个类对象, 这个指针指向元类.
	  Class isa;
	};

### objc_super

	struct objc_super
	{
	  id    self;        /* 消息的接受者  */
	  Class super_class; /* 接受者的父类  */
	};


> 原文地址：http://iostour.diandian.com/post/2013-01-16/40047469315
