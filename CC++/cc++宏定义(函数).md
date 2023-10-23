
```c
int mln_hash_insert(mln_hash_t *h, void *key, void *val)
{
    if (h->expandable && h->nr_nodes > h->threshold) {
        mln_hash_expand(h);
    }
    if (h->expandable && h->nr_nodes <= (h->threshold >> 3)) {
        mln_hash_reduce(h);
    }
    mln_u32_t index = h->hash(h, key);
    mln_hash_mgr_t *mgr = &(h->tbl[index]);
    mln_hash_entry_t *he = mln_hash_entry_new(h, key, val);
    if (he == NULL) return -1;
    mln_hash_entry_chain_add(&(mgr->head), &(mgr->tail), he);
    ++(h->nr_nodes);
    return 0;
}

MLN_CHAIN_FUNC_DEFINE(mln_hash_entry, \
                      mln_hash_entry_t, \
                      static inline void, \
                      prev, \
                      next);


#define MLN_CHAIN_FUNC_DEFINE(prefix,type,ret_attr,prev_ptr,next_ptr); \
    ret_attr prefix##_chain_add(type **head, type **tail, type *node) \
    {\
        if (head == NULL || tail == NULL || node == NULL) return;\
        node->prev_ptr = node->next_ptr = NULL;\
        if (*head == NULL) {\
            *head = *tail = node;\
            return ;\
        }\
        (*tail)->next_ptr = node;\
        node->prev_ptr = (*tail);\
        *tail = node;\
    }\
    \
    ret_attr prefix##_chain_del(type **head, type **tail, type *node) \
    {\
        if (head == NULL || tail == NULL || node == NULL) return;\
        if (*head == node) {\
            if (*tail == node) {\
                *head = *tail = NULL;\
            } else {\
                *head = node->next_ptr;\
                (*head)->prev_ptr = NULL;\
            }\
        } else {\
            if (*tail == node) {\
                *tail = node->prev_ptr;\
                (*tail)->next_ptr = NULL;\
            } else {\
                node->prev_ptr->next_ptr = node->next_ptr;\
                node->next_ptr->prev_ptr = node->prev_ptr;\
            }\
        }\
        node->prev_ptr = node->next_ptr = NULL;\
    }

```

## 宏代码解释

`MLN_CHAIN_FUNC_DEFINE` 这个宏定义的目的是生成通用的链表操作函数的实现代码。

宏定义的参数解释如下：

- `prefix`：函数名的前缀，用于生成函数名。
- `type`：链表节点的类型。
- `ret_attr`：函数返回值的属性，例如 `static`。
- `prev_ptr` 和 `next_ptr`：链表节点结构体中指向前一个节点和后一个节点的指针成员的名称。

宏定义展开后，会生成两个链表操作函数的实现代码：

- `prefix##_chain_add`：用于向链表中添加节点。
- `prefix##_chain_del`：用于从链表中删除节点。

这些生成的函数会根据给定的参数来操作链表，并且可以适用于不同的链表节点类型和链表结构。

这样，通过使用 `MLN_CHAIN_FUNC_DEFINE` 宏定义，可以方便地生成链表操作函数的实现代码，避免了重复编写相似的代码。
> 代码的来源为libmelon库

## 宏定义，函数替换

`mln_hash_entry_chain_add(&(mgr->head), &(mgr->tail), he)` 是对宏定义 `MLN_CHAIN_FUNC_DEFINE` 生成的 `prefix##_chain_add` 函数的调用。

在这个具体的调用中，`prefix` 被替换为 `mln_hash_entry`。假设在宏定义之前有如下定义：

```
typedef struct mln_hash_entry_s {
    struct mln_hash_entry_s *prev_ptr;
    struct mln_hash_entry_s *next_ptr;
    // 其他成员变量...
} mln_hash_entry_t;
```

通过宏定义 `MLN_CHAIN_FUNC_DEFINE(mln_hash_entry, mln_hash_entry_t, , prev_ptr, next_ptr)`，会生成以下代码：

```
void mln_hash_entry_chain_add(mln_hash_entry_t **head, mln_hash_entry_t **tail, mln_hash_entry_t *node)
{
    if (head == NULL || tail == NULL || node == NULL)
        return;
    node->prev_ptr = node->next_ptr = NULL;
    if (*head == NULL) {
        *head = *tail = node;
        return;
    }
    (*tail)->next_ptr = node;
    node->prev_ptr = *tail;
    *tail = node;
}
```

因此，`mln_hash_entry_chain_add(&(mgr->head), &(mgr->tail), he)` 实际上是调用了 `mln_hash_entry_chain_add` 函数，将 `mgr->head` 和 `mgr->tail` 作为指向链表头部和尾部的指针传递，以及 `he` 作为要添加的节点。

这段代码的作用是将新创建的哈希表节点 `he` 添加到哈希表管理器 `mgr` 的链表中。

## 作用

在哈希表的实现中，链表通常用于解决哈希冲突。当多个键被映射到同一个哈希桶时，它们将被存储在同一个桶中的链表中。

具体来说，这段代码执行以下操作：

1. 将新创建的哈希表节点 `he` 的 `prev_ptr` 和 `next_ptr` 成员设置为 `NULL`，以确保节点脱离任何链表。
1. 如果链表头部 `head` 为空，表示链表中没有节点，将 `he` 设为链表的头部和尾部，并结束函数。
1. 如果链表不为空，将新节点 `he` 的指针赋给当前尾部节点的 `next_ptr`，将当前尾部节点的指针赋给 `he` 的 `prev_ptr`，然后将 `he` 设为新的尾部节点。

这样，通过调用 `mln_hash_entry_chain_add` 函数，新节点 `he` 将被添加到哈希表管理器 `mgr` 的链表中。这个操作在哈希表插入时非常常见，确保相同哈希桶中的键值对能够按照插入顺序进行遍历。
