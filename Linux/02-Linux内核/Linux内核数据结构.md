# Linux内核数据结构

Linux内核的数据结构有：链表、队列、映射、二叉树。我们只需要掌握API接口即可，内核都有完整的实现，不需要重复的造轮子

## 1. 链表

> Linux内核链表是一个双向链表，每一个链表节点包含了一个next指针一个prev指针，头文件#include <linux/list.h>

### 1.1 链表结构和初始化

#### 数据结构

```c
struct list_head {
    struct list_head *next, *prev;
};
```

#### 初始化

```c
/**
 * LIST_HEAD - 静态定义并初始化链表头，该函数创建的链表头是一个管理者，用于管理整个链表,他完整展开是：
 * @name: 链表头变量名
 * 宏展开：#define LIST_HEAD(name) struct list_head name = LIST_HEAD_INIT(name)
 * 宏展开：#define LIST_HEAD_INIT(name) { &(name), &(name) }
 */
#define LIST_HEAD(name)

/**
 * INIT_LIST_HEAD - 动态初始化链表头，该函数一般用于创建的新的链表节点初始化
 * @list: 要初始化的链表头指针
 */
static inline void INIT_LIST_HEAD(struct list_head *list)
```

### 1.2 链表操作API

#### 添加操作

```c
/**
 * list_add - 在链表头部添加新节点
 * @new: 要添加的新节点
 * @head: 链表头节点
 */
static inline void list_add(struct list_head *new, struct list_head *head)

/**
 * list_add_tail - 在链表尾部添加新节点
 * @new: 要添加的新节点
 * @head: 链表头节点
 */
static inline void list_add_tail(struct list_head *new, struct list_head *head)
```

#### 删除操作

```c
/**
 * list_del - 从链表中删除节点
 * @entry: 要删除的节点
 */
static inline void list_del(struct list_head *entry)

/**
 * list_del_init - 从链表中删除节点并重新初始化
 * @entry: 要删除的节点
 */
static inline void list_del_init(struct list_head *entry)
```

#### 移动操作

```c
/**
 * list_move - 删除节点并将其添加到另一个链表的头部
 * @list: 要移动的链表节点
 * @head: 目标链表的头节点
 */
static inline void list_move(struct list_head *list, struct list_head *head)

/**
 * list_move_tail - 删除节点并将其添加到另一个链表的尾部
 * @list: 要移动的链表节点
 * @head: 目标链表的头节点
 */
static inline void list_move_tail(struct list_head *list, struct list_head *head)
```

#### 替换操作

```c
/**
 * list_replace - 用新节点替换旧节点
 * @old: 要被替换的旧节点
 * @new: 用于替换的新节点
 */
static inline void list_replace(struct list_head *old, struct list_head *new)

/**
 * list_replace_init - 用新节点替换旧节点并重新初始化旧节点
 * @old: 要被替换的旧节点
 * @new: 用于替换的新节点
 */
static inline void list_replace_init(struct list_head *old, struct list_head *new)
```

### 1.3 遍历操作

#### 基本遍历

```c
/**
 * list_for_each - 正向遍历链表
 * @pos: 当前节点指针（struct list_head *）
 * @head: 链表头节点
 */
#define list_for_each(pos, head)

/**
 * list_for_each_prev - 反向遍历链表
 * @pos: 当前节点指针（struct list_head *）
 * @head: 链表头节点
 */
#define list_for_each_prev(pos, head)
```

#### 安全遍历（允许在遍历时删除）

```c
/**
 * list_for_each_safe - 安全正向遍历链表（允许删除当前节点）
 * @pos: 当前节点指针（struct list_head *）
 * @n: 下一个节点指针（struct list_head *），用于临时存储
 * @head: 链表头节点
 */
#define list_for_each_safe(pos, n, head)

/**
 * list_for_each_prev_safe - 安全反向遍历链表（允许删除当前节点）
 * @pos: 当前节点指针（struct list_head *）
 * @n: 前一个节点指针（struct list_head *），用于临时存储
 * @head: 链表头节点
 */
#define list_for_each_prev_safe(pos, n, head)
```

#### 遍历包含链表的结构体

