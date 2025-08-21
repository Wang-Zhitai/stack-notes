## 并发与IO

### 编写x86架构的驱动

**Makefile**

> obj-m += file.o
> KDIR:=/lib/modules/5.4.0-150-generic/build
> PWD?=$(shell pwd)
> all:
> 	make -C $(KDIR) M=$(PWD) modules
> clean:
> 	rm -f *.ko *.0 *.mod.o *. mod.c *. symvers *.order

### Linux 错误处理

对于任何一个指针来说， 必然存在三种情况， 一种是合法指针， 一种是 NULL(也就是空指针)， 一种是错误指针(也就是无效指针)。 在 Linux 内核中， 所谓的错误指针已经指向了内核空间的最后一页， 例如， 对于一个 64 位系统来说， 内核空间最后地址为 0xffffffffffffffff， 那么最后一页的地址是 0xfffffffffffff000~0xffffffffffffffff， 这段地址是被保留的， 如果指针落在这段地址之内， 说明是错误的无效指针。

在 Linux 内核源码中实现了指针错误的处理机制， 相关的函数接口主要有 IS_ERR()、PTR_ERR()、 ERR_PTR()等， 其函数的源码在 include/linux/err.h 文件中

在 Linux 源码中 IS_ERR()函数其实就是判断指针是否出错， 如果指针指向了内核空间的最后一页， 就说明指针是一个无效指针， 如果指针并不是落在内核空间的最后一页，就说明这指针是有效的。 无效的指针能表示成一种负数的错误码， 如果想知道这个指针是哪个错误码， 使用 PTR_ERR 函数转化。 0xfffffffffffff000~0xffffffffffffffff 这段地址和 Linux 错误码是一一对应的， 内核错误码保存在 errno-base.h 文件中。 

> /* SPDX-License-Identifier: GPL-2.0 WITH Linux-syscall-note */
> #ifndef _ASM_GENERIC_ERRNO_BASE_H
> #define _ASM_GENERIC_ERRNO_BASE_H
> #define EPERM 1 /* Operation not permitted */
> #define ENOENT 2 /* No such file or directory */
> #define ESRCH 3 /* No such process */
> #define EINTR 4 /* Interrupted system call */
> #define EIO 5 /* I/O error */
> #define ENXIO 6 /* No such device or address */
> #define E2BIG 7 /* Argument list too long */
> #define ENOEXEC 8 /* Exec format error */
> #define EBADF 9 /* Bad file number */
> #define ECHILD 10 /* No child processes */
> #define EAGAIN 11 /* Try again */
> #define ENOMEM 12 /* Out of memory */
> #define EACCES 13 /* Permission denied */
> #define EFAULT 14 /* Bad address */
> #define ENOTBLK 15 /* Block device required */
> #define EBUSY 16 /* Device or resource busy */
> #define EEXIST 17 /* File exists */
> #define EXDEV 18 /* Cross-device link */
> #define ENODEV 19 /* No such device */
> #define ENOTDIR 20 /* Not a directory */*
>
> *#define EISDIR 21 /* Is a directory */
> #define EINVAL 22 /* Invalid argument */
> #define ENFILE 23 /* File table overflow */
> #define EMFILE 24 /* Too many open files */
> #define ENOTTY 25 /* Not a typewriter */
> #define ETXTBSY 26 /* Text file busy */
> #define EFBIG 27 /* File too large */
> #define ENOSPC 28 /* No space left on device */
> #define ESPIPE 29 /* Illegal seek */
> #define EROFS 30 /* Read-only file system */
> #define EMLINK 31 /* Too many links */
> #define EPIPE 32 /* Broken pipe */
> #define EDOM 33 /* Math argument out of domain of func */
> #define ERANGE 34 /* Math result not representable */
> #endif

错误处理代码

```c
myclass = class_create(THIS_MODULE, "myclass");
if (IS_ERR(myclass)) {
	ret = PTR_ERR(myclass);
	goto fail;
}
ydevice = device_create(myclass, NULL, MKDEV(major, 0), NULL, "simple-device");
if (IS_ERR(mydevice)) {
	class_destroy(myclass);
	ret = PTR_ERR(mydevice);
	goto fail;
}
```

### 并发与并行概念

1. 并发concurrent

   并发，就是通过算法将 CPU 资源合理地分配给多个任务，当一个任务执行 I/O 操作时，CPU 可以转而执行其它的任务，等到 I/O 操作完成以后，或者新的任务遇到 I/O 操作时，CPU 再回到原来的任务继续执行。

   ![image-20240707163937172](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240707163937172.png)

   虽然 CPU 在同一时刻只能执行一个任务，但是通过将 CPU 的使用权在恰当的时机分配给不同的任务，使得多个任务看起来是一起执行的。

2. 并行parallel

   多核 CPU 的每个核心都可以独立地执行一个任务，而且多个核心之间不会相互干扰。在不同核心上执行的多个任务，是真正地同时运行，这种状态就叫做并行。双核 CPU 的工作状态如下

   ![image-20240707164050671](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240707164050671.png)

   双核 CPU 执行两个任务时，每个核心各自执行一个任务，和单核 CPU 在两个任务之间不断切换相比，它的执行效率更高。

3. 并发+并行

   在并行的工作状态中，两个 CPU 分别执行两个任务，是一种理想状态。但是在实际场景中，处于运行状态的任务是非常多的，以实际办公电脑为例，windows 系统在开机之后会运行几十个任务，而 CPU 往往只有 4 核、8 核等，远远低于任务的数量，这个时候就会同时存在并发和并行两种情况，即所有核心在并行工作的同时，每个核心还要并发工作。

   ![image-20240707164302075](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240707164302075.png)

