---
title: input
date: 2019-08-15 14:26:57
categories:
- Kernel
tags:
- Kernel
- Android
typora-root-url: input
typora-copy-images-to: input
---

# Input 概述

## 系统整体框图

![](1.1-1.png)



## Input 子系统框图

![](1.1-2.png)

- 设备驱动层

  主要实现对硬件设备的读写访问,中断设置,并把硬件产生的事件转换为核心层定义的规范提交给事件处理层

- 核心层

  为设备驱动层提供了规范和接口. 设备驱动层只要关心如何驱动硬件并获得硬件数据, 然后调用核心层的接口,  核心层会自动把数据提交给事件处理层.

- 事件处理层

  是用户编程的接口(设备节点), 并处理驱动层提交的数据



## 数据关系框图

![](1.1-3.png)

- input_dev

  - 是硬件驱动层，代表一个input设备
  - 通过全局的input_dev_list链接在一起。设备注册的时候实现这个操作
  - Input_dev可以通过Input_handle找到Input_handler

- Input_handler

  - 是事件处理层，代表一个事件处理器(如 evdev mousedev等)
  - 通过全局的input_handler_list链接在一起。事件处理器注册的时候实现这个操作
  - 事件处理器一般内核自带，一般不需要我们来写
  - Input_handler可以通过Input_handle找到Input_dev

- Input_handle

  - 属于核心层，代表一个配对的input设备与input事件处理器
  - input_hande 没有一个全局的链表，它注册的时候将自己分别挂在了input_dev 和 input_handler 的h_list上了。
  - 通过input_handle也可以找到input_dev和input_handler

- 一个device可能对应多个handler,而一个handler也不能只处理一个device

  比如说一个鼠标,它可以对应evdev_handler,也可以对应mouse_handler,因此当其注册时与系统中的handler进行匹配,就有可能产生两个实例,一个是evdev,另一个是mousedev,而任何一个实例中都只有一个handle，至于以何种方式来传递事件,就由用户程序打开哪个实例来决定



# 设备驱动层

## 概述

纯硬件操作层, 包含不同硬件接口处理. 对于每种不同的具体硬件操作都对应着不同的Input_dev结构体.

## 数据结构

### input_dev

```c
struct input_dev {  
  
    void *private;              //输入设备私有指针，一般指向用于描述设备驱动层的设备结构  
  
    const char *name;           //提供给用户的输入设备的名称  
    const char *phys;           //提供给编程者的设备节点的名称  
    const char *uniq;           //指定唯一的ID号，就像MAC地址一样  
    struct input_id id;         //输入设备标识ID，用于和事件处理层进行匹配  
  
    unsigned long evbit[NBITS(EV_MAX)];     //位图，记录设备支持的事件类型  
    unsigned long keybit[NBITS(KEY_MAX)];       //位图，记录设备支持的按键类型  
    unsigned long relbit[NBITS(REL_MAX)];       //位图，记录设备支持的相对坐标  
    unsigned long absbit[NBITS(ABS_MAX)];       //位图，记录设备支持的绝对坐标  
    unsigned long mscbit[NBITS(MSC_MAX)];   //位图，记录设备支持的其他功能  
    unsigned long ledbit[NBITS(LED_MAX)];       //位图，记录设备支持的指示灯  
    unsigned long sndbit[NBITS(SND_MAX)];       //位图，记录设备支持的声音或警报  
    unsigned long ffbit[NBITS(FF_MAX)];     //位图，记录设备支持的作用力功能  
    unsigned long swbit[NBITS(SW_MAX)];     //位图，记录设备支持的开关功能  
  
    unsigned int keycodemax;        //设备支持的最大按键值个数  
    unsigned int keycodesize;       //每个按键的字节大小  
    void *keycode;              //指向按键池，即指向按键值数组首地址  
    int (*setkeycode)(struct input_dev *dev, int scancode, int keycode);    //修改按键值  
    int (*getkeycode)(struct input_dev *dev, int scancode, int *keycode);   //获取按键值  
  
    struct ff_device *ff;           //用于强制更新输入设备的部分内容  
  
    unsigned int repeat_key;        //重复按键的键值  
    struct timer_list timer;        //设置当有连击时的延时定时器  
  
    int state;      //设备状态  
  
    int sync;       //同步事件完成标识，为1说明事件同步完成  
  
    int abs[ABS_MAX + 1];       //记录坐标的值  
    int rep[REP_MAX + 1];       //记录重复按键的参数值  
  
    unsigned long key[NBITS(KEY_MAX)];      //位图，按键的状态  
    unsigned long led[NBITS(LED_MAX)];      //位图，led的状态  
    unsigned long snd[NBITS(SND_MAX)];      //位图，声音的状态  
    unsigned long sw[NBITS(SW_MAX)];            //位图，开关的状态  
  
    int absmax[ABS_MAX + 1];                    //位图，记录坐标的最大值  
    int absmin[ABS_MAX + 1];                    //位图，记录坐标的最小值  
    int absfuzz[ABS_MAX + 1];                   //位图，记录坐标的分辨率  
    int absflat[ABS_MAX + 1];                   //位图，记录坐标的基准值  
  
    int (*open)(struct input_dev *dev);         //输入设备打开函数  
    void (*close)(struct input_dev *dev);           //输入设备关闭函数  
    int (*flush)(struct input_dev *dev, struct file *file); //输入设备断开后刷新函数
    //事件处理 
    int (*event)(struct input_dev *dev, unsigned int type, unsigned int code, int value);   
  
    struct input_handle *grab;      //类似私有指针，可以直接访问到事件处理接口event  
  
    struct mutex mutex;     //用于open、close函数的连续访问互斥  
    unsigned int users;     //设备使用计数  
  
    struct class_device cdev;   //输入设备的类信息  
    union {             //设备结构体  
        struct device *parent;  
    } dev;  
  
    struct list_head    h_list; //handle链表  
    struct list_head    node;   //input_dev链表  
};
```