```c
/**
 * list_for_each_entry - 正向遍历包含链表的结构体
 * @pos: 当前结构体指针
 * @head: 链表头节点
 * @member: 链表在结构体中的成员名
 */
#define list_for_each_entry(pos, head, member)

/**
 * list_for_each_entry_reverse - 反向遍历包含链表的结构体
 * @pos: 当前结构体指针
 * @head: 链表头节点
 * @member: 链表在结构体中的成员名
 */
#define list_for_each_entry_reverse(pos, head, member)

/**
 * list_for_each_entry_safe - 安全正向遍历包含链表的结构体（允许删除当前节点）
 * @pos: 当前结构体指针
 * @n: 下一个结构体指针，用于临时存储
 * @head: 链表头节点
 * @member: 链表在结构体中的成员名
 */
#define list_for_each_entry_safe(pos, n, head, member)

/**
 * list_for_each_entry_safe_reverse - 安全反向遍历包含链表的结构体（允许删除当前节点）
 * @pos: 当前结构体指针
 * @n: 前一个结构体指针，用于临时存储
 * @head: 链表头节点
 * @member: 链表在结构体中的成员名
 */
#define list_for_each_entry_safe_reverse(pos, n, head, member)

/**
 * list_for_each_entry_continue - 从当前点继续正向遍历包含链表的结构体
 * @pos: 当前结构体指针
 * @head: 链表头节点
 * @member: 链表在结构体中的成员名
 */
#define list_for_each_entry_continue(pos, head, member)

/**
 * list_for_each_entry_continue_reverse - 从当前点继续反向遍历包含链表的结构体
 * @pos: 当前结构体指针
 * @head: 链表头节点
 * @member: 链表在结构体中的成员名
 */
#define list_for_each_entry_continue_reverse(pos, head, member)

/**
 * list_for_each_entry_from - 从指定点开始正向遍历包含链表的结构体
 * @pos: 当前结构体指针
 * @head: 链表头节点
 * @member: 链表在结构体中的成员名
 */
#define list_for_each_entry_from(pos, head, member)

/**
 * list_for_each_entry_from_reverse - 从指定点开始反向遍历包含链表的结构体
 * @pos: 当前结构体指针
 * @head: 链表头节点
 * @member: 链表在结构体中的成员名
 */
#define list_for_each_entry_from_reverse(pos, head, member)
```

### 1.4 状态查询

```c
/**
 * list_empty - 判断链表是否为空
 * @head: 要检查的链表头
 * 
 * 返回: 链表为空返回真，否则返回假
 */
static inline int list_empty(const struct list_head *head)

/**
 * list_empty_careful - 小心地判断链表是否为空（用于并发环境）
 * @head: 要检查的链表头
 * 
 * 返回: 链表为空返回真，否则返回假
 */
static inline int list_empty_careful(const struct list_head *head)

/**
 * list_is_singular - 判断链表是否只有一个节点
 * @head: 要检查的链表头
 * 
 * 返回: 链表只有一个节点返回真，否则返回假
 */
static inline int list_is_singular(const struct list_head *head)

/**
 * list_is_last - 判断节点是否为链表的最后一个节点
 * @list: 要检查的节点
 * @head: 链表头节点
 * 
 * 返回: 如果是最后一个节点返回真，否则返回假
 */
static inline int list_is_last(const struct list_head *list, const struct list_head *head)
```

### 1.5 切割和合并

```c
/**
 * list_cut_position - 将链表从指定位置切割成两部分
 * @list: 用于存放切割后部分的新链表头
 * @head: 原始链表头
 * @entry: 切割位置节点（新链表将包含此节点及之后的所有节点）
 */
static inline void list_cut_position(struct list_head *list, struct list_head *head, struct list_head *entry)

/**
 * list_splice - 合并两个链表（将list链表合并到head链表的头部）
 * @list: 要合并的源链表
 * @head: 目标链表的头节点
 */
static inline void list_splice(const struct list_head *list, struct list_head *head)

/**
 * list_splice_tail - 合并两个链表（将list链表合并到head链表的尾部）
 * @list: 要合并的源链表
 * @head: 目标链表的头节点
 */
static inline void list_splice_tail(struct list_head *list, struct list_head *head)

/**
 * list_splice_init - 合并两个链表并重新初始化源链表
 * @list: 要合并的源链表
 * @head: 目标链表的头节点
 */
static inline void list_splice_init(struct list_head *list, struct list_head *head)

/**
 * list_splice_tail_init - 合并两个链表到尾部并重新初始化源链表
 * @list: 要合并的源链表
 * @head: 目标链表的头节点
 */
static inline void list_splice_tail_init(struct list_head *list, struct list_head *head)
```

