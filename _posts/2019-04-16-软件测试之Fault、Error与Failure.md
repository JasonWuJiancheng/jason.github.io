---
layout: post
title: '软件测试（一）'
subtitle: '软件测试之Fault、Error与Failure'
date: 2019-04-16
categories: 技术
cover: ''
tags: 测试 java
---

- **要求：构造程序和测试用例，分别是：**
  1. 不能触发Fault
  2. 触发Fault，但是不能触发Error
  3. 触发Error，但是不能产生Failure

## 1、程序设计

> 采用java语言设计源程序

```java
package com.jason.findnumber;

public class FindLastNumber {
    /*
     * 找出目标值aim在数组最后出现的位置，若在数组中不存在则表示-1
     */
    public static void main(String[] args) {
        int array[] = {2};	//输入数组
        int aim = 2;		//目标值
        int pos = -1;

        if(array.length <= 1){
            if(aim == array[0]){
                pos = 1;
            }else{
                pos = -1;
            }
        }else{
            pos = findLast(aim,array);
        }
        
		//输出
        for(int i = 0; i < array.length; i++){
            System.out.print(array[i] + "  ");
        }
        
        System.out.println();
        System.out.println("find :"+ aim +" The last position:"+pos);
    }
    
    public static int findLast(int aim, int array[]){
        int position = -1;
        for(int i = 1; i < array.length ;i++){	//wrong!
            if(array[i] == aim){
                position = i;
                break;
            }            
        }
        return position + 1;
    }
}
```

## 2、结果代码分析

### 1）不触发Fault

- 测试用例：array = { 2 }；aim：2
- 输出：find 2  The last position:1

数组为单个元素，当数组元素只有一个时，执行以下代码：

```java
 if(array.length <= 1){
            if(aim == array[0]){
                pos = 1;
            }else{
                pos = -1;
            }
        }else{
            pos = findLast(aim,array);
        }
```

直接比较的是数组的值与目标值，并没有调用findLast()方法，所以并==没有触发Fault==。

### 2）触发Fault，但是不能触发Error

- 测试用例：array = { 9，4，3，8，1，6，2 }；aim = 8
- 输出：find 8  The last position:4

调用了findLast()方法，==触发了以下Fault==：

```java
//要求找aim最后出现的位置，应从后向前找，这里逻辑错误找到第一次出现位置
for(int i = 1; i < array.length ;i++){
    if(array[i] == aim){
        position = i;
        break;
    }            
}
```

但是寻找的8正好从前从后数位置一样，所以程序结果仍然正确，==没有触发Error==。

### 3）触发Error，但是不能产生Failure

- 测试用例：array = { 5，3，4，8，5，7，2 }；aim = 5
- 输出：find 5  The last position:5

同样调用了findLast()方法，程序查找的先后顺序出错，找的是第一个出现的值，但是由于程序出错，从第二个找起，==触发了Error==：

```java
//从1开始导致忽略了第一个位置上的5
for(int i = 1; i < array.length ;i++)
```

本来第一出现的5被略过，就找到了位于第5个位置的5，因此最后程序运行的结果是正确的，==没有触发Failure==。