Input_dev通过全局变量input_dev_list链接在一起. 设备注册的时候实现这个操作.



### input_id

```C
struct input_id {  
    __u16 bustype;   //总线类型  
    __u16 vendor;    //生产厂商  
    __u16 product;   //产品类型  
    __u16 version;   //版本  
 };
```

如果需要特定的事件处理器来处理这个设备的话,这几个参数非常重要, 因为子系统核心时通过他们将设备驱动与事件处理层联系起来的. 但因为触摸屏所用的处理器为evdev, 匹配所有,所以这个初始化也无关紧要了.



## 输入设备驱动注册

![](1.1-4.png)

![](1.1-6.jpeg)

- 输入设备驱动最终的目的即使能够与事件处理层的事件驱动相互匹配. 但是在drivers/input目录下有evdev.c事件驱动、mousedev.c事件驱动、joydev.c事件驱动等等. 我们的输入设备产生的事件最终上报给谁, 然后让事件驱动再去处理呢?
- evdev.c、mousedev.c 等是根据硬件输入设备的处理方式不同而抽象出来不同的事件处理接口帮助上层去调用的. 而我们写的设备驱动程序只不过是完成了硬件寄存器中的数据读写, 但提交给用户的事件必须是经过事件处理层的封装和同步才能够完成的.



# 核心层

## 概述

主要功能

- 注册主设备号
- 对于swi进入的open函数进行第一层处理, 并通过此设备号选择handler进入第二次open,也就是真正的open所在的file_operation, 并返回该file_operation的fd
- 提供input_register_device 和 input_register_handler函数分别用于注册device和handler



## 数据结构

### input_handle

```c
struct input_handle {  
    //每个配对的事件处理器都会分配一个对应的设备结构，如evdev事件处理器的evdev结构，注意这个结构与设备驱动层的input_dev不同，初始化handle时，保存到这里。
    void *private;     
    //打开标志，每个input_handle 打开后才能操作，这个一般通过事件处理器的open方法间接设置  
    int open;
    const char *name;   
    struct input_dev *dev;  //关联的input_dev结构  
    struct input_handler *handler; //关联的input_handler结构  
    struct list_head    d_node;  //input_handle通过d_node连接到了input_dev上的h_list链表上  
    struct list_head    h_node;  //input_handle通过h_node连接到了input_handler的h_list链表上  
};
```



## 函数调用

### input_register_device

