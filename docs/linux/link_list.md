# 链表

## 单链表

#### 单链表数据结构

```c
typedef int elem_type;
typedef struct s_list{
    elem_type data;
	struct s_list *next;
} s_list_node,*s_list_pt;

```

包含数据和节点指针

#### 头插法和尾插法

##### 头插法

```c
int s_list_head_insert(s_list_pt p_head, elem_type data)
{
    if(NULL == p_head) {
	    return -1;
	}
	s_list_pt p_node = s_list_node_create();
	if(NULL == p_node) {
	    return -2;
	}
	p_node->data = data;
    /*头插法*/
	p_node->next = p_head->next;
	p_head->next = p_node;
}
```

与链表的插入一样，先获取插入位置的指针`p_node->next = p_head->next;`，然后将此节点指针赋给上一个节点的`next`指针，`p_head->next = p_node;`

##### 尾插法

```c
int s_list_tail_insert(s_list_pt p_head, elem_type data)
{
	s_list_pt p_index = NULL;
	s_list_pt p_node = NULL;
    if(p_head == NULL) {
	    return -1;
	}
	p_index = p_head;
	while(p_index->next) {
	    p_index = p_index->next;

	}
	/*找到链表的末尾*/
	p_node = s_list_node_create();
	if(p_node == NULL){
	
	   return -1;
	}
	p_node->data = data;
	p_index->next = p_node;
	
    return 0;
}
```

尾插法需要遍历到链表的末端，将节点插入即可



#### 链表逆置

```c
int s_list_reverse(s_list_pt p_head)
{
    s_list_pt ptr = NULL, cur = NULL;
	ptr = p_head->next;
	p_head->next = NULL;
	while (ptr){
	    cur = ptr;
		ptr = ptr->next;
		/*头插法*/
		cur->next = p_head->next;
		p_head->next = cur;
	}
	return 0;
}
```

首先断开头节点和首节点的链接，然后需要两个指针，`ptr`作为指针索引，向后遍历链表，`cur`保存当前指针，每次索引将`cur`指针使用头插法 插入头结点。

> 此方法引申出 链表排序

## 双链表

#### 链表插入

链表插入注意，应该先填充新节点的前驱后继，然后断开原节点，插入新节点

> 参考linux的 list.h 代码

```c
/**
 * list_add - add a new entry
 * @new: new entry to be added
 * @head: list head to add it after
 *
 * Insert a new entry after the specified head.
 * This is good for implementing stacks.
 */
static inline void list_add(struct list_head *new, struct list_head *head)
{
	__list_add(new, head, head->next);
}
static inline void __list_add(struct list_head *new,
			      struct list_head *prev,
			      struct list_head *next)
{
	next->prev = new;    //g:后面节点的前驱设为 new
	new->next = next;    //g:new节点设置前驱后继
	new->prev = prev;
	prev->next = new;    //g:前面节点的后继（必须放在 next->prev = new 之后）
}
```

## 栈

栈的特性为先进后出，实现栈可以用顺序表和链式表。顺序表实现栈需要有标记栈顶的指针。链式表实现栈的思路是使用相同的方法插入取出，比如头插法插入然后取出，便实现了先进后出。

## 队列

### 顺序

```c
//循环队列
typedef int elem_type;
typedef struct queue{
  elem_type queue[QUEUE_MAIX_SIZE];
  unsigned int rear;
  unsigned int front;
}seq_queue_t;

```

1. 入队移动rear，出队移动front

2. rear front 类型为 `unsigned int` 当溢出时回归0，依然满足条件，实际要用的数组索引 `idnex = front % QUEUE_MAIX_SIZE`

3. `rear = front` 为空，  `(rear + 1) % QUEUE_MAIX_SIZE= front % QUEUE_MAIX_SIZE`为满

   

## 链式

```c
typedef int elem_type;
typedef struct node {
    elem_type data;
	struct node *next;
} link_list_t;
typedef struct queue{
   link_list_t *front;
   link_list_t *rear;
}link_queue_t;

```

**NOTE**: 

`front`指向链表的头结点，出队时候可以直接`free`，`rear`指向链表的尾节点，入队时采用尾插法。

```c

int link_queue_input(link_queue_t *p_queue, elem_type data)
{
   if(NULL == p_queue) {
       return -1;
   }
   link_list_t * p_node = (link_list_t*)malloc(sizeof(link_list_t));
   if (p_node == NULL ){
	   return -1;
   }
   p_node->data = data;
   p_node->next = NULL;

   p_queue->rear->next = p_node;
   p_queue->rear = p_node;
   return 0;
}
int link_queue_output(link_queue_t *p_queue, elem_type *data)
{
	link_list_t *p_temp = NULL;
	if(link_queue_is_empty(p_queue)) {
	    return -1;
	}
	p_temp = p_queue->front->next;
	*data = p_temp->data;
	p_queue->front->next = p_temp->next;
	if (p_temp != NULL){
	free(p_temp);
	p_temp = NULL;
	}
	return 0;
}
```

## 树

### 完全二叉树

#### 创建

```c
btree_t* binary_tree_create(int cur, int node_num)
{
    btree_t *p_node = (btree_t*)malloc(sizeof(btree_t));
	if(NULL == p_node) {
	    return NULL;
	}
	p_node->data = cur;
	if (2 * cur <= node_num) {
       p_node->lchild = binary_tree_create(2 * cur, node_num);	   
	} else {
	    p_node->lchild  = NULL;
	}
	if (2 * cur + 1 <= node_num) {
	    p_node->rchild = binary_tree_create(2 * cur + 1, node_num);
	} else {
	    p_node->rchild = NULL;
	}
	return p_node;  
}
```



#### 遍历

1. 前序

   ```c
   void pre_order_traverse(btree_t *p_node)
   {
       if(NULL == p_node) {
   	    return;
   	}
   	printf("%d,",p_node->data);
   
   	pre_order_traverse(p_node->lchild);
   	pre_order_traverse(p_node->rchild);
   
   }
   ```

   

2. 中序

   ```c
   void in_order_traverse(btree_t *p_node)
   {
       if(NULL == p_node) {
   	    return;
   	}
   	pre_order_traverse(p_node->lchild);
   	printf("%d,",p_node->data);
   	pre_order_traverse(p_node->rchild);
   }
   ```

   

3. 后序

   ```c
   void post_order_traverse(btree_t *p_node)
   {
       if(NULL == p_node) {
   	    return;
   	}
       pre_order_traverse(p_node->lchild);
       pre_order_traverse(p_node->rchild);
       printf("%d,",p_node->data);
   
   }
   ```

   

4. 层序

   ```c
   void lever_order_traverse(btree_t *p_node)
   {
       //按照根左右的顺序入队
   	btree_t *p_index = NULL;
       seq_queue_t *p_queue = seq_queue_create();
   	if(NULL != p_node) {
   	    seq_queue_input(p_queue, p_node);
   	}
   	while (!seq_queue_is_empty(p_queue)) {
            seq_queue_output(p_queue, &p_index);
   
   		 if(p_index->lchild != NULL) {
   		     seq_queue_input(p_queue, p_index->lchild);
   		 }
   	     if(p_index->rchild != NULL) {
   		     seq_queue_input(p_queue,p_index->rchild);
   		 } 
   		 printf("%d,",p_index->data);
   	}
   	printf("\n-----------\n");
   }
   ```

   

   二叉树的创建和遍历主要使用了递归的思想。









