---
layout: post
title: "Block的五种使用场景"
date: 2015-01-05 23:08:02 +0800
comments: true
categories: 技术
---


关于Block（闭包）的介绍就不具体的介绍了，可以自行搜索，Block其实就是一个函数块，这里主要讲Block的使用。主要有以下五种情况：

<!--more-->

### 1. 作为本地变量

```
returnType (^blockName)(parameterTypes) = ^returnType(parameters) {...};
```
### 2. 作为类的属性
```
@property (nonatomic, copy) returnType (^blockName)(parameterTypes);
```
### 3. 作为方法的参数
```
- (void)someMethodThatTakesABlock:(returnType (^)(parameterTypes))blockName;
```
### 4. 调用带Block的方法
```
[someObject someMethodThatTakesABlock:^returnType (parameters) {...}];
```
### 5. typedef一个Block
```
typedef returnType (^TypeName)(parameterTypes);
TypeName blockName = ^returnType(parameters) {...};
```
这里，1、4、5中后面的表达式`^returnType(parameters) {...}`可以省略`returnType`，写成这样:

```
returnType (^blockName)(parameterTypes) = ^(parameters) {...};
[someObject someMethodThatTakesABlock:^(parameters) {...}];
TypeName blockName = ^(parameters) {...};
```

国外有一个网站就用一个域名光介绍Block的这五种使用场景，真够浪费这域名的。[fuckingblocksyntax](http://fuckingblocksyntax.com/)