```c
int input_register_device(struct input_dev *dev)  
{  
    /* 用于记录输入设备名称的索引值 */  
    static atomic_t input_no = ATOMIC_INIT(0);  
    /* 输入事件的处理接口指针，用于和设备的事件类型进行匹配 */  
    struct input_handler *handler;  
    const char *path;  
    int error;  
  
    /* 默认所有的输入设备都支持EV_SYN同步事件 */  
    set_bit(EV_SYN, dev->evbit);  
  
    /* 
     * 如果设备驱动没有指定重复按键（连击），系统默认提供以下的支持 
     * 其中init_timer为连击产生的定时器，时间到调用input_repeat_key函数 
     * 上报，REP_DELAY用于设置重复按键的键值，REP_PERIOD设置延时时间 
     */  
    init_timer(&dev->timer);  
    if (!dev->rep[REP_DELAY] && !dev->rep[REP_PERIOD]) {  
        dev->timer.data = (long) dev;  
        dev->timer.function = input_repeat_key;  
        dev->rep[REP_DELAY] = 250;  
        dev->rep[REP_PERIOD] = 33;  
    }  
  
    /* 如果设备驱动没有设置自己的获取键值的函数，系统默认 */  
    if (!dev->getkeycode)  
        dev->getkeycode = input_default_getkeycode;  
  
    /* 如果设备驱动没有指定按键重置函数，系统默认 */  
    if (!dev->setkeycode)  
        dev->setkeycode = input_default_setkeycode;  
  
    /* 重要，把设备挂到全局的input子系统设备链表input_dev_list上 */  
    list_add_tail(&dev->node, &input_dev_list);  
  
    /* 动态获取input设备的ID号，名称为input*，其中后面的“*”动态获得，唯一的 */  
    snprintf(dev->cdev.class_id, sizeof(dev->cdev.class_id),  
         "input%ld", (unsigned long) atomic_inc_return(&input_no) - 1);  
  
    /* 如果这个值没有设置，系统把输入设备挂入设备链表 */  
    if (!dev->cdev.dev)  
        dev->cdev.dev = dev->dev.parent;  
  
    /* 在/sys目录下创建设备目录和文件 */  
    error = class_device_add(&dev->cdev);  
    if (error)  
        return error;  
  
    /* 获取并打印设备的绝对路径名称 */  
    path = kobject_get_path(&dev->cdev.kobj, GFP_KERNEL);  
    printk(KERN_INFO "input: %s as %s\n",  
        dev->name ? dev->name : "Unspecified device", path ? path : "N/A");  
    kfree(path);  
  
    /* 核心重点，input设备在增加到input_dev_list链表上之后，会查找 
     * input_handler_list事件处理链表上的handler进行匹配，这里的匹配 
     * 方式与设备模型的device和driver匹配过程很相似，所有的input 
     * 都挂在input_dev_list上，所有类型的事件都挂在input_handler_list 
     * 上，进行“匹配相亲”*/  
    list_for_each_entry(handler, &input_handler_list, node)  
        input_attach_handler(dev, handler);  
  
    input_wakeup_procfs_readers();  
  
    return 0;  
}
```

主要功能:

- 进一步初始化输入设备
- 将自己的device结构添加到linux设备模型当中，将input_dev添加到input_dev_list链表中
- 需要合适的handler与Input_handler配对，配对的核心函数Input_attach_handler



#### input_attach_handler

```c
static int input_attach_handler(struct input_dev *dev, struct input_handler *handler)  
{  
    const struct input_device_id *id;  
    int error;  
  
    //如果handler的blacklist被赋值了并且则优先匹配
    if (handler->blacklist && input_match_device(handler->blacklist, dev))  
        return -ENODEV;  
    
    //否则利用handler->id_table和dev进行匹配，后面讲述匹配什么和过程
    id = input_match_device(handler->id_table, dev);  
    if (!id)  
        return -ENODEV;  
    
    /*
     *配对成功调用handler的connect函数
     *这个函数在事件处理器中定义，主要生成一个input_handle结构，并初始化
     *还生成一个事件处理器相关的设备结构
     */
    error = handler->connect(handler, dev, id);
    if (error && error != -ENODEV)  
        printk(KERN_ERR  
            "input: failed to attach handler %s to device %s, "  
            "error: %d\n",  
            handler->name, kobject_name(&dev->dev.kobj), error);
    
    return error;  
 }
```



#### input_match_device

