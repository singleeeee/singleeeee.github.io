---
title: 《Vue的设计与实现》
tags: Vue
categories: 底层原理
---

# 《Vue的设计与实现》——霍春阳

## 响应系统

### 作用与实现

``` javascript
const targetMap = new WeakMap();


const obj = new Proxy(data, {
    // 监听读取操作
    get(target, key) {
        track(target, key);
        return targer[key]; 
    };
    set(target, key, newVal) {
    	target[key] = newVal;
		triger(targer, key);

	}
})

function track(target, key) {

}

function triger(target, key) {

}
```



a. 分支切换导致的副作用冗余问题

> cleanup函数

b. 遍历set时导致的无限循环问题

> 创建一个新的Set数据结构

c. 嵌套副作用函数导致响应式关系错乱

> 副作用函数栈





### 计算属性

- 通过`lazy`实现副作用函数延迟执行
- 通过`value `用来缓存上一次计算的值， `dirty` 代表是否需要重新计算



### watch 函数

**watch 的本质其实是对 effect 的二次封装。**

- 通过`scheduler` 调度函数实现监听回调
- 通过`lazy`实现在回调函数中拿到旧值与新值
- 通过`flush`实现`immediate`
- 过期的副作用函数导致的`竞态问题`





## 非原始值的响应式方案

> 这里的非原始值基于一个认识，proxy只能监听对象的简单操作，





## 渲染器

### 事件处理

- 每次更新事件都卸载事件再重新挂载会比较消耗性能
- 通过el对象挂载事件处理对象会出现覆盖问题



## diff算法

### 简单diff算法

背景： 操作Dom的开销比较大，我们希望增量更新

问题1：

- 如果两个节点的子节点完全相同，只有文本不一样时，这时候我们希望只更新文本

问题2：

- 新节点跟旧节点的子节点数量不一定完全相同，我们如何确定要进行挂载还是卸载

**解决：**我们不应该总是遍历旧的一组子节点或者新的一组子节点，而应该遍历较短的一边，然后比较

问题3：

- 如果子节点只是顺序不同，完全不需要进行卸载再挂载的操作，通过移动DOM的性能更好
- 如何确定新的子节点是否出现在旧的一组子节点中？

**解决：**引入`Key`属性，通过比较key属性和节点类型type确定是否有节点变动

### 双端diff算法

- 简单diff算法移动DOM并非最优解

### 快速diff算法

- 预处理阶段
  - 对首尾相同的元素不做移动
- 处理理想情况下的新增和删除节点
- 构造source数组
- 节点位置移动
