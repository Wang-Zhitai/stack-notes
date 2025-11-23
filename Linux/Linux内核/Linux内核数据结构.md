# Linux内核数据结构

Linux内核的数据结构有：链表、队列、映射、二叉树。我们只需要掌握API接口即可，内核都有完整的实现，不需要重复的造轮子

## 1. 链表

> Linux内核链表是一个双向链表，每一个链表节点包含了一个next指针一个prev指针

### 1.1 链表结构和初始化

#### 数据结构

```c
struct list_head {
    struct list_head *next, *prev;
};
```

#### 初始化宏

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

### 1.4 判断和检查

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

### 1.7 完整实例

```c
#include <linux/kernel.h>
#include <linux/module.h>
#include <linux/slab.h>
#include <linux/list.h>

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