```c
struct input_device_id *input_match_device(struct input_handler *handler,
							struct input_dev *dev)
{
	const struct input_device_id *id;

    /* 事件处理层中的对应flags如果设置或者driver_info被设置则进行匹配 */  
	for (id = handler->id_table; id->flags || id->driver_info; id++) {

        /* 以下通过flags中设置的位来匹配设备的总线类型、经销商、生产ID和版本ID 
          如果没有匹配上将进行MATCH_BIT匹配 */
		if (id->flags & INPUT_DEVICE_ID_MATCH_BUS)
			if (id->bustype != dev->id.bustype)
				continue;

		if (id->flags & INPUT_DEVICE_ID_MATCH_VENDOR)
			if (id->vendor != dev->id.vendor)
				continue;

		if (id->flags & INPUT_DEVICE_ID_MATCH_PRODUCT)
			if (id->product != dev->id.product)
				continue;

		if (id->flags & INPUT_DEVICE_ID_MATCH_VERSION)
			if (id->version != dev->id.version)
				continue;

        /* 用于匹配设备驱动中是否设置了这些位 
        * 我们在设备驱动中设置的事件类型会与事件链表中的 
        * 所有事件类型进行比较，匹配成功了将返回id，证明真的很合适，否则NULL 
        */ 
		if (!bitmap_subset(id->evbit, dev->evbit, EV_MAX))
			continue;
		if (!bitmap_subset(id->keybit, dev->keybit, KEY_MAX))
			continue;
		if (!bitmap_subset(id->relbit, dev->relbit, REL_MAX))
			continue;
		if (!bitmap_subset(id->absbit, dev->absbit, ABS_MAX))
			continue;
		if (!bitmap_subset(id->mscbit, dev->mscbit, MSC_MAX))
			continue;
		if (!bitmap_subset(id->ledbit, dev->ledbit, LED_MAX))
			continue;
		if (!bitmap_subset(id->sndbit, dev->sndbit, SND_MAX))
			continue;
		if (!bitmap_subset(id->ffbit, dev->ffbit, FF_MAX))
			continue;
		if (!bitmap_subset(id->swbit, dev->swbit, SW_MAX))
			continue;
		if (!handler->match || handler->match(handler, dev))
			return id;
	}

	return NULL;
}
```

- 首先看id->driver_info有没有设置, 如果设置了说明匹配所有的id, evdev就是这个样的handler
- 然后依据id->flag来比较内容, 如果比较成功后则比较所支持事件的类型,只有所有位都匹配才成功返回
- 主要是比较Input_dev中的id和handler支持的id, 这个存放在handler的id_table中



### input_register_handle

```c
int input_register_handle(struct input_handle *handle)  
{  
    struct input_handler *handler = handle->handler;  
    struct input_dev *dev = handle->dev;  
    int error;  
  
    /* 将d_node链接到输入设备的h_list，h_node链接到事件层的h_list链表上 
    * 因此，在handle中是输入设备和事件层的关联结构体，通过输入设备可以 
    * 找到对应的事件处理层接口，通过事件处理层也可找到匹配的输入设备 
    */  
    error = mutex_lock_interruptible(&dev->mutex);  
    if (error)  
        return error;  
    list_add_tail_rcu(&handle->d_node, &dev->h_list);  
    mutex_unlock(&dev->mutex);  
    
    list_add_tail(&handle->h_node, &handler->h_list);  
    
    if (handler->start)  
        handler->start(handle);  
  
    return 0;  
}
```



### input_register_handler

```c
int input_register_handler(struct input_handler *handler)
{
	struct input_dev *dev;
	int error;

	error = mutex_lock_interruptible(&input_mutex);
	if (error)
		return error;

	INIT_LIST_HEAD(&handler->h_list);

    // 链接到input_handler_list链表中
	list_add_tail(&handler->node, &input_handler_list);

    //又是配对，不过这次遍历input_dev，和注册input_dev过程一样的
	list_for_each_entry(dev, &input_dev_list, node)
		input_attach_handler(dev, handler);

	input_wakeup_procfs_readers();

	mutex_unlock(&input_mutex);
	return 0;
}
```



# 事件处理层

## 概述

handler层是纯软件层， 包含不同的解决方案，如键盘、鼠标、游戏手柄等，但是没有涉及到硬件方面的操作。对于不同的解决方案，都包含一个名为Input_handler的结构体。



## 数据结构

### Input_handler

```c
struct input_handler {

    void *private;

    void (*event)(struct input_handle *handle, unsigned int type, unsigned int code, int value);/*event用于处理事件*/
    void (*events)(struct input_handle *handle,const struct input_value *vals, unsigned int count);
    bool (*filter)(struct input_handle *handle, unsigned int type, unsigned int code, int value);
    bool (*match)(struct input_handler *handler, struct input_dev *dev);
    int (*connect)(struct input_handler *handler, struct input_dev *dev, const struct input_device_id *id);/*connect用于建立handler和device的联系*/
    void (*disconnect)(struct input_handle *handle);/*disconnect用于解除handler和device的联系*/
    void (*start)(struct input_handle *handle);

    bool legacy_minors;
    int minor;//次设备号
    const char *name;

    const struct input_device_id *id_table;//用于和input_dev匹配

    struct list_head    h_list;//用于链接和此input_handler相关的input_handle
    struct list_head    node;//用于将该input_handler链入input_handler_list
};
```