### 1.6 链表节点获取宏

```c
/**
 * list_entry - 获取包含链表节点的结构体指针
 * @ptr: 链表节点指针
 * @type: 结构体类型
 * @member: 链表在结构体中的成员名
 * 
 * 返回: 包含链表节点的结构体指针
 */
#define list_entry(ptr, type, member)

/**
 * list_first_entry - 获取链表的第一个结构体
 * @ptr: 链表头指针
 * @type: 结构体类型
 * @member: 链表在结构体中的成员名
 * 
 * 返回: 链表的第一个结构体指针
 */
#define list_first_entry(ptr, type, member)

/**
 * list_last_entry - 获取链表的最后一个结构体
 * @ptr: 链表头指针
 * @type: 结构体类型
 * @member: 链表在结构体中的成员名
 * 
 * 返回: 链表的最后一个结构体指针
 */
#define list_last_entry(ptr, type, member)

/**
 * list_first_entry_or_null - 获取链表的第一个结构体，如果链表为空返回NULL
 * @ptr: 链表头指针
 * @type: 结构体类型
 * @member: 链表在结构体中的成员名
 * 
 * 返回: 第一个结构体指针或NULL
 */
#define list_first_entry_or_null(ptr, type, member)

/**
 * list_next_entry - 获取下一个结构体
 * @pos: 当前结构体指针
 * @member: 链表在结构体中的成员名
 * 
 * 返回: 下一个结构体指针
 */
#define list_next_entry(pos, member)

/**
 * list_prev_entry - 获取前一个结构体
 * @pos: 当前结构体指针
 * @member: 链表在结构体中的成员名
 * 
 * 返回: 前一个结构体指针
 */
#define list_prev_entry(pos, member)
```

### 1.7 完整示例

```c
#include <linux/kernel.h>	//包含 printk() 等内核函数
#include <linux/module.h>	//包含模块相关的函数和宏
#include <linux/slab.h>	//包含动态内存管理函数
#include <linux/list.h>	//包含双向链表操作的函数和结构

// 1. 专门的头节点（链表管理者）
static LIST_HEAD(device_list);

// 2. 纯粹的数据结构
struct device {
    int id;
    char name[32];
    int status;
    struct list_head list;  // 链表钩子
};

// 3. 清晰的操作函数
struct device *create_device(int id, const char *name)
{
    struct device *dev = kmalloc(sizeof(*dev), GFP_KERNEL);
    if (!dev) return NULL;
    
    // 初始化业务数据
    dev->id = id;
    strlcpy(dev->name, name, sizeof(dev->name));
    dev->status = 0;
    
    // 初始化链表钩子
    INIT_LIST_HEAD(&dev->list);
    
    // 将节点插入到链表尾部
    list_add_tail(&dev->list, &device_list);
    
    return dev;
}

void print_all_devices(void)
{
    struct device *dev;
    
    printk(KERN_INFO "设备列表:\n");
    
    // 清晰：从头节点开始遍历所有数据节点
    list_for_each_entry(dev, &device_list, list) {
        printk(KERN_INFO "  ID=%d, 名称=%s, 状态=%d\n", 
               dev->id, dev->name, dev->status);
    }
}

// 4. 安全的清理函数
void cleanup_all_devices(void)
{
    struct device *dev, *tmp;
    
    // 安全遍历并删除所有数据节点
    list_for_each_entry_safe(dev, tmp, &device_list, list) {
        list_del(&dev->list);
        kfree(dev);
    }
    
    // 当删除所有数据节点后，头节点 device_list 自动恢复为空链表状态
}
```

## 2. 队列

> Linux内核的队列实现是kfifo，kfifo 是一个无锁环形缓冲区，专门为单生产者/消费者模型设计，头文件#include <linux/kfifo.h>

### 2.1 kfifo结构和初始化

#### 数据结构

```c
struct __kfifo {
	unsigned int	in; //tail pointer
	unsigned int	out; //head pointer
	unsigned int	mask; //queue_size-1, (power of 2)-1
	unsigned int	esize; //element size
	void		*data; 
};
```

#### 声明和定义宏