4. 竞争

   并发可能会造成多个程序同时访问一个共享资源，这时候由并发同时访问一个共享资源产生的问题就叫做竞争。

   竞争产生的原因如下所示：

   （1）多线程的并发访问。由于 Linux 是多任务操作系统，所以多线程访问是竞争产生的基本原因。

   （2）中断程序的并发访问。中断任务产生后，CPU 会立刻停止当前工作，从而去执行中断中的任务，如果中断任务对共享资源进行了修改，就会产生竞争。

   （3）抢占式并发访问。linux2.6 及更高版本引入了抢占式内核，高优先级的任务可以打断低优先级的任务。在线程访问共享资源的时候，另一个线程打断了现在正在访问共享资源的线程同时也对共享资源进行操作，从而造成了竞争。

   （4）多处理器(SMP）并发访问。多核处理器之间存在核间并发访问。

5. 共享资源的保护

   竞争是由并发访问同一个共享资源产生的。为了防止“竞争”的产生就要对共享资源进行保护，这里提到的共享资源又是什么呢？

   以实际生活中的共享资源为例，可以是公共电话，也可以是共享单车、共享充电宝等公共物品，以上都属于共享资源的范畴，以公共电话为例，每个人都可以对它进行使用，但在同一时间内只能由一个人进行使用，如果两个人都要对电话进行使用，则产生了竞争。而在实际的驱动的代码中，共享资源可以是全局变量，也可以是驱动中的设备结构体等，需要根据具体的驱动程序来进行分析。在下一小节的实验中，会以全局变量为例，进行并发与竞争实验。

代码

```c
#include <linux/init.h>
#include <linux/module.h>
#include <linux/fs.h>
#include <linux/cdev.h>
#include <linux/kdev_t.h>
#include <linux/uaccess.h>
#include <linux/delay.h>

static int open_test(struct inode *inode,struct file *file)
{
	printk("\nthis is open_test \n");
	return 0;
}

static ssize_t read_test(struct file *file,char __user *ubuf,size_t len,loff_t *off)
{
	int ret;
	char kbuf[10] = "woniu";//定义char类型字符串变量kbuf
	printk("\nthis is read_test \n");
	ret = copy_to_user(ubuf,kbuf,strlen(kbuf));//使用copy_to_user接收用户空间传递的数据
	if (ret != 0){
		printk("copy_to_user is error \n");
	}
	printk("copy_to_user is ok \n");
	return 0;
}
static char kbuf[10] = {0};//定义char类型字符串全局变量kbuf
static ssize_t write_test(struct file *file,const char __user *ubuf,size_t len,loff_t *off)
{
	int ret;
	ret = copy_from_user(kbuf,ubuf,len);//使用copy_from_user接收用户空间传递的数据
	if (ret != 0){
		printk("copy_from_user is error\n");
	}
	if(strcmp(kbuf,"woniu") == 0 ){//如果传递的kbuf是woniu就睡眠四秒钟
		ssleep(4);
	}
	else if(strcmp(kbuf,"666") == 0){//如果传递的kbuf是666就睡眠两秒钟
		ssleep(2);
	}
	printk("copy_from_user buf is %s \n",kbuf);
	return 0;
}
static int release_test(struct inode *inode,struct file *file)
{
	//printk("\nthis is release_test \n");
	return 0;
}


struct chrdev_test {
       dev_t dev_num;//定义dev_t类型变量dev_num来表示设备号
       int major,minor;//定义int类型的主设备号major和次设备号minor
       struct cdev cdev_test;//定义struct cdev 类型结构体变量cdev_test，表示要注册的字符设备
       struct class *class_test;//定于struct class *类型结构体变量class_test，表示要创建的类
};
struct chrdev_test dev1;//创建chrdev_test类型的
struct file_operations fops_test = {
      .owner = THIS_MODULE,//将owner字段指向本模块，可以避免在模块的操作正在被使用时卸载该模块
      .open = open_test,//将open字段指向open_test(...)函数
      .read = read_test,//将read字段指向read_test(...)函数
      .write = write_test,//将write字段指向write_test(...)函数
      .release = release_test,//将release字段指向release_test(...)函数
};
 

static int __init atomic_init(void)
{
	if(alloc_chrdev_region(&dev1.dev_num,0,1,"chrdev_name") < 0 ){//自动获取设备号，设备名chrdev_name
		printk("alloc_chrdev_region is error \n");
	}
	printk("alloc_chrdev_region is ok \n");
	dev1.major = MAJOR(dev1.dev_num);//使用MAJOR()函数获取主设备号
	dev1.minor = MINOR(dev1.dev_num);//使用MINOR()函数获取次设备号
	printk("major is %d,minor is %d\n",dev1.major,dev1.minor);
	cdev_init(&dev1.cdev_test,&fops_test);//使用cdev_init()函数初始化cdev_test结构体，并链接到fops_test结构体
	dev1.cdev_test.owner = THIS_MODULE;//将owner字段指向本模块，可以避免在模块的操作正在被使用时卸载该模块
	cdev_add(&dev1.cdev_test,dev1.dev_num,1);//使用cdev_add()函数进行字符设备的添加
	dev1.class_test = class_create(THIS_MODULE,"class_test");//使用class_create进行类的创建，类名称为class_test
	device_create(dev1.class_test,0,dev1.dev_num,0,"device_test");//使用device_create进行设备的创建，设备名称为device_test
	return 0;
}

static void __exit atomic_exit(void)
{
	device_destroy(dev1.class_test,dev1.dev_num);//删除创建的设备
	class_destroy(dev1.class_test);//删除创建的类
	cdev_del(&dev1.cdev_test);//删除添加的字符设备cdev_test
	unregister_chrdev_region(dev1.dev_num,1);//释放字符设备所申请的设备号
	printk("module exit \n");
}
module_init(atomic_init);
module_exit(atomic_exit)
MODULE_LICENSE("GPL v2");
MODULE_AUTHOR("woniu");
```

应用代码

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <string.h>
 #include <unistd.h>
int main(int argc, char *argv[])
{
	int fd;//定义int类型的文件描述符
	char str1[10] = {0};//定义读取缓冲区str1
	fd = open(argv[1],O_RDWR);//调用open函数，打开输入的第一个参数文件，权限为可读可写
	if(fd < 0 ){
		printf("file open failed \n");
		return -1;
	}
	/*如果第二个参数为woniu，条件成立，调用write函数，写入woniu*/    
	if (strcmp(argv[2],"woniu") == 0 ){
		write(fd,"woniu",10);
	}
	/*如果第二个参数为666，条件成立，调用write函数，写入666*/  
	else if (strcmp(argv[2],"666") == 0 ){
		write(fd,"666",10);
	}
	close(fd); 
	return 0;
}
```

### 原子操作

对并发与竞争进行了实验，两个 app 应用程序之间对共享资源的竞争访问引起了数据传输错误，而在 Linux 内核中，提供了四种处理并发与竞争的常见方法，分别是原子操作、自旋锁、信号量、互斥体，在之后的几个章节中会依次对上述四种方法进行讲解。

在 Linux 内核中使用 atomic_t 和 atomic64_t 结构体分别来完成 32 位系统和 64 位系统的整形数据原子操作，两个结构体定义在“内核源码/include/linux/types.h”文件中，具体定义如下

```c
typedef struct {
	int counter;
} atomic_t;
#ifdef CONFIG_64BIT
typedef struct {
	long counter;
} atomic64_t;
#endif
```

可以使用以下代码定义一个 64 位系统的原子整形变量:

```c
atomic64_t v;
```

在成功定义原子变量之后，必然要对原子变量进行读取、加减等动作，原子操作的部分常用 API 函数如下所示，定义在“内核源码/include/linux/atomic.h”文件中，所以在接下来的实验中需要加入该头文件的引用。

**使用时，如果是64位系统，下面的函数都需要在atomic后加上64。**

> ATOMIC_INIT(int i) 定义原子变量的时候对其初始化，赋值为 i
>
> int atomic_read(atomic_t *v) 读取 v 的值，并且返回。
>
> void atomic_set(atomic_t *v, int i) 向原子变量 v 写入 i 值。
>
> void atomic_add(int i, atomic_t *v) 原子变量 v 加上 i 值。
>
> void atomic_sub(int i, atomic_t *v) 原子变量 v 减去 i 值。
>
> void atomic_inc(atomic_t *v) 原子变量 v 加 1
>
> void atomic_dec(atomic_t *v) 原子变量 v 减 1
>
> int atomic_dec_return(atomic_t *v) 原子变量 v 减 1，并返回 v 的值。
>
> int atomic_inc_return(atomic_t *v) 原子变量 v 加 1，并返回 v 的值。
>
> int atomic_sub_and_test(int i, atomic_t *v) 原子变量 v 减 i，如果结果为 0 就返回真，否则返回假
>
> int atomic_dec_and_test(atomic_t *v) 原子变量 v 减 1，如果结果为 0 就返回真，否则返回假
>
> int atomic_inc_and_test(atomic_t *v) 原子变量 v 加 1，如果结果为 0 就返回真，否则返回假
>
> int atomic_add_negative(int i, atomic_t *v) 原子变量 v 加 i，如果结果为负就返回真，否则返回假

下面对原子位操作进行讲解，和原子整形变量不同，原子位操作没有 atomic_t 的数据结构，原子位操作是直接对内存进行操作，原子位操作相关 API 函数如下

> void set_bit(int nr, void *p) 将 p 地址的第 nr 位置 1。
>
> void clear_bit(int nr,void *p) 将 p 地址的第 nr 位清零。
>
> void change_bit(int nr, void *p) 将 p 地址的第 nr 位进行翻转。
>
> int test_bit(int nr, void *p) 获取 p 地址的第 nr 位的值。
>
> int test_and_set_bit(int nr, void *p) 将 p 地址的第 nr 位置 1，并且返回 nr 位原来的值。
>
> int test_and_clear_bit(int nr, void *p) 将 p 地址的第 nr 位清零，并且返回 nr 位原来的值。
>
> int test_and_change_bit(int nr, void *p) 将 p 地址的第 nr 位翻转，并且返回 nr 位原来的值。

本章节实验将加入原子整形操作相关实验代码，在 open()函数和 release()函数中加入原子整形变量 v 的赋值代码，并且在 open()函数中加入原子整形变量 v 的判断代码，从而实现同一时间内只允许一个应用打开该设备节点，以此来防止共享资源竞争的产生。

```c
#include <linux/init.h>
#include <linux/module.h>
#include <linux/fs.h>
#include <linux/cdev.h>
#include <linux/kdev_t.h>
#include <linux/uaccess.h>
#include <linux/delay.h>
#include <linux/atomic.h>
#include <linux/errno.h>

static atomic64_t v = ATOMIC_INIT(1);//初始化原子类型变量v,并设置为1
static int open_test(struct inode *inode,struct file *file)
{
	if(atomic64_read(&v) != 1){//读取原子类型变量v的值并判断是否等于1
		return -EBUSY;
	}
	atomic64_set(&v,0);//将原子类型变量v的值设置为0
	//printk("\nthis is open_test \n");
	return 0;
}

static ssize_t read_test(struct file *file,char __user *ubuf,size_t len,loff_t *off)
{
	int ret;
	char kbuf[10] = "woniu";//定义char类型字符串变量kbuf
	printk("\nthis is read_test \n");
	ret = copy_to_user(ubuf,kbuf,strlen(kbuf));//使用copy_to_user接收用户空间传递的数据
	if (ret != 0){
		printk("copy_to_user is error \n");
	}
	printk("copy_to_user is ok \n");
	return 0;
}
static char kbuf[10] = {0};//定义char类型字符串全局变量kbuf
static ssize_t write_test(struct file *file,const char __user *ubuf,size_t len,loff_t *off)
{
	int ret;
	ret = copy_from_user(kbuf,ubuf,len);//使用copy_from_user接收用户空间传递的数据
	if (ret != 0){
		printk("copy_from_user is error\n");
	}
	if(strcmp(kbuf,"woniu") == 0 ){//如果传递的kbuf是woniu就睡眠四秒钟
		ssleep(4);
	}
	else if(strcmp(kbuf,"666") == 0){//如果传递的kbuf是666就睡眠两秒钟
		ssleep(2);
	}
	printk("copy_from_user buf is %s \n",kbuf);
	return 0;
}
static int release_test(struct inode *inode,struct file *file)
{
	//printk("\nthis is release_test \n");
	atomic64_set(&v,1);//将原子类型变量v的值赋1
	return 0;
}

struct chrdev_test {
       dev_t dev_num;//定义dev_t类型变量dev_num来表示设备号
       int major,minor;//定义int类型的主设备号major和次设备号minor
       struct cdev cdev_test;//定义struct cdev 类型结构体变量cdev_test，表示要注册的字符设备
       struct class *class_test;//定于struct class *类型结构体变量class_test，表示要创建的类
};
struct chrdev_test dev1;//创建chrdev_test类型的
struct file_operations fops_test = {
      .owner = THIS_MODULE,//将owner字段指向本模块，可以避免在模块的操作正在被使用时卸载该模块
      .open = open_test,//将open字段指向open_test(...)函数
      .read = read_test,//将read字段指向read_test(...)函数
      .write = write_test,//将write字段指向write_test(...)函数
      .release = release_test,//将release字段指向release_test(...)函数
};
 
static int __init atomic_init(void)
{
	if(alloc_chrdev_region(&dev1.dev_num,0,1,"chrdev_name") < 0 ){//自动获取设备号，设备名chrdev_name
		printk("alloc_chrdev_region is error \n");
	}
	printk("alloc_chrdev_region is ok \n");
	dev1.major = MAJOR(dev1.dev_num);//使用MAJOR()函数获取主设备号
	dev1.minor = MINOR(dev1.dev_num);//使用MINOR()函数获取次设备号
	printk("major is %d,minor is %d\n",dev1.major,dev1.minor);
	cdev_init(&dev1.cdev_test,&fops_test);//使用cdev_init()函数初始化cdev_test结构体，并链接到fops_test结构体
	dev1.cdev_test.owner = THIS_MODULE;//将owner字段指向本模块，可以避免在模块的操作正在被使用时卸载该模块
	cdev_add(&dev1.cdev_test,dev1.dev_num,1);//使用cdev_add()函数进行字符设备的添加
	dev1.class_test = class_create(THIS_MODULE,"class_test");//使用class_create进行类的创建，类名称为class_test
	device_create(dev1.class_test,0,dev1.dev_num,0,"device_test");//使用device_create进行设备的创建，设备名称为device_test
	return 0;
}

static void __exit atomic_exit(void)
{
	device_destroy(dev1.class_test,dev1.dev_num);//删除创建的设备
	class_destroy(dev1.class_test);//删除创建的类
	cdev_del(&dev1.cdev_test);//删除添加的字符设备cdev_test
	unregister_chrdev_region(dev1.dev_num,1);//释放字符设备所申请的设备号
	printk("module exit \n");
}
module_init(atomic_init);
module_exit(atomic_exit)
MODULE_LICENSE("GPL v2");
MODULE_AUTHOR("woniu");
```

调用代码

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <string.h>
 #include <unistd.h>
int main(int argc, char *argv[])
{
	int fd;//定义int类型的文件描述符
	char str1[10] = {0};//定义读取缓冲区str1
	fd = open(argv[1],O_RDWR);//调用open函数，打开输入的第一个参数文件，权限为可读可写
	if(fd < 0 ){
		printf("file open failed \n");
		return -1;
	}
	/*如果第二个参数为woniu，条件成立，调用write函数，写入woniu*/    
	if (strcmp(argv[2],"woniu") == 0 ){
		write(fd,"woniu",10);
	}
	/*如果第二个参数为666，条件成立，调用write函数，写入666*/  
	else if (strcmp(argv[2],"666") == 0 ){
		write(fd,"666",10);
	}
	close(fd); 
	return 0;
}
```

### 自旋锁

自旋锁是为了保护共享资源提出的一种锁机制。自旋锁（spin lock）是一种非阻塞锁，也就是说，如果某线程需要获取锁，但该锁已经被其他线程占用时，该线程不会被挂起，而是在不断的消耗 CPU 的时间，不停的试图获取锁。

在有些场景中，同步资源(用来保持一致性的两个或多个资源)的锁定时间很短，为了这一小段时间去切换线程，线程挂起和恢复现场的花费可能会让系统得不偿失。如果计算机有多个CPU 核心，能够让两个或以上的线程同时并行执行，这样我们就可以让后面那个请求锁的线程不放弃 CPU 的执行时间，直到持有锁的线程释放锁，后面请求锁的线程才可以获取锁。

为了让后面那个请求锁的线程“稍等一下”，我们需让它进行自旋，如果在自旋完成后前面锁定同步资源的线程已经释放了锁，那么该线程便不必阻塞，并且直接获取同步资源，从而避免切换线程的开销。这就是自旋锁。我们再举个形象生动的例子，以现实生活中银行 ATM 机办理业务为例，ATM 机防护舱在同一时间内只允许一个人进入，当有人进入 ATM 机防护舱之后，两秒钟之后自动上锁，其他也想要存取款的人员，只能在外部等待，办理完相应的存取款业务之后，舱内人员需要手动打开防护锁，其他人才能进入其中，办理业务。而自旋锁在驱动中的使用和上述 ATM 机办理业务流程相同，当一个任务要访问某个共享资源之前需要先获取相应的自旋锁，自旋锁只能被一个任务持有，在该任务持有自旋锁的过程中，其他任务只能原地等待该自旋锁的释放，在等待过程中的任务同样会持续占用 CPU，消耗 CPU 资源，所以临界区的代码不能太多。

如果自旋锁被错误使用可能会导致死锁的产生。

内核中以 spinlock_t 结构体来表示自旋锁，定义在“内核源码/include/linux/spinlock_types.h” 文件中

```c
typedef struct spinlock {
	union {
		struct raw_spinlock rlock;
#ifdef CONFIG_DEBUG_LOCK_ALLOC
    # define LOCK_PADSIZE (offsetof(struct raw_spinlock, dep_map))
		struct {
			u8 __padding[LOCK_PADSIZE];
			struct lockdep_map dep_map;
		};
#endif
	};
} spinlock_t;
```

自旋锁相关 API 函数定义在“内核源码/include/linux/spinlock.h”文件中，所以在本章节的实验中要加入该头文件（spinlock.h 头文件包含 spinlock_types.h 等，所以只需加入 spinlock.h 头文件即可），部分 API 函数如下

```c
DEFINE_SPINLOCK(spinlock_t lock) 定义并初始化自旋锁。
int spin_lock_init(spinlock_t *lock) 初始化自旋锁。
void spin_lock(spinlock_t *lock) 获取指定的自旋锁，也叫做加锁。
void spin_unlock(spinlock_t *lock) 释放指定的自旋锁。
int spin_trylock(spinlock_t *lock) 尝试获取指定的自旋锁，如果没有获取到就返回 0
int spin_is_locked(spinlock_t *lock) 检查指定的自旋锁是否被获取，如果没有被获取就返回非 0，否则返回 0
```

自旋锁的使用步骤：

1 在访问临界资源的时候先申请自旋锁

2 获取到自旋锁之后就进入临界区，获取不到自旋锁就“原地等待”。

3 退出临界区的时候要释放自旋锁。

代码

```c
#include <linux/init.h>
#include <linux/module.h>
#include <linux/fs.h>
#include <linux/cdev.h>
#include <linux/kdev_t.h>
#include <linux/delay.h>
#include <linux/uaccess.h>
#include <linux/spinlock.h>

static spinlock_t spinlock_test;//定义spinlock_t类型的自旋锁变量spinlock_test
static int flag = 1;//定义flag标准为，flag等于1表示设备没有被打开，等于0则证明设备已经被打开了
static int open_test(struct inode *inode,struct file *file)
{
	//printk("\nthis is open_test \n");
	spin_lock(&spinlock_test);//自旋锁加锁
	if(flag != 1){//判断标志位flag的值是否等于1
		spin_unlock(&spinlock_test);//自旋锁解锁
		return -EBUSY;
	 }
	flag = 0;//将标志位的值设置为0
	spin_unlock(&spinlock_test);//自旋锁解锁
	return 0;
}

static ssize_t read_test(struct file *file,char __user *ubuf,size_t len,loff_t *off)
{
	int ret;
	char kbuf[10] = "woniu";//定义char类型字符串变量kbuf
	printk("\nthis is read_test \n");
	ret = copy_to_user(ubuf,kbuf,strlen(kbuf));//使用copy_to_user接收用户空间传递的数据
	if (ret != 0){
		printk("copy_to_user is error \n");
	}
	printk("copy_to_user is ok \n");
	return 0;
}
static char kbuf[10] = {0};//定义char类型字符串全局变量kbuf
static ssize_t write_test(struct file *file,const char __user *ubuf,size_t len,loff_t *off)
{
	int ret;
	ret = copy_from_user(kbuf,ubuf,len);//使用copy_from_user接收用户空间传递的数据
	if (ret != 0){
		printk("copy_from_user is error\n");
	}
	if(strcmp(kbuf,"woniu") == 0 ){//如果传递的kbuf是woniu就睡眠四秒钟
		ssleep(4);
	}
	else if(strcmp(kbuf,"666") == 0){//如果传递的kbuf是666就睡眠两秒钟
		ssleep(2);
	}
	printk("copy_from_user buf is %s \n",kbuf);
	return 0;
}
static int release_test(struct inode *inode,struct file *file)
{
	printk("\nthis is release_test \n");
	spin_lock(&spinlock_test);//自旋锁加锁
	flag = 1;
	spin_unlock(&spinlock_test);//自旋锁解锁
	return 0;
}

struct chrdev_test {
       dev_t dev_num;//定义dev_t类型变量dev_num来表示设备号
       int major,minor;//定义int类型的主设备号major和次设备号minor
       struct cdev cdev_test;//定义struct cdev 类型结构体变量cdev_test，表示要注册的字符设备
       struct class *class_test;//定于struct class *类型结构体变量class_test，表示要创建的类
};
struct chrdev_test dev1;//创建chrdev_test类型的
struct file_operations fops_test = {
      .owner = THIS_MODULE,//将owner字段指向本模块，可以避免在模块的操作正在被使用时卸载该模块
      .open = open_test,//将open字段指向open_test(...)函数
      .read = read_test,//将read字段指向read_test(...)函数
      .write = write_test,//将write字段指向write_test(...)函数
      .release = release_test,//将release字段指向release_test(...)函数
};
 
static int __init atomic_init(void)
{
	spin_lock_init(&spinlock_test);
	if(alloc_chrdev_region(&dev1.dev_num,0,1,"chrdev_name") < 0 ){//自动获取设备号，设备名chrdev_name
		printk("alloc_chrdev_region is error \n");
	}
	printk("alloc_chrdev_region is ok \n");
	dev1.major = MAJOR(dev1.dev_num);//使用MAJOR()函数获取主设备号
	dev1.minor = MINOR(dev1.dev_num);//使用MINOR()函数获取次设备号
	printk("major is %d,minor is %d\n",dev1.major,dev1.minor);
	cdev_init(&dev1.cdev_test,&fops_test);//使用cdev_init()函数初始化cdev_test结构体，并链接到fops_test结构体
	dev1.cdev_test.owner = THIS_MODULE;//将owner字段指向本模块，可以避免在模块的操作正在被使用时卸载该模块
	cdev_add(&dev1.cdev_test,dev1.dev_num,1);//使用cdev_add()函数进行字符设备的添加
	dev1.class_test = class_create(THIS_MODULE,"class_test");//使用class_create进行类的创建，类名称为class_test
	device_create(dev1.class_test,0,dev1.dev_num,0,"device_test");//使用device_create进行设备的创建，设备名称为device_test
	return 0;
}

static void __exit atomic_exit(void)
{
	device_destroy(dev1.class_test,dev1.dev_num);//删除创建的设备
	class_destroy(dev1.class_test);//删除创建的类
	cdev_del(&dev1.cdev_test);//删除添加的字符设备cdev_test
	unregister_chrdev_region(dev1.dev_num,1);//释放字符设备所申请的设备号
	printk("module exit \n");
}
module_init(atomic_init);
module_exit(atomic_exit)
MODULE_LICENSE("GPL v2");
MODULE_AUTHOR("woniu");
```

调用代码同上

### 信号量

自旋锁会让请求的任务原地“自旋”，在等待的过程中会循环检测自旋锁的状态，进而占用系统资源，而本章节要讲解的信号量也是解决竞争的一种常用方法，与自旋锁不同的是，信号量会使等待的线程进入休眠状态，适用于那些占用资源比较久的场合。

信号量是操作系统中最典型的用于同步和互斥的手段，本质上是一个全局变量，信号量的值表示控制访问资源的线程数，可以根据实际情况来自行设置，如果在初始化的时候将信号量量值设置为大于 1，那么这个信号量就是计数型信号量，允许多个线程同时访问共享资源。如果将信号量量值设置为 1，那么这个信号量就是二值信号量，同一时间内只允许一个线程访问共享资源，注意！信号量的值不能小于 0。当信号量的值为 0 时，想访问共享资源的线程必须等待，直到信号量大于 0 时，等待的线程才可以访问。当访问共享资源时，信号量执行“减一”操作，访问完成后再执行“加一”操作。

相比于自旋锁，信号量具有休眠特性，因此适用长时间占用资源的场合，但由于信号量会引起休眠，所以不能用在中断函数中，最后如果共享资源的持有时间比较短，使用信号量的话会造成频繁的休眠，反而带来更多资源的消耗，使用自旋锁反而效果更好。再同时使用信号量和自旋锁的时候，要先获取信号量，再使用自旋锁，因为信号量会导致睡眠。

Linux 内核使用 semaphore 结构体来表示信号量，该结构体定义在“内核源码/include/linux/semaphore.h”文件内（所以在下一章节的信号量实验中需要加入该头文件），结构体内容如下所示

```c
struct semaphore {
	raw_spinlock_t lock;
	unsigned int count;
	struct list_head wait_list;
};
```

与信号量相关的 API 函数同样定义在 semaphore.h 文件内，部分常用 API 函数如下

> DEFINE_SEMAPHORE(name) 定义信号量，并且设置信号量的值为 1。
>
> void sema_init(struct semaphore *sem, int val) 初始化信号量 sem，设置信号量值为 val。
>
> void down(struct semaphore *sem) 获取信号量，不能被中断打断，如 ctrl+c
>
> int down_interruptible(struct semaphore *sem) 获取信号量，可以被中断打断，如 ctrl+c
>
> void up(struct semaphore *sem) 释放信号量
>
> int down_trylock(struct semaphore *sem); 尝试获取信号量，如果能获取到信号量就获取，并且返回 0。如果不能就返回非 0

实验代码

```c
#include <linux/init.h>
#include <linux/module.h>
#include <linux/fs.h>
#include <linux/cdev.h>
#include <linux/kdev_t.h>
#include <linux/uaccess.h>
#include <linux/delay.h>
#include <linux/semaphore.h>

struct semaphore semaphore_test;//定义一个semaphore类型的结构体变量semaphore_test
static int open_test(struct inode *inode,struct file *file)
{
	printk("\nthis is open_test \n");
	down(&semaphore_test);//信号量数量-1
	return 0;
}

static ssize_t read_test(struct file *file,char __user *ubuf,size_t len,loff_t *off)
{
	int ret;
	char kbuf[10] = "woniu";//定义char类型字符串变量kbuf
	printk("\nthis is read_test \n");
	ret = copy_to_user(ubuf,kbuf,strlen(kbuf));//使用copy_to_user接收用户空间传递的数据
	if (ret != 0){
		printk("copy_to_user is error \n");
	}
	printk("copy_to_user is ok \n");
	return 0;
}
static char kbuf[10] = {0};//定义char类型字符串全局变量kbuf
static ssize_t write_test(struct file *file,const char __user *ubuf,size_t len,loff_t *off)
{
	int ret;
	ret = copy_from_user(kbuf,ubuf,len);//使用copy_from_user接收用户空间传递的数据
	if (ret != 0){
		printk("copy_from_user is error\n");
	}
	if(strcmp(kbuf,"woniu") == 0 ){//如果传递的kbuf是woniu就睡眠四秒钟
		ssleep(4);
	}
	else if(strcmp(kbuf,"666") == 0){//如果传递的kbuf是666就睡眠两秒钟
		ssleep(2);
	}
	printk("copy_from_user buf is %s \n",kbuf);
	return 0;
}
static int release_test(struct inode *inode,struct file *file)
{
	up(&semaphore_test);//信号量数量加1	
	printk("\nthis is release_test \n");
	return 0;
}

struct chrdev_test {
       dev_t dev_num;//定义dev_t类型变量dev_num来表示设备号
       int major,minor;//定义int类型的主设备号major和次设备号minor
       struct cdev cdev_test;//定义struct cdev 类型结构体变量cdev_test，表示要注册的字符设备
       struct class *class_test;//定于struct class *类型结构体变量class_test，表示要创建的类
};
struct chrdev_test dev1;//创建chrdev_test类型的
struct file_operations fops_test = {
      .owner = THIS_MODULE,//将owner字段指向本模块，可以避免在模块的操作正在被使用时卸载该模块
      .open = open_test,//将open字段指向open_test(...)函数
      .read = read_test,//将read字段指向read_test(...)函数
      .write = write_test,//将write字段指向write_test(...)函数
      .release = release_test,//将release字段指向release_test(...)函数
};
 

static int __init atomic_init(void)
{
	sema_init(&semaphore_test,1);//初始化信号量结构体semaphore_test，并设置信号量的数量为1
	if(alloc_chrdev_region(&dev1.dev_num,0,1,"chrdev_name") < 0 ){//自动获取设备号，设备名chrdev_name
		printk("alloc_chrdev_region is error \n");
	}
	printk("alloc_chrdev_region is ok \n");
	dev1.major = MAJOR(dev1.dev_num);//使用MAJOR()函数获取主设备号
	dev1.minor = MINOR(dev1.dev_num);//使用MINOR()函数获取次设备号
	printk("major is %d,minor is %d\n",dev1.major,dev1.minor);
	cdev_init(&dev1.cdev_test,&fops_test);//使用cdev_init()函数初始化cdev_test结构体，并链接到fops_test结构体
	dev1.cdev_test.owner = THIS_MODULE;//将owner字段指向本模块，可以避免在模块的操作正在被使用时卸载该模块
	cdev_add(&dev1.cdev_test,dev1.dev_num,1);//使用cdev_add()函数进行字符设备的添加
	dev1.class_test = class_create(THIS_MODULE,"class_test");//使用class_create进行类的创建，类名称为class_test
	device_create(dev1.class_test,0,dev1.dev_num,0,"device_test");//使用device_create进行设备的创建，设备名称为device_test
	return 0;
}

static void __exit atomic_exit(void)
{
	device_destroy(dev1.class_test,dev1.dev_num);//删除创建的设备
	class_destroy(dev1.class_test);//删除创建的类
	cdev_del(&dev1.cdev_test);//删除添加的字符设备cdev_test
	unregister_chrdev_region(dev1.dev_num,1);//释放字符设备所申请的设备号
	printk("module exit \n");
}
module_init(atomic_init);
module_exit(atomic_exit)
MODULE_LICENSE("GPL v2");
MODULE_AUTHOR("woniu");
```

调用代码同上

### 互斥锁

在上一章节中，将信号量量值设置为 1，最终实现的就是互斥效果，与本章节要学习的互斥锁功能相同，虽然两者功能相同但是具体的实现方式是不同的，但是使用互斥锁效率更高、更简洁，所以如果使用到的信号量“量值”为 1，一般将其修改为使用互斥锁实现。

当有多个线程几乎同时修改某一个共享数据的时候，需要进行同步控制。线程同步能够保证多个线程安全访问竞争资源，最简单的同步机制是引入互斥锁。互斥锁为资源引入一个状态：锁定或者非锁定。某个线程要更改共享数据时，先将其锁定，此时资源的状态为“锁定”，其他线程不能更改；直到该线程释放资源，将资源的状态变成“非锁定”，其他的线程才能再次锁定该资源。互斥锁保证了每次只有一个线程进行写入操作，从而保证了多线程情况下数据的正确性，能够保证多个线程访问共享数据不会出现资源竞争及数据错误。

互斥锁会导致休眠，所以在中断里面不能用互斥锁。同一时刻只能有一个线程持有互斥锁，并且只有持有者才可以解锁，并且不允许递归上锁和解锁。

内核中以 mutex 结构体来表示互斥体，定义在“内核源码/include/linux/mutex.h”文件中，如下所示

```c
struct mutex {
	atomic_long_t owner;
	spinlock_t wait_lock;
#ifdef CONFIG_MUTEX_SPIN_ON_OWNER
	struct optimistic_spin_queue osq; /* Spinner MCS lock */
#endif
  struct list_head wait_list;
#ifdef CONFIG_DEBUG_MUTEXES
	void *magic;
#endif
#ifdef CONFIG_DEBUG_LOCK_ALLOC
	struct lockdep_map dep_map;
#endif
};
```

一些和互斥体相关的 API 函数也定义在 mutex.h 文件中，常用 API 函数如下

> DEFINE_MUTEX(name) 定义并初始化一个 mutex 变量。
>
> void mutex_init(mutex *lock) 初始化 mutex。
>
> void mutex_lock(struct mutex *lock) 获取 mutex，也就是给 mutex 上锁。如果获取不到就进休眠。
>
> void mutex_unlock(struct mutex *lock) 释放 mutex，也就给 mutex 解锁。
>
> int mutex_is_locked(struct mutex *lock) 判断 mutex 是否被获取，如果是的话就返回1，否则返回 0。

实验代码

```c
#include <linux/init.h>
#include <linux/module.h>
#include <linux/fs.h>
#include <linux/cdev.h>
#include <linux/kdev_t.h>
#include <linux/uaccess.h>
#include <linux/delay.h>
#include <linux/errno.h>
#include <linux/mutex.h>

struct mutex mutex_test;//定义mutex类型的互斥锁结构体变量mutex_test
static int open_test(struct inode *inode,struct file *file)
{
	printk("\nthis is open_test \n");
	mutex_lock(&mutex_test);//互斥锁加锁
	return 0;
}

static ssize_t read_test(struct file *file,char __user *ubuf,size_t len,loff_t *off)
{
	int ret;
	char kbuf[10] = "woniu";//定义char类型字符串变量kbuf
	printk("\nthis is read_test \n");
	ret = copy_to_user(ubuf,kbuf,strlen(kbuf));//使用copy_to_user接收用户空间传递的数据
	if (ret != 0){
		printk("copy_to_user is error \n");
	}
	printk("copy_to_user is ok \n");
	return 0;
}
static char kbuf[10] = {0};//定义char类型字符串全局变量kbuf
static ssize_t write_test(struct file *file,const char __user *ubuf,size_t len,loff_t *off)
{
	int ret;
	ret = copy_from_user(kbuf,ubuf,len);//使用copy_from_user接收用户空间传递的数据
	if (ret != 0){
		printk("copy_from_user is error\n");
}
	if(strcmp(kbuf,"woniu") == 0 ){//如果传递的kbuf是woniu就睡眠四秒钟
		ssleep(4);
	}
	else if(strcmp(kbuf,"666") == 0){//如果传递的kbuf是666就睡眠两秒钟
		ssleep(2);
	}
	printk("copy_from_user buf is %s \n",kbuf);
	return 0;
}
static int release_test(struct inode *inode,struct file *file)
{
	mutex_unlock(&mutex_test);//互斥锁解锁
	printk("\nthis is release_test \n");
	return 0;
}

struct chrdev_test {
       dev_t dev_num;//定义dev_t类型变量dev_num来表示设备号
       int major,minor;//定义int类型的主设备号major和次设备号minor
       struct cdev cdev_test;//定义struct cdev 类型结构体变量cdev_test，表示要注册的字符设备
       struct class *class_test;//定于struct class *类型结构体变量class_test，表示要创建的类
};
struct chrdev_test dev1;//创建chrdev_test类型的
struct file_operations fops_test = {
      .owner = THIS_MODULE,//将owner字段指向本模块，可以避免在模块的操作正在被使用时卸载该模块
      .open = open_test,//将open字段指向open_test(...)函数
      .read = read_test,//将read字段指向read_test(...)函数
      .write = write_test,//将write字段指向write_test(...)函数
      .release = release_test,//将release字段指向release_test(...)函数
};

static int __init atomic_init(void)
{
	mutex_init(&mutex_test);
	if(alloc_chrdev_region(&dev1.dev_num,0,1,"chrdev_name") < 0 ){//自动获取设备号，设备名chrdev_name
		printk("alloc_chrdev_region is error \n");
	}
	printk("alloc_chrdev_region is ok \n");
	dev1.major = MAJOR(dev1.dev_num);//使用MAJOR()函数获取主设备号
	dev1.minor = MINOR(dev1.dev_num);//使用MINOR()函数获取次设备号
	printk("major is %d,minor is %d\n",dev1.major,dev1.minor);
	cdev_init(&dev1.cdev_test,&fops_test);//使用cdev_init()函数初始化cdev_test结构体，并链接到fops_test结构体
	dev1.cdev_test.owner = THIS_MODULE;//将owner字段指向本模块，可以避免在模块的操作正在被使用时卸载该模块
	cdev_add(&dev1.cdev_test,dev1.dev_num,1);//使用cdev_add()函数进行字符设备的添加
	dev1.class_test = class_create(THIS_MODULE,"class_test");//使用class_create进行类的创建，类名称为class_test
	device_create(dev1.class_test,0,dev1.dev_num,0,"device_test");//使用device_create进行设备的创建，设备名称为device_test
	return 0;
}

static void __exit atomic_exit(void)
{
	device_destroy(dev1.class_test,dev1.dev_num);//删除创建的设备
	class_destroy(dev1.class_test);//删除创建的类
	cdev_del(&dev1.cdev_test);//删除添加的字符设备cdev_test
	unregister_chrdev_region(dev1.dev_num,1);//释放字符设备所申请的设备号
	printk("module exit \n");
}
module_init(atomic_init);
module_exit(atomic_exit)
MODULE_LICENSE("GPL v2");
MODULE_AUTHOR("woniu");
```

> 原子变量：相当于是一个int类型的变量，只是所有的操作都原子的（使用相应的函数）。
>
> 自旋锁： 当进程或线程获取不到资源（锁）时，会自旋（程序一直占用CPU，不断尝试获取资源），所以比较消耗内存和CPU，当其它进程或线程释放资源时，当前进程或线程会立即获取到资源。
>
> 信号量：当进程或线程获取不到资源时，会进入休眠状态，直到其它进程或线程释放资源后，当时进程再一次分配到时间片后再能继续执行。所以，响应速度会比自旋锁慢，但是没那么占用cpu资源。
>
> 互斥锁：是信号量为1的特殊情况

### IO模型

IO 是英文 Input 和 Output 的首字母，代表了输入和输出，当然这样的描述有一点点抽象，更直观的意思是计算机的输入与输出。在冯.诺依曼结构中，将计算机分成了 5 个部分，分别是运算器，控制器，存储器，输入设备，输出设备。如下图

![image-20240707172323650](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240707172323650.png)

上图中的输入设备指的是鼠标和键盘等向计算机输入数据和信息的设备，输出设备指的是电脑显示器等用于计算机信息输出的设备，下面对计算机输入输出过程进行实际举例，当敲击键盘（输入设备）任意按键后，按键的数据会传递给计算机，计算机 CPU 会对数据进行运算，运算完成之后会将数据输出到显示器（输出设备）上。

**IO 执行过程**

操作系统（Linux）负责对计算机的资源进行管理和对进程进行调度，应用程序运行在操作系统上，处于用户空间。应用程序不能直接对硬件进行操作，只能通过操作系统提供的 API 来操作硬件。需要将进程切换到内核空间，才能进行 IO 操作，并且应用程序不能直接操作内核空间的数据，需要把内核空间的数据拷贝到用户空间。

应用程序运行在用户空间，它不存在实质的 IO 过程，真正的 IO 是在操作系统执行的。那么应用程序操作 IO 就会有两个动作：IO 调用和 IO 执行。IO 调用是应用程序向操作系统内核发起调用，IO 执行是操作系统内核完成的 IO 操作。

一个完整的 IO 过程需要包含以下三个步骤：

> （1） 用户空间的应用程序向内核发起 IO 调用请求(系统调用)
>
> （2） 内核操作系统准备数据，把 IO 设备的数据加载到内核缓冲区
>
> （3） 操作系统拷贝数据，把内核缓冲区的数据拷贝到用户进程缓冲区

![image-20240707172540165](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240707172540165.png)

**IO 模型的分类**

假设有这样一个场景，从磁盘中循环读取 100M 的数据并处理，磁盘读取 100M 需要花费20 秒的时间，CPU 同样也需要 20 秒的时间处理完这些数据。如果采用传统的模式编写代码：读数据->等待数据读取完毕->数据处理，可以发现，数据的读取花费了一半的时间，而这就导致该任务的效率极其低下，那么能不能在等待数据的同时对数据进行处理呢？当然可以！这时候就轮到 IO 编程模型来出场了。

IO 模型根据实现的功能可以划分为为阻塞 IO、非阻塞 IO、信号驱动 IO， IO 多路复用和异步 IO。根据等待 IO 的执行结果进行划分，前四个 IO 模型又被称为同步 IO，

![image-20240707172641116](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240707172641116.png)

所谓同步，即发出一个功能调用后，只有得到结果该调用才会返回。异步的概念和同步相对。当一个异步过程调用发出后，调用者并不能立刻得到结果，实际处理这个调用的部件在完成后，通过状态、通知和回调来通知调用者。

#### **阻塞** IO

以**阻塞读**为例：进程进行 IO 操作时(如 read 操作)，首先会发起一个系统调用，从而转到内核空间进行处理，内核空间的数据没有准备就绪时，进程会被阻塞，不会继续向下执行，直到内核空间的数据准备完成后，数据才会从内核空间拷贝到用户空间，最后返回用户进程，由用户空间进行数据的处理。

![image-20240707172815915](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240707172815915.png)

#### **非阻塞** IO

和阻塞 IO 模型不同，非阻塞 IO 进行 IO 操作时，如果内核数据没有准备好，内核会立即向进程返回 err，不会进行阻塞；如果内核空间数据准备就绪，内核会立即把数据返回给用户空间的进程。

![image-20240707172921193](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240707172921193.png)

#### **IO** 多路复用

通常情况下使用 select()、poll()、epoll()函数实现 IO 多路复用。这里以 select 函数为例进行讲解，使用时可以对 select 传入多个描述符，并设置超时时间。当执行 select 的时候，系统会发起一个系统调用，内核会遍历检查传入的描述符是否有事件发生（如可读、可写事件）。如有，立即返回，否则进入睡眠状态，使进程进入阻塞状态，直到任何一个描述符事件产生后（或者等待超时）立刻返回。此时用户空间需要对全部描述符进行遍历，以确认具体是哪个发生了事件，这样就能使用一个进程对多个 IO 进行管理。

![image-20240707173116326](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240707173116326.png)

#### 信号驱动

信号驱动 IO 顾名思义与信号相关。系统在一些事件发生之后，会对进程发出特定的信号，而信号与处理函数相绑定，当信号产生时就会调用绑定的处理函数。例如在 Linux 系统任务执行的过程中可以按下 ctrl+C 来对任务进行终止，系统实际上是对该进程发送一个 SIGINT 信号，该信号的默认处理函数就是退出当前程序。

具体到 IO 模型上，可以对 SIGIO 信号注册相应的信号处理函数，并打开对应描述符的信号驱动。每当有 IO 数据产生时，系统就会发送一个 SIGIO 信号，进而调用相应的信号处理函数，从而在这个处理函数中对数据进行读取。

![image-20240707173229236](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240707173229236.png)

1. **异步** IO

   aio_read 函数常常用于异步 IO，当进程使用 aio_read 读取数据时，如果数据尚未准备就绪就立即返回，不会阻塞。若数据准备就绪就会把数据从内核空间拷贝到用户空间的缓冲区中，然后执行定义好的回调函数对接收到的数据进行处理。

   ![image-20240707173316940](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240707173316940.png)



### 阻塞IO

在 Linux 驱动程序中，阻塞进程可以使用等待队列来实现。等待队列是内核实现阻塞和唤醒的内核机制，以双循环链表为基础结构，由链表头和链表项两部分组成，分别表示等待队列头和等待队列元素。

![image-20240707173514553](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240707173514553.png)

等待队列头使用结构体 wait_queue_head_t 来表示，等待队列头是一个等待队列的头部，这个结构体定义在文件 include/linux/wait.h 里面，结构体内容如下

```c
struct _wait_queue_head{
	spinlock_t lock; //自旋锁
	struct list_head task_list //链表头
}；
typefef struct _wait_queue_head wait_queue_head_t;
```

等待队列项使用结构体 wait_queue_t 来表示，等待队列项是等待队列元素，该结构体同样定义在文件 include/linux/wait.h 里面，结构体内容如下

```c
struct _wait_queue{
	unsigned int flags;
	void *private;
  wait_queue_func_t func;
	struct list_head task_list;
};
typedef struct _wait_queue wait_queue_t;
```

#### 等待队列API

1. **定义并初始化等待队列头**

   等待队列要想被使用，第一步就是对等待队列头进行初始化，有俩种办法

   1. 方法一：使用 DECLARE_WAIT_QUEUE_HEAD 宏静态创建等待队列头，宏定义如下

      ```c
      #define DECLARE_WAIT_QUEUE_HEAD(name) \
      wait_queue_head_t name = __WAIT_QUEUE_HEAD_INITIALIZER(name)
      ```

      参数 name 表示要定义的队列头名字。通常以全局变量的方式定义,如下所示：

      ```c
      DECLARE_WAIT_QUEUE_HEAD(head);
      ```

   2. 方法二：使用 init_waitqueue_head 宏动态初始化等待队列头，宏定义如下

      ```c
      #define init_waitqueue_head(q) \
      do { \
      	static struct lock_class_key __key; \
      	\ 
      	__init_waitqueue_head((q), #q, &__key); \
      } while (0)
      ```

      参数 q 表示需要初始化的队列头指针。使用宏定义如下所示：

      ```c
      wait_queue_head_t head; //等待队列头
      init_waitqueue_head(&head); //初始化等待队列头指针
      ```

2. **创建等待队列项**

一般使用宏 DECLARE_WAITQUEUE(name,tsk)给当前正在运行的进程创建并初始化一个等待队列项，宏定义如下：

```c
#define DECLARE_WAITQUEUE(name, tsk) \
struct wait_queue_entry name = __WAITQUEUE_INITIALIZER(name, tsk)
```

第一个参数 name 是等待队列项的名字，第二个参数 tsk 表示此等待队列项属于哪个任务（进程），一般设置为 current。在 Linux 内核中 current 相当于一个全局变量，表示当前进程。创建等待队列项如下所示：

```c
DECLARE_WAITQUEUE(wait,current); //给当前正在运行的进程创建一个名为wait的等待队列项。
add_wait_queue(wq,&wait); //将 wait 这个等待队列项加到 wq 这个等待队列当中
```

3. **添加/删除队列**

当设备没有准备就绪（如没有可读数据）而需要进程阻塞的时候，就需要将进程对应的等待队列项添加到前面创建的等待队列中，只有添加到等待队列中以后进程才能进入休眠态。当设备可以访问时（如有可读数据），再将进程对应的**等待队列项**从等待队列中移除即可。

等待队列项添加队列函数如下所示：

> **函数原型****:**
>
> void add_wait_queue(wait_queue_head_t *q,wait_queue_t *wait)
>
> **函数功能****:**
>
> (通过等待队列头)向等待队列中添加队列项
>
> **参数含义****:**
>
> wq_head 表示等待队列项要加入等待队列的等待队列头
>
> wq_entry 表示要加入的等待队列项
>
> **函数返回值**
>
> 无

等待队列项移除队列函数如下所示

> **函数原型：**
>
> void remove_wait_queue(wait_queue_head_t *q,wait_queue_t *wait)
>
> **函数功能：**
>
> 要删除的等待队列项所处的等待队列头
>
> **函数含义：**
>
> 第一个参数 q 表示等待队列项要加入等待队列的等待队列头
>
> 第二个参数 wait 表示要加入的等待队列项
>
> **函数返回值：**
>
> 无

4. **等待事件**

除了主动唤醒以外，也可以设置等待队列等待某个事件，当这个事件满足以后就自动唤醒等待队列中的进程，使用如下所示的宏，是不可中断的阻塞等待。

```c
#define __wait_event(wq_head, condition) \
(void)___wait_event(wq_head, condition, TASK_UNINTERRUPTIBLE, 0, 0, \
schedule())
```

> **宏定义功能：**
>
> 不可中断的阻塞等待，让调用进程进入不可中断的睡眠状态，在等待队列里面睡眠直到condition 变成真，被内核唤醒。
>
> **参数含义：**
>
> 第一个参数 wq: wait_queue_head_t 类型变量
>
> 第二个参数 condition : 等待条件，为假时才可以进入休眠。如果 condition 为真，则不会休眠

除此之外，wait_event_interruptible 的宏是可中断的阻塞等待。

```c
#define __wait_event_interruptible(wq_head, condition) \ ___wait_event(wq_head, condition, TASK_INTERRUPTIBLE, 0, 0, \
schedule())
```

> **宏含义功能：**
>
> 可中断的阻塞等待，让调用进程进入可中断的睡眠状态，直到 condition 变成真被内核唤醒或信号打断唤醒。
>
> **参数含义：**
>
> 第一个参数 wq :wait_queue_head_t 类型变量
>
> 第二个参数 condition :等待条件。为假时才可以进入休眠。如果 condition 为真，则不会休眠。

wait_event_timeout() 宏也与 wait_event()类似.不过如果所给的睡眠时间为负数则立即返回.如果在睡眠期间被唤醒,且 condition 为真则返回剩余的睡眠时间,否则继续睡眠直到到达或超过给定的睡眠时间,然后返回 0。

wait_event_interruptible_timeout() 宏与 wait_event_timeout()类似,不过如果在睡眠期间被信号打断则返回 ERESTARTSYS 错误码。

wait_event_interruptible_exclusive() 宏同样和 wait_event_interruptible()一样,不过该睡眠的进程是一个互斥进程

**注意：调用的时要确认 condition 值是真还是假，如果调用 condition 为真，则不会休眠。**

5. **等待队列唤醒**

当设备可以使用的时候就要唤醒进入休眠态的进程，唤醒可以使用如下俩个函数

> **函数原型：**
>
> wake_up(wait_queue_head_t *q)
>
> **函数功能：**
>
> 唤醒所有休眠进程
>
> **参数含义：**
>
> q 表示要唤醒的等待队列的等待队列头

> **函数原型：**
>
> wake_up_interruptible(wait_queue_head_t *q)
>
> **函数功能：**
>
> 唤醒可中断的休眠进程
>
> **参数含义：**
>
> q 表示要唤醒的等待队列的等待队列头

#### 等待队列使用

步骤一：初始化等待队列头，并将条件置成假(condition=0)。

步骤二：在需要阻塞的地方调用 wait_event()，使进程进入休眠状态。

步骤三：当条件满足时，需要解除休眠，先将条件(condition=1),然后调用 wake_up 函数唤醒等待队列中的休眠进程。

代码

```c
#include <linux/init.h>
#include <linux/module.h>
#include <linux/kdev_t.h>
#include <linux/fs.h>
#include <linux/cdev.h>
#include <linux/uaccess.h>
#include <linux/io.h>
#include  <linux/wait.h>


struct device_test{
   
    dev_t dev_num;  //设备号
    int major ;  //主设备号
    int minor ;  //次设备号
    struct cdev cdev_test; // cdev
    struct class *class;   //类
    struct device *device; //设备
    char kbuf[32];
    int  flag;  //标志位
};


struct  device_test dev1;  

DECLARE_WAIT_QUEUE_HEAD(read_wq); //定义并初始化等待队列头

/*打开设备函数*/
static int cdev_test_open(struct inode *inode, struct file *file)
{
    file->private_data=&dev1;//设置私有数据
    printk("This is cdev_test_open\r\n");

    return 0;
}

/*向设备写入数据函数*/
static ssize_t cdev_test_write(struct file *file, const char __user *buf, size_t size, loff_t *off)
{
     struct device_test *test_dev=(struct device_test *)file->private_data;

    if (copy_from_user(test_dev->kbuf, buf, size) != 0) // copy_from_user:用户空间向内核空间传数据
    {
        printk("copy_from_user error\r\n");
        return -1;
    }
    test_dev->flag=1;//将条件置1
    wake_up_interruptible(&read_wq); //并使用wake_up_interruptible唤醒等待队列中的休眠进程
	
    return 0;
}

/**从设备读取数据*/
static ssize_t cdev_test_read(struct file *file, char __user *buf, size_t size, loff_t *off)
{
    
    struct device_test *test_dev=(struct device_test *)file->private_data;

    wait_event_interruptible(read_wq,test_dev->flag); //可中断的阻塞等待，使进程进入休眠态
    if (copy_to_user(buf, test_dev->kbuf, strlen( test_dev->kbuf)) != 0) // copy_to_user:内核空间向用户空间传数据
    {
        printk("copy_to_user error\r\n");
        return -1;
    }
	datap->flag = 0;
    return 0;
}

static int cdev_test_release(struct inode *inode, struct file *file)
{
    
    return 0;
}

/*设备操作函数*/
struct file_operations cdev_test_fops = {
    .owner = THIS_MODULE, //将owner字段指向本模块，可以避免在模块的操作正在被使用时卸载该模块
    .open = cdev_test_open, //将open字段指向chrdev_open(...)函数
    .read = cdev_test_read, //将open字段指向chrdev_read(...)函数
    .write = cdev_test_write, //将open字段指向chrdev_write(...)函数
    .release = cdev_test_release, //将open字段指向chrdev_release(...)函数
};

static int __init chr_fops_init(void) //驱动入口函数
{
    /*注册字符设备驱动*/
    int ret;
    /*1 创建设备号*/
    ret = alloc_chrdev_region(&dev1.dev_num, 0, 1, "alloc_name"); //动态分配设备号
    if (ret < 0)
    {
       goto err_chrdev;
    }
    printk("alloc_chrdev_region is ok\n");

    dev1.major = MAJOR(dev1.dev_num); //获取主设备号
   dev1.minor = MINOR(dev1.dev_num); //获取次设备号

    printk("major is %d \r\n", dev1.major); //打印主设备号
    printk("minor is %d \r\n", dev1.minor); //打印次设备号
     /*2 初始化cdev*/
    dev1.cdev_test.owner = THIS_MODULE;
    cdev_init(&dev1.cdev_test, &cdev_test_fops);

    /*3 添加一个cdev,完成字符设备注册到内核*/
   ret =  cdev_add(&dev1.cdev_test, dev1.dev_num, 1);
    if(ret<0)
    {
        goto  err_chr_add;
    }
    /*4 创建类*/
  dev1. class = class_create(THIS_MODULE, "test");
    if(IS_ERR(dev1.class))
    {
        ret=PTR_ERR(dev1.class);
        goto err_class_create;
    }
    /*5  创建设备*/
  dev1.device = device_create(dev1.class, NULL, dev1.dev_num, NULL, "test");
    if(IS_ERR(dev1.device))
    {
        ret=PTR_ERR(dev1.device);
        goto err_device_create;
    }

return 0;

 err_device_create:
        class_destroy(dev1.class);                 //删除类

err_class_create:
       cdev_del(&dev1.cdev_test);                 //删除cdev

err_chr_add:
        unregister_chrdev_region(dev1.dev_num, 1); //注销设备号

err_chrdev:
        return ret;
}




static void __exit chr_fops_exit(void) //驱动出口函数
{
    /*注销字符设备*/
    unregister_chrdev_region(dev1.dev_num, 1); //注销设备号
    cdev_del(&dev1.cdev_test);                 //删除cdev
    device_destroy(dev1.class, dev1.dev_num);       //删除设备
    class_destroy(dev1.class);                 //删除类
}
module_init(chr_fops_init);
module_exit(chr_fops_exit);
MODULE_LICENSE("GPL v2");
MODULE_AUTHOR("topeet");
```

调用代码read

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <stdlib.h>

int main(int argc, char *argv[])  
{
    int fd;
    char buf1[32] = {0};   
    char buf2[32] = {0};
    fd = open("/dev/test", O_RDWR);  //打开led驱动
    if (fd < 0)
    {
        perror("open error \n");
        return fd;
    }
    printf("read before \n");
    read(fd,buf1,sizeof(buf1));  //从/dev/test文件读取数据
    printf("buf is %s  \n",buf1);
    printf("read after \n");
    close(fd);     //关闭文件
    return 0;
}
```

调用代码write

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <stdlib.h>
#include <string.h>

int main(int argc, char *argv[])  
{
    int fd;
    char buf1[32] = {0};   
    char buf2[32] = "nihao";
    fd = open("/dev/test", O_RDWR);  //打开led驱动
    if (fd < 0)
    {
        perror("open error \n");
        return fd;
    }
    printf("write before \n");
    write(fd,buf2,sizeof(buf2));  //向/dev/test文件写入数据
     printf("write after\n");
    close(fd);     //关闭文件
    return 0;
}
```

### 非阻塞IO

如果应用程序要采用非阻塞的方式来访问驱动设备文件，可以使用如下所示代码

```c
int fd;
int data = 0;
fd = open("/dev/xxx_dev", O_RDWR | O_NONBLOCK); /* 非阻塞方式打开 */
ret = read(fd, &data, sizeof(data)); /* 读取数据 */
```

使用 open 函数打开“/dev/xxx_dev”设备文件的时候添加了参数“O_NONBLOCK”，表示以非阻塞方式打开设备，这样从设备中读取数据的时候是非阻塞方式了。

驱动代码：

```c
#include <linux/init.h>
#include <linux/module.h>
#include <linux/kdev_t.h>
#include <linux/fs.h>
#include <linux/cdev.h>
#include <linux/uaccess.h>
#include <linux/io.h>
#include  <linux/wait.h>


struct device_test{
   
    dev_t dev_num;  //设备号
     int major ;  //主设备号
    int minor ;  //次设备号
    struct cdev cdev_test; // cdev
    struct class *class;   //类
    struct device *device; //设备
    char kbuf[32];
    int  flag;  //标志位
};


struct  device_test dev1;  

DECLARE_WAIT_QUEUE_HEAD(read_wq); //定义并初始化等待队列头

/*打开设备函数*/
static int cdev_test_open(struct inode *inode, struct file *file)
{
    file->private_data=&dev1;//设置私有数据
    
    return 0;
}

/*向设备写入数据函数*/
static ssize_t cdev_test_write(struct file *file, const char __user *buf, size_t size, loff_t *off)
{
     struct device_test *test_dev=(struct device_test *)file->private_data;

    if (copy_from_user(test_dev->kbuf, buf, size) != 0) // copy_from_user:用户空间向内核空间传数据
    {
        printk("copy_from_user error\r\n");
        return -1;
    }
    test_dev->flag=1;    //将条件置1，并使用wake_up_interruptible唤醒等待队列中的休眠进程
    wake_up_interruptible(&read_wq); 

    return 0;
}

/**从设备读取数据*/
static ssize_t cdev_test_read(struct file *file, char __user *buf, size_t size, loff_t *off)
{
    
    struct device_test *test_dev=(struct device_test *)file->private_data;
    if(file->f_flags & O_NONBLOCK ){
        if (test_dev->flag !=1)
        return -EAGAIN;
    }
    wait_event_interruptible(read_wq,test_dev->flag); //可中断的阻塞等待，使进程进入休眠态

    if (copy_to_user(buf, test_dev->kbuf, strlen( test_dev->kbuf)) != 0) // copy_to_user:内核空间向用户空间传数据
    {
        printk("copy_to_user error\r\n");
        return -1;
    }
    return 0;
}

static int cdev_test_release(struct inode *inode, struct file *file)
{

    return 0;
}

/*设备操作函数*/
struct file_operations cdev_test_fops = {
    .owner = THIS_MODULE, //将owner字段指向本模块，可以避免在模块的操作正在被使用时卸载该模块
    .open = cdev_test_open, //将open字段指向chrdev_open(...)函数
    .read = cdev_test_read, //将open字段指向chrdev_read(...)函数
    .write = cdev_test_write, //将open字段指向chrdev_write(...)函数
    .release = cdev_test_release, //将open字段指向chrdev_release(...)函数
};

static int __init chr_fops_init(void) //驱动入口函数
{
    /*注册字符设备驱动*/
    int ret;
    /*1 创建设备号*/
    ret = alloc_chrdev_region(&dev1.dev_num, 0, 1, "alloc_name"); //动态分配设备号
    if (ret < 0)
    {
       goto err_chrdev;
    }
    printk("alloc_chrdev_region is ok\n");

    dev1.major = MAJOR(dev1.dev_num); //获取主设备号
   dev1.minor = MINOR(dev1.dev_num); //获取次设备号

    printk("major is %d \r\n", dev1.major); //打印主设备号
    printk("minor is %d \r\n", dev1.minor); //打印次设备号
     /*2 初始化cdev*/
    dev1.cdev_test.owner = THIS_MODULE;
    cdev_init(&dev1.cdev_test, &cdev_test_fops);

    /*3 添加一个cdev,完成字符设备注册到内核*/
   ret =  cdev_add(&dev1.cdev_test, dev1.dev_num, 1);
    if(ret<0)
    {
        goto  err_chr_add;
    }
    /*4 创建类*/
  dev1. class = class_create(THIS_MODULE, "test");
    if(IS_ERR(dev1.class))
    {
        ret=PTR_ERR(dev1.class);
        goto err_class_create;
    }
    /*5  创建设备*/
  dev1.device = device_create(dev1.class, NULL, dev1.dev_num, NULL, "test");
    if(IS_ERR(dev1.device))
    {
        ret=PTR_ERR(dev1.device);
        goto err_device_create;
    }

return 0;

 err_device_create:
        class_destroy(dev1.class);                 //删除类

err_class_create:
       cdev_del(&dev1.cdev_test);                 //删除cdev

err_chr_add:
        unregister_chrdev_region(dev1.dev_num, 1); //注销设备号

err_chrdev:
        return ret;
}




static void __exit chr_fops_exit(void) //驱动出口函数
{
    /*注销字符设备*/
    unregister_chrdev_region(dev1.dev_num, 1); //注销设备号
    cdev_del(&dev1.cdev_test);                 //删除cdev
    device_destroy(dev1.class, dev1.dev_num);       //删除设备
    class_destroy(dev1.class);                 //删除类
}
module_init(chr_fops_init);
module_exit(chr_fops_exit);
MODULE_LICENSE("GPL v2");
MODULE_AUTHOR("woniu");
```

### IO 多路复用

IO 多路复用是一种同步的 IO 模型。IO 多路复用可以实现一个进程监视多个文件描述符。一旦某个文件描述符准备就绪，就通知应用程序进行相应的读写操作。没有文件描述符就绪时就会阻塞应用程序，从而释放出 CPU 资源

在应用层 Linux 提供了三种实现 IO 多路复用的模型，分别是 select、poll 和 epoll。

首先来学习下 select、poll 和 epoll 函数有什么区别呢？poll 函数和 seslect 函数都可以监听多个文件描述符，通过轮询来获取已经准备好的文件描述符。但是 epoll 函数将主动轮询变成了被动通知，当事件发生时被动接收通知。

select,poll,epoll 有什么区别呢？在单个线程中，select 函数最大可以监视 1024 个文件描述符，而 poll 函数和 select 函数并没有什么区别，只是 poll 函数没有最大文件描述符的限制。在本章节的实验中，以 poll 为例进行实验。在 Linux 应用程序中 poll 函数如下所示

> **函数原型：**
>
> int poll(struct pollfd *fds, nfds_t nfds,int timeout);
>
> **函数功能：**
>
> 监视并等待多个文件描述符的属性变化
>
> **函数参数：**
>
> **第一个参数** **fds**: 要监视的文件描述符集合以及要监视的事件，为一个数组，数组元素都是结构体 pollfd 类型

pollfd 结构体如下

```c
struct pollfd {
	int fd; //被监视的文件描述符
	short events; //等待的事件
	short revents; //实际发生的事件
}；
```

在 pollfd 结构体中，第一个成员 fd 是被监视的文件描述符。第二个成员 events 是要监视的事件，可监视的事件类型如下

> POLLIN 有数据可以读取
>
> POLLPRI 有紧急的数据需要读取
>
> POLLOUT 可以写数据
>
> POLLERR 指定的文件描述符发生错误
>
> POLLHUP 指定的文件描述符挂起
>
> POLLNVAL 无效的请求
>
> POLLRDNORM 等同于 POLLIN

> 第三个成员是返回事件，由 Linux 内核设置具体的返回事件。
>
> **第二个参数** **nfds**: poll 函数要监视的文件描述符数量
>
> **第三个参数** **timeout**:指定等待的时间，单位是 ms。无论 I/O 是否准备好，时间到 POLL就会返回。如果 timepoll 大于 0 等待指定的时间，如果 timeout 等于 0，立即返回。如果 timeout等于-1，事件发生以后才返回。
>
> **函数返回值：**
>
> 失败返回-1，成功返回 revents 不为 0 

当应用程序使用 select 或者 poll 函数对驱动程序进行非阻塞访问时，驱动程序中file_operations 操作集的 poll 函数会执行。所以需要完善驱动中的 poll 函数。驱动中的 poll 函数原型如下所示

```c
unsigned int (*poll)(struct file *filp,struct poll_table_struct *wait);
```

> **函数参数：**
>
> filp:要打开的文件描述符
>
> wait: 结构体 poll_table_struct 类型指针，此参数是由应用程序中传递的。一般此参数要传递给 poll_wait 函数。
>
> **返回值：**
>
> 向应用程序返回资源状态，可以返回的资源状态如下：
>
> POLLIN 有数据可以读取
>
> POLLPRI 有紧急的数据需要读取
>
> POLLOUT 可以写数据
>
> POLLERR 
>
> 指定的文件描述符发生错误
>
> POLLHUP 
>
> 指定的文件描述符挂起
>
> POLLNVAL 
>
> 无效的请求
>
> POLLRDNORM 等同于 POLLIN，普通数据可读。
>
> **函数功能：**
>
> 这个函数要进行下面两项工作。首先，对可能引起设备文件状态变化的等待队列调用poll_wait(),将对应的等待队列头添加到 poll_table.然后返回表示是否能对设备进行无阻塞读写访问的掩码。

驱动程序的 poll 函数中调用 poll_wait 函数，**注意！poll_wait 函数是不会引起阻塞的。poll_wait 函数原型如下所示**

```c
void poll_wait(struct file *filp,wait_queue_head_t *queue,poll_table *wait);
```

参数 queue 是要添加到 poll_table 中的等待队列头，参数 wait 是 poll_table，也就是file_operations 中 poll 函数的 wait 参数。

调用代码read

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <stdlib.h>
#include <poll.h>

int main(int argc, char *argv[])  
{
    int fd;//要监视的文件描述符
    char buf1[32] = {0};   
    char buf2[32] = {0};
    struct pollfd  fds[1];
    int ret;
    fd = open("/dev/test", O_RDWR);  //打开/dev/test设备，阻塞式访问
    if (fd < 0)
    {
        perror("open error \n");
        return fd;
    }

    fds[0].fd =fd;
    fds[0].events = POLLIN;
    printf("read before \n");
    while (1)
    {
        ret = poll(fds,1,3000);
        if(!ret){
            printf("time out !!\n");

        }else if(fds[0].revents == POLLIN)
        {
            read(fd,buf1,sizeof(buf1));  //从/dev/test文件读取数据
             printf("buf is %s \n",buf1);
             sleep(1);
        }
    }
    
    printf("read after\n");
    close(fd);     //关闭文件
    return 0;
}
```

调用代码write

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <stdlib.h>
#include <string.h>

int main(int argc, char *argv[])  
{
    int fd;
    char buf1[32] = {0};   
    char buf2[32] = "nihao";
    fd = open("/dev/test",O_RDWR);  //打开led驱动
    if (fd < 0)
    {
        perror("open error \n");
        return fd;
    }
    printf("write before \n");
    write(fd,buf2,sizeof(buf2));  //向/dev/test文件写入数据
     printf("write after\n");
    close(fd);     //关闭文件
    return 0;
}
```

驱动代码

```c
#include <linux/init.h>
#include <linux/module.h>
#include <linux/kdev_t.h>
#include <linux/fs.h>
#include <linux/cdev.h>
#include <linux/uaccess.h>
#include <linux/io.h>
#include  <linux/wait.h>
#include <linux/poll.h>

struct device_test{
   
    dev_t dev_num;  //设备号
     int major ;  //主设备号
    int minor ;  //次设备号
    struct cdev cdev_test; // cdev
    struct class *class;   //类
    struct device *device; //设备
    char kbuf[32];
    int  flag;  //标志位
};


struct  device_test dev1;  

DECLARE_WAIT_QUEUE_HEAD(read_wq); //定义并初始化等待队列头

/*打开设备函数*/
static int cdev_test_open(struct inode *inode, struct file *file)
{
    file->private_data=&dev1;//设置私有数据
   
    return 0;
}

/*向设备写入数据函数*/
static ssize_t cdev_test_write(struct file *file, const char __user *buf, size_t size, loff_t *off)
{
     struct device_test *test_dev=(struct device_test *)file->private_data;

    if (copy_from_user(test_dev->kbuf, buf, size) != 0) // copy_from_user:用户空间向内核空间传数据
    {
        printk("copy_from_user error\r\n");
        return -1;
    }
    test_dev->flag=1;
    wake_up_interruptible(&read_wq);

    return 0;
}

/**从设备读取数据*/
static ssize_t cdev_test_read(struct file *file, char __user *buf, size_t size, loff_t *off)
{
    
    struct device_test *test_dev=(struct device_test *)file->private_data;
    if(file->f_flags & O_NONBLOCK ){
        if (test_dev->flag !=1)
        return -EAGAIN;
    }
    wait_event_interruptible(read_wq,test_dev->flag);

    if (copy_to_user(buf, test_dev->kbuf, strlen( test_dev->kbuf)) != 0) // copy_to_user:内核空间向用户空间传数据
    {
        printk("copy_to_user error\r\n");
        return -1;
    }

    return 0;
}

static int cdev_test_release(struct inode *inode, struct file *file)
{
    
    return 0;
}

static  __poll_t  cdev_test_poll(struct file *file, struct poll_table_struct *p){
     struct device_test *test_dev=(struct device_test *)file->private_data;  //设置私有数据
     __poll_t mask=0;    
     poll_wait(file,&read_wq,p);     //应用阻塞

     if (test_dev->flag == 1)    
     {
         mask |= POLLIN;
     }
     return mask;
     
}

/*设备操作函数*/
struct file_operations cdev_test_fops = {
    .owner = THIS_MODULE, //将owner字段指向本模块，可以避免在模块的操作正在被使用时卸载该模块
    .open = cdev_test_open, //将open字段指向chrdev_open(...)函数
    .read = cdev_test_read, //将open字段指向chrdev_read(...)函数
    .write = cdev_test_write, //将open字段指向chrdev_write(...)函数
    .release = cdev_test_release, //将open字段指向chrdev_release(...)函数
    .poll = cdev_test_poll,  //将poll字段指向chrdev_poll(...)函数
};

static int __init chr_fops_init(void) //驱动入口函数
{
    /*注册字符设备驱动*/
    int ret;
    /*1 创建设备号*/
    ret = alloc_chrdev_region(&dev1.dev_num, 0, 1, "alloc_name"); //动态分配设备号
    if (ret < 0)
    {
       goto err_chrdev;
    }
    printk("alloc_chrdev_region is ok\n");

    dev1.major = MAJOR(dev1.dev_num); //获取主设备号
   dev1.minor = MINOR(dev1.dev_num); //获取次设备号

    printk("major is %d \r\n", dev1.major); //打印主设备号
    printk("minor is %d \r\n", dev1.minor); //打印次设备号
     /*2 初始化cdev*/
    dev1.cdev_test.owner = THIS_MODULE;
    cdev_init(&dev1.cdev_test, &cdev_test_fops);

    /*3 添加一个cdev,完成字符设备注册到内核*/
   ret =  cdev_add(&dev1.cdev_test, dev1.dev_num, 1);
    if(ret<0)
    {
        goto  err_chr_add;
    }
    /*4 创建类*/
  dev1. class = class_create(THIS_MODULE, "test");
    if(IS_ERR(dev1.class))
    {
        ret=PTR_ERR(dev1.class);
        goto err_class_create;
    }
    /*5  创建设备*/
  dev1.device = device_create(dev1.class, NULL, dev1.dev_num, NULL, "test");
    if(IS_ERR(dev1.device))
    {
        ret=PTR_ERR(dev1.device);
        goto err_device_create;
    }

return 0;

 err_device_create:
        class_destroy(dev1.class);                 //删除类

err_class_create:
       cdev_del(&dev1.cdev_test);                 //删除cdev

err_chr_add:
        unregister_chrdev_region(dev1.dev_num, 1); //注销设备号

err_chrdev:
        return ret;
}




static void __exit chr_fops_exit(void) //驱动出口函数
{
    /*注销字符设备*/
    unregister_chrdev_region(dev1.dev_num, 1); //注销设备号
    cdev_del(&dev1.cdev_test);                 //删除cdev
    device_destroy(dev1.class, dev1.dev_num);       //删除设备
    class_destroy(dev1.class);                 //删除类
}
module_init(chr_fops_init);
module_exit(chr_fops_exit);
MODULE_LICENSE("GPL v2");
MODULE_AUTHOR("woniu");
```

### 信号驱动 IO

信号驱动 IO 不需要应用程序查询设备的状态，一旦设备准备就绪，会触发 SIGIO 信号，进而调用注册的处理函数。

如果要实现信号驱动 IO，需要应用程序和驱动程序配合，应用程序使用信号驱动 IO 的步骤有三步：

> 步骤 1： 注册信号处理函数 应用程序使用 signal 函数来注册 SIGIO 信号的信号处理函数。
>
> 步骤 2： 设置能够接收这个信号的进程
>
> 步骤 3： 开启信号驱动 IO 通常使用 fcntl 函数的 F_SETFL 命令打开 FASYNC 标志。

fcntl 函数如下所示：

> **函数原型**：
>
> int fcntl(int fd,int cmd, ...) 
>
> **函数功能**：
>
> fcntl 函数可以用来操作文件描述符
>
> **函数参数**:
>
> fd: 被操作的文件描述符
>
> cmd: 操作文件描述符的命令，cmd 参数决定了要如何操作文件描述符 fd
>
> ...: 根据 cmd 的参数来决定是不是需要使用第三个参数

操作文件描述符的命令如下

> F_DUPFD 复制文件描述符
>
> F_GETFD 获取文件描述符标志
>
> F_SETFD 设置文件描述符标志
>
> F_GETFL 获取文件状态标志
>
> F_SETFL 设置文件状态标志
>
> F_GETLK 获取文件锁
>
> F_SETLK 设置文件锁
>
> F_SETLKW 类似 F_SETLK，但等待返回
>
> F_GETOWN 获取当前接收 SIGIO 和 SIGURG 信号的进程 ID 和进程组 ID
>
> F_SETOWN 设置当前接收 SIGIO 和 SIGURG 信号的进程 ID 和进程组 ID

接下来学习驱动程序实现 fasync 方法.

1. 步骤 1

   当应用程序开启信号驱动 IO 时，会触发驱动中的 fasync 函数。所以首先在 file_operations结构体中实现 fasync 函数，函数原型如下：

   ```c
   int (*fasync) (int fd,struct file *filp,int on)
   ```

2. 步骤 2

   在驱动中的 fasync 函数调用 fasync_helper 函数来操作 fasync_struct 结构体，fasync_helper函数原型如下

   ```c
   int fasync_helper(int fd,struct file *filp,int on,struct fasync_struct **fapp)
   ```

3. 步骤 3

   当设备准备好的时候，驱动程序需要调用 kill_fasync 函数通知应用程序，此时应用程序的SIGIO 信号处理函数就会被执行。kill_fasync 负责发送指定的信号，函数原型如下：

   ```c
   void kill_fasync(struct fasync_struct **fp,int sig,int band)
   ```

   > **函数参数：**
   >
   > fp: 要操作的 fasync_struct
   >
   > sig: 发送的信号
   >
   > band: 可读的时候设置成 POLLIN ，可写的时候设置成 POLLOUT

调用代码write

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <stdlib.h>
#include <string.h>

int main(int argc, char *argv[])  
{
    int fd;
    char buf1[32] = {0};   
    char buf2[32] = "nihao";
    fd = open("/dev/test",O_RDWR);  //打开led驱动
    if (fd < 0)
    {
        perror("open error \n");
        return fd;
    }
    printf("write before \n");
    write(fd,buf2,sizeof(buf2));  //向/dev/test文件写入数据
     printf("write after\n");
    close(fd);     //关闭文件
    return 0;
}
```

调用代码read

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <stdlib.h>
#include <poll.h>
#include <fcntl.h>
#include <signal.h>

int fd;
char buf1[32] = {0};   


//SIGIO信号的信号处理函数
static void func(int signum)
{
    read(fd,buf1,32);
    printf ("buf is %s\n",buf1);
}
int main(int argc, char *argv[])  
{
    int ret;
    int flags;
       fd = open("/dev/test", O_RDWR);  //打开led驱动
    if (fd < 0)
    {
        perror("open error \n");
        return fd;
    }

    signal(SIGIO,func);  //步骤一：使用signal函数注册SIGIO信号的信号处理函数
     //步骤二：设置能接收这个信号的进程
     //fcntl函数用来操作文件描述符，
     //F_SETOWN 设置当前接收的SIGIO的进程ID
     fcntl(fd,F_SETOWN,getpid()); 
	
    flags = fcntl(fd,F_GETFL); //获取文件描述符标志
    //步骤三  开启信号驱动IO 使用fcntl函数的F_SETFL命令打开FASYNC标志
    fcntl(fd,F_SETFL,flags| FASYNC);
    while(1);
    
    close(fd);     //关闭文件
    return 0;
}
```

驱动代码

```c
#include <linux/init.h>
#include <linux/module.h>
#include <linux/kdev_t.h>
#include <linux/fs.h>
#include <linux/cdev.h>
#include <linux/uaccess.h>
#include <linux/io.h>
#include  <linux/wait.h>
#include <linux/poll.h>
#include <linux/fcntl.h>
#include <linux/signal.h>

struct device_test{
   
    dev_t dev_num;  //设备号a
     int major ;  //主设备号
    int minor ;  //次设备号
    struct cdev cdev_test; // cdev
    struct class *class;   //类
    struct device *device; //设备
    char kbuf[32];
    int  flag;  //标志位
    struct fasync_struct *fasync;
};


struct  device_test dev1;  

DECLARE_WAIT_QUEUE_HEAD(read_wq); //定义并初始化等待队列头

/*打开设备函数*/
static int cdev_test_open(struct inode *inode, struct file *file)
{
    file->private_data=&dev1;//设置私有数据
   
    return 0;
}

/*向设备写入数据函数*/
static ssize_t cdev_test_write(struct file *file, const char __user *buf, size_t size, loff_t *off)
{
     struct device_test *test_dev=(struct device_test *)file->private_data;

    if (copy_from_user(test_dev->kbuf, buf, size) != 0) // copy_from_user:用户空间向内核空间传数据
    {
        printk("copy_from_user error\r\n");
        return -1;
    }
    test_dev->flag=1;
    wake_up_interruptible(&read_wq);
    kill_fasync(&test_dev->fasync,SIGIO,POLLIN);
    return 0;
}

/**从设备读取数据*/
static ssize_t cdev_test_read(struct file *file, char __user *buf, size_t size, loff_t *off)
{
    
    struct device_test *test_dev=(struct device_test *)file->private_data;
    if(file->f_flags & O_NONBLOCK ){
        if (test_dev->flag !=1)
        return -EAGAIN;
    }
    wait_event_interruptible(read_wq,test_dev->flag);

    if (copy_to_user(buf, test_dev->kbuf, strlen( test_dev->kbuf)) != 0) // copy_to_user:内核空间向用户空间传数据
    {
        printk("copy_to_user error\r\n");
        return -1;
    }

    return 0;
}

static int cdev_test_release(struct inode *inode, struct file *file)
{
    
    return 0;
}

static  __poll_t  cdev_test_poll(struct file *file, struct poll_table_struct *p){
     struct device_test *test_dev=(struct device_test *)file->private_data;  //设置私有数据
     __poll_t mask=0;    
     poll_wait(file,&read_wq,p);     //应用阻塞

     if (test_dev->flag == 1)    
     {
         mask |= POLLIN;
     }
     return mask;
     
}

static int cdev_test_fasync (int fd, struct file *file, int on)
{
     struct device_test *test_dev=(struct device_test *)file->private_data;  //设置私有数据
    return  fasync_helper(fd,file,on,&test_dev->fasync);
}
/*设备操作函数*/
struct file_operations cdev_test_fops = {
    .owner = THIS_MODULE, //将owner字段指向本模块，可以避免在模块的操作正在被使用时卸载该模块
    .open = cdev_test_open, //将open字段指向chrdev_open(...)函数
    .read = cdev_test_read, //将open字段指向chrdev_read(...)函数
    .write = cdev_test_write, //将open字段指向chrdev_write(...)函数
    .release = cdev_test_release, //将open字段指向chrdev_release(...)函数
    .poll = cdev_test_poll,  //将poll字段指向chrdev_poll(...)函数
    .fasync = cdev_test_fasync,   //将fasync字段指向cdev_test_fasync(...)函数
};

static int __init chr_fops_init(void) //驱动入口函数
{
    /*注册字符设备驱动*/
    int ret;
    /*1 创建设备号*/
    ret = alloc_chrdev_region(&dev1.dev_num, 0, 1, "alloc_name"); //动态分配设备号
    if (ret < 0)
    {
       goto err_chrdev;
    }
    printk("alloc_chrdev_region is ok\n");

    dev1.major = MAJOR(dev1.dev_num); //获取主设备号
   dev1.minor = MINOR(dev1.dev_num); //获取次设备号

    printk("major is %d \r\n", dev1.major); //打印主设备号
    printk("minor is %d \r\n", dev1.minor); //打印次设备号
     /*2 初始化cdev*/
    dev1.cdev_test.owner = THIS_MODULE;
    cdev_init(&dev1.cdev_test, &cdev_test_fops);

    /*3 添加一个cdev,完成字符设备注册到内核*/
   ret =  cdev_add(&dev1.cdev_test, dev1.dev_num, 1);
    if(ret<0)
    {
        goto  err_chr_add;
    }
    /*4 创建类*/
  dev1. class = class_create(THIS_MODULE, "test");
    if(IS_ERR(dev1.class))
    {
        ret=PTR_ERR(dev1.class);
        goto err_class_create;
    }
    /*5  创建设备*/
  dev1.device = device_create(dev1.class, NULL, dev1.dev_num, NULL, "test");
    if(IS_ERR(dev1.device))
    {
        ret=PTR_ERR(dev1.device);
        goto err_device_create;
    }

return 0;

 err_device_create:
        class_destroy(dev1.class);                 //删除类

err_class_create:
       cdev_del(&dev1.cdev_test);                 //删除cdev

err_chr_add:
        unregister_chrdev_region(dev1.dev_num, 1); //注销设备号

err_chrdev:
        return ret;
}




static void __exit chr_fops_exit(void) //驱动出口函数
{
    /*注销字符设备*/
    unregister_chrdev_region(dev1.dev_num, 1); //注销设备号
    cdev_del(&dev1.cdev_test);                 //删除cdev
    device_destroy(dev1.class, dev1.dev_num);       //删除设备
    class_destroy(dev1.class);                 //删除类
}
module_init(chr_fops_init);
module_exit(chr_fops_exit);
MODULE_LICENSE("GPL v2");
MODULE_AUTHOR("woniu");
```