## 事件处理器(evdev)

### evdev设备结构

```c
struct evdev {  
    int exist;  
    int open;           //打开标志  
    int minor;          //次设备号  
    struct input_handle handle;  //关联的input_handle  
    
    /*等待队列，当进程读取设备，而没有事件产生的时候，进程就会睡在其上面*/
    wait_queue_head_t wait; 
    
    /*强制绑定的evdev_client结构，这个结构后面再分析*/ 
    struct evdev_client *grab;
     
    /*
      *evdev_client 链表
      *这说明一个evdev设备可以处理多个evdev_client
      *可以有多个进程访问evdev设备
      */
    struct list_head client_list;   
    spinlock_t client_lock;
    struct mutex mutex; 
    
    /*device结构，说明这是一个设备结构*/
    struct device dev;
};
```

- evdev结构体在配对成功后生成,由handler->connect生成, 对应设备文件为/sys/class/Input/event(n)
- 如触摸屏驱动的event0，这个设备是用户空间要访问的设备，可以理解它是一个虚拟设备，因为没有对应的硬件，但是通过handle->dev 就可以找到input_dev结构，而它对应着触摸屏，设备文件为/sys/class/input/input0。这个设备结构生成之后保存在evdev_table中，索引值是minor



### evdev用户端设备结构

```c
struct evdev_client {
    /*
     *这个是一个input_event数据结构的数组
     *input_event代表一个事件
     *基本成员：类型（type），编码（code），值（value）
     */
    struct input_event buffer[EVDEV_BUFFER_SIZE];    
    int head;              //针对buffer数组的索引  
    int tail;              //针对buffer数组的索引，当head与tail相等的时候，说明没有事件  
    spinlock_t buffer_lock;  
    struct fasync_struct *fasync;  //异步通知函数  
    struct evdev *evdev;           //evdev设备  
    struct list_head node;         // evdev_client 链表项  
};
```

- 这个结构在进程打开event0设备时候调用evdev的open函数, 在open中创建设个结构,并初始化. 在关闭设备文件的时候释放这个结构



### evdev_handler结构体

```c
static struct input_handler evdev_handler = {
	.event		= evdev_event,//当硬件有事件上传将调用
	.connect	= evdev_connect,//当有新的input_dev和它匹配成功时调用
	.disconnect	= evdev_disconnect,
	.fops		= &evdev_fops,//提供读写函数
	.minor		= EVDEV_MINOR_BASE,//次设备起始号
	.name		= "evdev",
	.id_table	= evdev_ids,//它可接受的input_dev条件
};
```



### 数据结构关系图

![](1.1-5.jpg)



### connect函数