```c
/**
 * DECLARE_KFIFO - 声明一个静态FIFO对象（不初始化）
 * @fifo: FIFO变量名称
 * @type: FIFO中元素的类型
 * @size: FIFO的容量（必须是2的幂次方）
 * 
 * 注意：此宏只声明FIFO结构，需要使用INIT_KFIFO进行初始化
 */
#define DECLARE_KFIFO(fifo, type, size)

/**
 * INIT_KFIFO - 初始化已声明的静态FIFO
 * @fifo: 要初始化的FIFO对象指针
 * 
 * 注意：仅用于DECLARE_KFIFO声明的FIFO，将FIFO重置为空状态
 */
#define INIT_KFIFO(fifo)

/**
 * DEFINE_KFIFO - 定义并初始化一个静态FIFO对象
 * @fifo: FIFO变量名称  
 * @type: FIFO中元素的类型
 * @size: FIFO的容量（必须是2的幂次方）
 * 
 * 注意：此宏会完成FIFO的声明和初始化，定义后立即可用
 */
#define DEFINE_KFIFO(fifo, type, size)

/**
 * DECLARE_KFIFO_PTR - 声明一个动态FIFO指针
 * @fifo: FIFO指针变量名称
 * @type: FIFO中元素的类型
 * 
 * 注意：此宏只声明FIFO指针，需要使用kfifo_alloc分配内存
 */
#define DECLARE_KFIFO_PTR(fifo, type)
```

#### 内存管理

```c
/**
 * kfifo_alloc - 动态分配并初始化FIFO
 * @fifo: FIFO指针的地址（&fifo_ptr）
 * @size: 请求的FIFO容量（元素个数，会自动向上取整为2的幂次方）
 * @gfp_mask: 内存分配标志
 * 
 * 返回值: 成功返回0，失败返回错误码
 * 
 * 注意：使用前需要用DECLARE_KFIFO_PTR声明FIFO指针，需要使用kfifo_free释放内存
 */
#define kfifo_alloc(fifo, size, gfp_mask)

/**
 * kfifo_init - 使用预分配缓冲区初始化FIFO
 * @fifo: FIFO指针的地址（&fifo_ptr）
 * @buffer: 预分配的内存缓冲区指针
 * @size: 缓冲区总大小（字节）
 * 
 * 返回值: 成功返回0，失败返回错误码
 * 
 * 注意：缓冲区大小必须足够容纳FIFO元素，不会自动释放缓冲区内存
 */
#define kfifo_init(fifo, buffer, size)

/**
 * kfifo_free - 释放动态分配的FIFO
 * @fifo: FIFO指针的地址（&fifo_ptr）
 * 
 * 注意：仅释放由kfifo_alloc分配的FIFO，对静态FIFO或kfifo_init初始化的FIFO无效
 */
#define kfifo_free(fifo)
```

### 2.2 状态查询

```c
/**
 * kfifo_initialized - 检查FIFO是否已初始化
 * @fifo: FIFO对象指针
 * 
 * 返回值: 已初始化返回true，未初始化返回false
 */
#define kfifo_initialized(fifo)

/**
 * kfifo_size - 获取FIFO总容量
 * @fifo: FIFO对象指针
 * 
 * 返回值: FIFO的最大容量（元素个数）
 */
#define kfifo_size(fifo)

/**
 * kfifo_len - 获取FIFO中已使用元素数量
 * @fifo: FIFO对象指针
 * 
 * 返回值: FIFO中当前元素数量
 */
#define kfifo_len(fifo)

/**
 * kfifo_avail - 返回FIFO中可用（未使用）元素数量
 * @fifo: FIFO对象指针
 * 
 * 返回值: FIFO中剩余可用空间（元素个数）
 */
#define kfifo_avail(fifo)

/**
 * kfifo_is_empty - 检查FIFO是否为空
 * @fifo: FIFO对象指针
 * 
 * 返回值: 为空返回true，否则返回false
 */
#define kfifo_is_empty(fifo)

/**
 * kfifo_is_full - 检查FIFO是否已满  
 * @fifo: FIFO对象指针
 * 
 * 返回值: 已满返回true，否则返回false
 */
#define kfifo_is_full(fifo)

/**
 * kfifo_esize - 返回FIFO中单个元素的大小
 * @fifo: FIFO对象指针
 * 
 * 返回值: 元素大小（字节）
 */
#define kfifo_esize(fifo)

/**
 * kfifo_recsize - 返回记录长度字段的大小
 * @fifo: FIFO对象指针
 * 
 * 返回值: 记录长度字段大小（字节）
 */
#define kfifo_recsize(fifo)

/**
 * kfifo_peek_len - 获取下一条记录的长度
 * @fifo: FIFO对象指针
 * 
 * 返回值: 记录长度
 */
#define kfifo_peek_len(fifo)
```

