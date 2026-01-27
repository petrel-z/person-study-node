https://leetcode.cn/problems/lru-cache-lcci/description/

实现：最近最少使用

使用：双向链表 + map数组

优势：使用链表时，插入和删除操作比较方便，使用map数组来查找数据比较快捷和方便

**get(key)**：当能查到key时，返回该节点的value，并将该节点移到链表头部moveToHead
**put(key，value)**：先get(key)，如果存在，就直接moveToHead
如果不存在，就需要将（key，value）调用addToHead

**首先**要定义一个链表的结构体 key、value、prev、next

**其次**定义一个缓存的对象
为啥要定义它：
1. 用来存放可容纳的数据（对象存储）
2. 还要能记录容量
3. 还能记录头结点和尾结点

**moveToHead（）：**
第一步：删除当前节点
	让前一个节点的next指向要删除节点的下一个节点
	要删除的下一个节点的prev指向删除节点的prev节点
第二步：将该删除的节点添加到头部
	让删除的节点的next指向头节点的next
	让头结点的next节点的prev指向删除的节点
	让删除的节点的prev指向头节点
	让头节点的next指向删除的节点

**addToHead（）：**
检测cache容量是否满了：
满了：
	第一步：删除尾结点
		让尾结点的prev节点的prev指向尾节点
	第二步：添加新节点到头部
		让新节点的next指向头节点的next
		让头结点的next节点的prev指向新节点
		让新节点的prev指向头节点
		让头节点的next指向新节点
没有满：
	添加新节点到头部

```js
let ListNode = function (key, value) {
    this.key = key;
    this.value = value;
    this.prev = null;
    this.next = null;
}
/**
 * @param {number} capacity
 */
 
var LRUCache = function (capacity) {
    this.listNodeMap = {};
    this.capacity = capacity;
    this.head = new ListNode(-1, -1);
    this.tail = new ListNode(-1, -1);
    this.head.next = this.tail;
    this.tail.prev = this.head;
};

/** 
 * @param {number} key
 * @return {number}
 */
LRUCache.prototype.get = function (key) {
    let node = this.listNodeMap[key];
    if (node) {
        this.moveToHead(node);
        return node.value;
    } else {
        return -1;
    }
};

/** 
 * @param {number} key 
 * @param {number} value
 * @return {void}
 */
LRUCache.prototype.put = function (key, value) {
    let node = this.listNodeMap[key];
    if (node) {
        this.listNodeMap[key].value = value;
        let cur = this.listNodeMap[key];
        this.moveToHead(cur);
    } else {
        let cur = new ListNode(key, value);
        this.addToHead(cur);
    }
};

LRUCache.prototype.moveToHead = function (node) {
    // 删除当前节点
    node.next.prev = node.prev;
    node.prev.next = node.next;
    // 插入头节点
    node.next = this.head.next;
    node.next.prev = node;
    this.head.next = node;
    node.prev = this.head;
}

LRUCache.prototype.addToHead = function (node) {
    let len = Object.keys(this.listNodeMap).length;
    if (len < this.capacity) {
        // 有位，插入头部
    } else {
        // 没有位，先删除最后一个节点，再插入头部
        let temp = this.tail.prev;
        this.tail.prev = temp.prev;
        temp.prev.next = this.tail;
        delete this.listNodeMap[temp.key];
    }
    node.next = this.head.next;
    node.next.prev = node;
    this.head.next = node;
    node.prev = this.head;
    this.listNodeMap[node.key] = node;

}
/** 
 * Your LRUCache object will be instantiated and called as such:
 * var obj = new LRUCache(capacity)
 * var param_1 = obj.get(key)
 * obj.put(key,value)
 */
```