```c
static int evdev_connect(struct input_handler *handler, struct input_dev *dev,
             const struct input_device_id *id)
{
    struct evdev *evdev;
    int minor;
    int dev_no;
    int error;
    
    /*申请一个新的次设备号*/
    minor = input_get_new_minor(EVDEV_MINOR_BASE, EVDEV_MINORS, true);

    /* 这说明内核已经没办法再分配这种类型的设备了 */ 
    if (minor < 0) {
        error = minor;
        pr_err("failed to reserve new minor: %d\n", error);
        return error;
    }

    /* 开始给evdev事件层驱动分配空间了 */  
    evdev = kzalloc(sizeof(struct evdev), GFP_KERNEL);
    if (!evdev) {
        error = -ENOMEM;
        goto err_free_minor;
    }

    /* 初始化client_list列表和evdev_wait队列 */  
    INIT_LIST_HEAD(&evdev->client_list);
    spin_lock_init(&evdev->client_lock);
    mutex_init(&evdev->mutex);
    init_waitqueue_head(&evdev->wait);
    evdev->exist = true;

    dev_no = minor;
    /* Normalize device number if it falls into legacy range */
    if (dev_no < EVDEV_MINOR_BASE + EVDEV_MINORS)
        dev_no -= EVDEV_MINOR_BASE;
    
    /*设置设备节点名称，/dev/eventX 就是在此时设置*/
    dev_set_name(&evdev->dev, "event%d", dev_no);

    /* 初始化evdev结构体，其中handle为输入设备和事件处理的关联接口 */  
    evdev->handle.dev = input_get_device(dev);
    evdev->handle.name = dev_name(&evdev->dev);
    evdev->handle.handler = handler;
    evdev->handle.private = evdev;

    /*设置设备号，应用层就是通过设备号，找到该设备的*/
    evdev->dev.devt = MKDEV(INPUT_MAJOR, minor);
    evdev->dev.class = &input_class;
    evdev->dev.parent = &dev->dev;
    evdev->dev.release = evdev_free;
    device_initialize(&evdev->dev);

    /* input_dev设备驱动和handler事件处理层的关联，就在这时由handle完成 */ 
    error = input_register_handle(&evdev->handle);
    if (error)
        goto err_free_evdev;

    cdev_init(&evdev->cdev, &evdev_fops);
    evdev->cdev.kobj.parent = &evdev->dev.kobj;
    error = cdev_add(&evdev->cdev, evdev->dev.devt, 1);
    if (error)
        goto err_unregister_handle;

    /*将设备加入到Linux设备模型，它的内部将找到它的bus，然后让它的bus
    给它找到它的driver，在驱动或者总线的probe函数中，一般会在/dev/目录
    先创建相应的设备节点，这样应用程序就可以通过该设备节点来使用设备了
    ，/dev/eventX 设备节点就是在此时生成
    */
    error = device_add(&evdev->dev);
    if (error)
        goto err_cleanup_evdev;

    return 0;

 err_cleanup_evdev:
    evdev_cleanup(evdev);
 err_unregister_handle:
    input_unregister_handle(&evdev->handle);
 err_free_evdev:
    put_device(&evdev->dev);
 err_free_minor:
    input_free_minor(minor);
    return error;
}
```



# A/B(slot)协议

## A协议

### 描述

- 如果不管当前坐标点和上一坐标点数据是否一致都上报, 称为A协议

### 上报流程

#### 按下

```C
ABS_MT_POSITION_X x[0]
ABS_MT_POSITION_Y y[0]
SYN_MT_REPORT
ABS_MT_POSITION_X x[1]
ABS_MT_POSITION_Y y[1]
SYN_MT_REPORT
...
SYN_REPORT
```

- 是以SYN_MT_REPORT作为一个点的结尾
- 是以SYN_REPORT为一次事件的结尾
- 多指触摸的时候,native层每收到一次SYN_MT_REPORT就形成一个点的信息,收到一个点之后并不会立即处理,而是一个事件完成之后才会处理, SYN_REPORT就是这个事件的标志.
- A协议中有的只是点的坐标信息,那么系统如何去判断当前的多点属于哪一条线呢?
  - 假设一次事件共有5个点, 本次触摸也有5个点. 
  - 系统会分别计算一次前5个点和当前5个点的距离, 这样就产生了 5x5=25个数据.
  - 对这个25个数字进行排序, 判断当前点与哪一次的点最近,那么赋予它们相同的id, 就可以知道当前点属于哪条线了.

#### 抬起

```c
SYN_MT_REPORT
SYN_REPORT
```

- 只有SYNC, 没有任何信息,系统就会认为此次事件为UP



## B(slot)协议

### 描述

- 如果当前坐标点和上一坐标点数据一致,则该点不上报, 称为B(slot)协议
- slot直译为位置、槽,两层含义:一层是位置, 另一层是容器
- B协议使用了slot, 还有一个新的面孔TRACKING_ID

### 上报流程

#### 按下

```c
ABS_MT_SLOT 0
ABS_MT_TRACKING_ID **
ABS_MT_POSITION_X x[0]
ABS_MT_POSITION_Y y[0]
ABS_MT_SLOT 1
ABS_MT_TRACKING_ID **
ABS_MT_POSITION_X x[1]
ABS_MT_POSITION_Y y[1]
...
SYN_REPORT
```

- 相同的ABS_MT_TRACKING_ID点,它们属于同一条线

#### 抬起

```c
ABS_MT_SLOT 0
ABS_MT_TRACKING_ID -1
SYN_REPORT
ABS_MT_SLOT 1
ABS_MT_TRACKING_ID -1
SYN_REPORT
```

- 点上报的时候 ABS_MT_TRACKING_ID 为-1, 系统会清除对应的ID.