### 2.3 数据操作

#### 单元素操作

```c
/**
 * kfifo_put - 向FIFO放入单个元素
 * @fifo: FIFO对象指针
 * @val: 要放入的元素值
 * 
 * 返回值: 成功返回1，FIFO已满返回0
 */
#define kfifo_put(fifo, val)

/**
 * kfifo_get - 从FIFO取出单个元素
 * @fifo: FIFO对象指针  
 * @val: 存储取出元素的变量地址
 * 
 * 返回值: 成功返回1，FIFO为空返回0
 */
#define kfifo_get(fifo, val)

/**
 * kfifo_peek - 查看FIFO中的下一个元素（不取出）
 * @fifo: FIFO对象指针
 * @val: 存储查看结果的变量地址
 * 
 * 返回值: 成功查看返回1，FIFO为空返回0
 */
#define kfifo_peek(fifo, val)
```

#### 批量操作

```c
/**
 * kfifo_in - 向FIFO批量放入元素
 * @fifo: FIFO对象指针
 * @buf: 源数据缓冲区指针
 * @n: 要放入的元素数量
 * 
 * 返回值: 实际放入的元素数量
 */
#define kfifo_in(fifo, buf, n)

/**
 * kfifo_out - 从FIFO批量取出元素  
 * @fifo: FIFO对象指针
 * @buf: 目标数据缓冲区指针
 * @n: 要取出的最大元素数量
 * 
 * 返回值: 实际取出的元素数量
 */
#define kfifo_out(fifo, buf, n)

/**
 * kfifo_out_peek - 批量查看FIFO中的多个元素（不取出）
 * @fifo: FIFO对象指针
 * @buf: 存储查看结果的缓冲区指针
 * @n: 要查看的最大元素数量
 * 
 * 返回值: 实际查看的元素数量
 */
#define kfifo_out_peek(fifo, buf, n)
```

### 2.4 重置和清理

```c
/**
 * kfifo_reset - 重置FIFO（清空所有内容）
 * @fifo: FIFO对象指针
 * 
 * 注意：需要确保没有其他线程在访问FIFO，将读写指针都重置为0
 */
#define kfifo_reset(fifo)
```

### 2.5 线程安全版本读写操作

```c
/**
 * kfifo_in_spinlocked - 使用自旋锁保护的批量放入操作
 * @fifo: FIFO对象指针
 * @buf: 源数据缓冲区指针
 * @n: 要放入的元素数量
 * @lock: 自旋锁指针
 * 
 * 返回值: 实际放入的元素数量
 * 
 * 注意：在操作期间会获取自旋锁，确保线程安全，适合在中断上下文或原子上下文中使用
 */
#define kfifo_in_spinlocked(fifo, buf, n, lock)

/**
 * kfifo_out_spinlocked - 使用自旋锁保护的批量取出操作
 * @fifo: FIFO对象指针
 * @buf: 目标数据缓冲区指针
 * @n: 要取出的最大元素数量
 * @lock: 自旋锁指针
 * 
 * 返回值: 实际取出的元素数量
 * 
 * 注意：在操作期间会获取自旋锁，确保线程安全，适合在中断上下文或原子上下文中使用
 */
#define kfifo_out_spinlocked(fifo, buf, n, lock)
```

### 2.6 用户空间读写操作

```c
/**
 * kfifo_from_user - 从用户空间复制数据到内核FIFO
 * @fifo: FIFO对象指针
 * @from: 用户空间源地址指针
 * @len: 要复制的数据长度（字节）
 * @copied: 输出参数，返回实际复制的字节数
 * 
 * 返回值: 成功返回0，失败返回错误码
 * 
 * 注意：自动处理用户空间地址验证和拷贝
 *       考虑FIFO可用空间，可能只复制部分数据
 *       适合系统调用实现中使用
 */
#define kfifo_from_user(fifo, from, len, copied)

/**
 * kfifo_to_user - 从内核FIFO复制数据到用户空间
 * @fifo: FIFO对象指针
 * @to: 用户空间目标地址指针
 * @len: 要复制的数据长度（字节）
 * @copied: 输出参数，返回实际复制的字节数
 * 
 * 返回值: 成功返回0，失败返回错误码
 * 
 * 注意：自动处理用户空间地址验证和拷贝
 *       考虑FIFO中实际数据量，可能只复制部分数据
 *       适合系统调用实现中使用
 */
#define kfifo_to_user(fifo, to, len, copied)
```

### 2.7 动态声明完整示例

```c
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/kfifo.h>
#include <linux/spinlock.h>
#include <linux/slab.h>
#include <linux/uaccess.h>

// 定义设备数据结构
struct kfifo_device {
    // 静态FIFO - 用于命令队列
    DEFINE_KFIFO(cmd_fifo, u32, 32);
    
    // 动态FIFO - 用于数据缓冲区
    DECLARE_KFIFO_PTR(data_fifo, u8);
    
    // 保护锁
    spinlock_t lock;
    
    // 设备状态
    bool initialized;
};

static struct kfifo_device *kfifo_dev;

// 初始化设备
static int kfifo_device_init(void)
{
    int ret;
    
    printk(KERN_INFO "Initializing kfifo device\n");
    
    // 分配设备结构
    kfifo_dev = kzalloc(sizeof(*kfifo_dev), GFP_KERNEL);
    if (!kfifo_dev) {
        printk(KERN_ERR "Failed to allocate kfifo device\n");
        return -ENOMEM;
    }
    
    // 静态FIFO已经通过DEFINE_KFIFO自动初始化
    
    // 初始化动态FIFO
    ret = kfifo_alloc(&kfifo_dev->data_fifo, 4096, GFP_KERNEL);
    if (ret) {
        printk(KERN_ERR "Failed to allocate data FIFO: %d\n", ret);
        kfree(kfifo_dev);
        return ret;
    }
    
    // 初始化自旋锁
    spin_lock_init(&kfifo_dev->lock);
    kfifo_dev->initialized = true;
    
    printk(KERN_INFO "kfifo device initialized\n");
    printk(KERN_INFO "Command FIFO: size=%u, esize=%zu\n", 
           kfifo_size(&kfifo_dev->cmd_fifo), 
           kfifo_esize(&kfifo_dev->cmd_fifo));
    printk(KERN_INFO "Data FIFO: size=%u, esize=%zu\n", 
           kfifo_size(&kfifo_dev->data_fifo), 
           kfifo_esize(&kfifo_dev->data_fifo));
    
    return 0;
}

// 清理设备
static void kfifo_device_cleanup(void)
{
    if (!kfifo_dev || !kfifo_dev->initialized)
        return;
    
    printk(KERN_INFO "Cleaning up kfifo device\n");
    
    // 释放动态FIFO
    kfifo_free(&kfifo_dev->data_fifo);
    
    // 释放设备结构
    kfree(kfifo_dev);
    kfifo_dev = NULL;
    
    printk(KERN_INFO "kfifo device cleaned up\n");
}

// 命令处理函数
static void process_commands(void)
{
    u32 command;
    unsigned int processed = 0;
    
    // 处理所有命令（非线程安全，假设单消费者）
    while (kfifo_get(&kfifo_dev->cmd_fifo, &command)) {
        printk(KERN_INFO "Processing command: 0x%08x\n", command);
        processed++;
        
        // 模拟命令处理
        switch (command & 0xFF) {
            case 0x01:
                printk(KERN_DEBUG "Command: START\n");
                break;
            case 0x02:
                printk(KERN_DEBUG "Command: STOP\n");
                break;
            case 0x03:
                printk(KERN_DEBUG "Command: RESET\n");
                break;
            default:
                printk(KERN_DEBUG "Command: UNKNOWN\n");
        }
    }
    
    if (processed > 0) {
        printk(KERN_INFO "Processed %u commands\n", processed);
    }
}

// 数据生产者（多线程安全）
static int produce_data(const u8 *data, size_t len)
{
    unsigned long flags;
    unsigned int written;
    
    if (!kfifo_dev || !kfifo_dev->initialized) {
        printk(KERN_ERR "Device not initialized\n");
        return -ENODEV;
    }
    
    // 检查FIFO状态
    if (kfifo_is_full(&kfifo_dev->data_fifo)) {
        printk(KERN_WARNING "Data FIFO is full\n");
        return -ENOSPC;
    }
    
    // 线程安全的数据写入
    spin_lock_irqsave(&kfifo_dev->lock, flags);
    written = kfifo_in_spinlocked(&kfifo_dev->data_fifo, data, len, &kfifo_dev->lock);
    spin_unlock_irqrestore(&kfifo_dev->lock, flags);
    
    printk(KERN_DEBUG "Produced %u bytes, FIFO usage: %u/%u\n",
           written, kfifo_len(&kfifo_dev->data_fifo), 
           kfifo_size(&kfifo_dev->data_fifo));
    
    return written;
}

// 数据消费者（多线程安全）
static int consume_data(u8 *buffer, size_t max_len)
{
    unsigned long flags;
    unsigned int read;
    
    if (!kfifo_dev || !kfifo_dev->initialized) {
        printk(KERN_ERR "Device not initialized\n");
        return -ENODEV;
    }
    
    // 检查FIFO状态
    if (kfifo_is_empty(&kfifo_dev->data_fifo)) {
        return -ENODATA;
    }
    
    // 线程安全的数据读取
    spin_lock_irqsave(&kfifo_dev->lock, flags);
    read = kfifo_out_spinlocked(&kfifo_dev->data_fifo, buffer, max_len, &kfifo_dev->lock);
    spin_unlock_irqrestore(&kfifo_dev->lock, flags);
    
    printk(KERN_DEBUG "Consumed %u bytes, FIFO remaining: %u/%u\n",
           read, kfifo_len(&kfifo_dev->data_fifo), 
           kfifo_size(&kfifo_dev->data_fifo));
    
    return read;
}

// 预览数据（不消费）
static int peek_data(u8 *buffer, size_t max_len)
{
    unsigned long flags;
    unsigned int peeked;
    
    if (!kfifo_dev || !kfifo_dev->initialized) {
        return -ENODEV;
    }
    
    spin_lock_irqsave(&kfifo_dev->lock, flags);
    peeked = kfifo_out_peek(&kfifo_dev->data_fifo, buffer, max_len);
    spin_unlock_irqrestore(&kfifo_dev->lock, flags);
    
    if (peeked > 0) {
        printk(KERN_DEBUG "Peeked %u bytes (not consumed)\n", peeked);
    }
    
    return peeked;
}

// 模拟用户空间数据写入
static int simulate_user_write(const char __user *user_data, size_t len)
{
    unsigned int copied = 0;
    int ret;
    
    if (!kfifo_dev || !kfifo_dev->initialized) {
        return -ENODEV;
    }
    
    printk(KERN_INFO "Simulating user space write: %zu bytes\n", len);
    
    // 使用用户空间API（在实际驱动中会真正处理用户指针）
    ret = kfifo_from_user(&kfifo_dev->data_fifo, user_data, len, &copied);
    
    if (ret) {
        printk(KERN_ERR "Failed to copy from user: %d\n", ret);
        return ret;
    }
    
    printk(KERN_INFO "Successfully copied %u bytes from user\n", copied);
    return copied;
}

// 设备状态监控
static void monitor_device_status(void)
{
    if (!kfifo_dev || !kfifo_dev->initialized) {
        printk(KERN_ERR "Device not initialized\n");
        return;
    }
    
    printk(KERN_INFO "=== Device Status ===\n");
    printk(KERN_INFO "Initialized: %s\n", kfifo_dev->initialized ? "yes" : "no");
    
    // 命令FIFO状态
    printk(KERN_INFO "Command FIFO - Size: %u, Used: %u, Available: %u, Empty: %s, Full: %s\n",
           kfifo_size(&kfifo_dev->cmd_fifo),
           kfifo_len(&kfifo_dev->cmd_fifo),
           kfifo_avail(&kfifo_dev->cmd_fifo),
           kfifo_is_empty(&kfifo_dev->cmd_fifo) ? "yes" : "no",
           kfifo_is_full(&kfifo_dev->cmd_fifo) ? "yes" : "no");
    
    // 数据FIFO状态
    printk(KERN_INFO "Data FIFO - Size: %u, Used: %u, Available: %u, Empty: %s, Full: %s\n",
           kfifo_size(&kfifo_dev->data_fifo),
           kfifo_len(&kfifo_dev->data_fifo),
           kfifo_avail(&kfifo_dev->data_fifo),
           kfifo_is_empty(&kfifo_dev->data_fifo) ? "yes" : "no",
           kfifo_is_full(&kfifo_dev->data_fifo) ? "yes" : "no");
}

// 测试函数
static void run_kfifo_tests(void)
{
    u32 test_commands[] = {0x00000001, 0x00000002, 0x00000003, 0x00000004};
    u8 test_data[64];
    u8 buffer[128];
    int i, ret;
    
    printk(KERN_INFO "=== Starting kfifo tests ===\n");
    
    // 测试1: 命令FIFO操作
    printk(KERN_INFO "Test 1: Command FIFO operations\n");
    for (i = 0; i < ARRAY_SIZE(test_commands); i++) {
        if (kfifo_put(&kfifo_dev->cmd_fifo, test_commands[i])) {
            printk(KERN_DEBUG "Command %d queued successfully\n", i);
        } else {
            printk(KERN_WARNING "Failed to queue command %d (FIFO full)\n", i);
        }
    }
    
    // 处理命令
    process_commands();
    
    // 测试2: 数据FIFO操作
    printk(KERN_INFO "Test 2: Data FIFO operations\n");
    
    // 准备测试数据
    for (i = 0; i < sizeof(test_data); i++) {
        test_data[i] = i & 0xFF;
    }
    
    // 生产数据
    ret = produce_data(test_data, sizeof(test_data));
    if (ret > 0) {
        printk(KERN_INFO "Produced %d bytes of test data\n", ret);
    }
    
    // 预览数据（不消费）
    ret = peek_data(buffer, 16);
    if (ret > 0) {
        printk(KERN_INFO "Peeked first %d bytes: ", ret);
        for (i = 0; i < ret && i < 8; i++) {
            printk(KERN_CONT "%02x ", buffer[i]);
        }
        printk(KERN_CONT "...\n");
    }
    
    // 消费数据
    ret = consume_data(buffer, sizeof(buffer));
    if (ret > 0) {
        printk(KERN_INFO "Consumed %d bytes of data\n", ret);
    }
    
    // 测试3: FIFO重置
    printk(KERN_INFO "Test 3: FIFO reset\n");
    
    // 先放一些数据
    produce_data(test_data, 16);
    printk(KERN_INFO "Before reset - Data FIFO used: %u\n", 
           kfifo_len(&kfifo_dev->data_fifo));
    
    // 重置FIFO
    kfifo_reset(&kfifo_dev->data_fifo);
    printk(KERN_INFO "After reset - Data FIFO used: %u\n", 
           kfifo_len(&kfifo_dev->data_fifo));
    
    // 显示最终状态
    monitor_device_status();
    
    printk(KERN_INFO "=== kfifo tests completed ===\n");
}

// 模块初始化
static int __init kfifo_example_init(void)
{
    int ret;
    
    printk(KERN_INFO "kfifo example module loading\n");
    
    ret = kfifo_device_init();
    if (ret) {
        return ret;
    }
    
    // 运行测试
    run_kfifo_tests();
    
    printk(KERN_INFO "kfifo example module loaded successfully\n");
    return 0;
}

// 模块退出
static void __exit kfifo_example_exit(void)
{
    printk(KERN_INFO "kfifo example module unloading\n");
    
    kfifo_device_cleanup();
    
    printk(KERN_INFO "kfifo example module unloaded\n");
}

module_init(kfifo_example_init);
module_exit(kfifo_example_exit);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Kernel Developer");
MODULE_DESCRIPTION("Example kfifo usage in Linux kernel");
MODULE_VERSION("1.0");
```

### 2.8 静态声明完整示例

```c
#include <linux/kfifo.h>

// 声明一个完整的管理32个int的FIFO
DEFINE_KFIFO(number_fifo, int, 32);

void use_static_fifo(void)
{
    int i, value;
    
    printk(KERN_INFO "静态FIFO初始状态:\n");
    printk(KERN_INFO "大小: %u, 已用: %u, 可用: %u\n",
           kfifo_size(&number_fifo),
           kfifo_len(&number_fifo), 
           kfifo_avail(&number_fifo));
    
    // 填充数据
    for (i = 0; i < 35; i++) {  // 尝试超过容量
        if (kfifo_put(&number_fifo, i * 10)) {
            printk(KERN_DEBUG "成功放入: %d\n", i * 10);
        } else {
            printk(KERN_DEBUG "FIFO已满，无法放入: %d\n", i * 10);
        }
    }
    
    printk(KERN_INFO "填充后状态:\n");
    printk(KERN_INFO "大小: %u, 已用: %u, 可用: %u\n",
           kfifo_size(&number_fifo),
           kfifo_len(&number_fifo),
           kfifo_avail(&number_fifo));
    
    // 取出数据
    while (kfifo_get(&number_fifo, &value)) {
        printk(KERN_INFO "取出: %d\n", value);
    }
}
```

