[遥感原理与应用-GEE1.pdf](https://dengbaiyu2023.yuque.com/attachments/yuque/0/2024/pdf/2371504/1731491054102-d318c859-0588-4519-8122-cbdb35d5b24e.pdf)

[Google Earth Engine](https://earthengine.google.com/)

教程1：[https://developers.google.com/earth-engine](https://developers.google.com/earth-engine)

教程2：[https://space.bilibili.com/527404442/video](https://space.bilibili.com/527404442/video)

### Google Earth Engine 介绍
+ 免费、云上数据与算力、易学易用
+ 工作原理：金字塔的数据组织类型
+ 编程语言：Javascript和Python
+ GEE数据：遥感影像、地理变量、气候变量、人工变量

### JavaScript基础
[现代 JavaScript 教程](https://zh.javascript.info/)

#### 简介
+ 顺序执行
+ 大小写敏感：变量的大小写不同，不同的变量
+ 命名规则：使用驼峰式，尽量不要用下划线方法
+ 多于一个空格时，只认为有一个空格，字符串String类型中以外
+ 语句
    - 定义变量可以以分号结束或者以换行结束都行

#### 赋值语句
```javascript
var myVariable = 1 // 声明变量并赋值
myVariable = 2 // 变量赋值
var otherVariable // 仅仅声明变量但不赋值
otherVariable = 2 // 变量赋值
otherVariable = "Hello world!" // 变量数据类型是动态
```

[老旧的 “var”](https://zh.javascript.info/var)

#### 数据类型
1: Boolean  2: String  3: Number  4: Array  5: Object

#### 数据类型：字符串 String
```javascript
var aString = "This is a astring."
console.log(aString.length) // 输出字符串aString的长度
```

#### 数据类型：数组 Array
```javascript
var emptyArray=[]
var myArray = [1,4,9]var myArray2 = ["phrases are okay", false, 3.14159, 6, "x**2"] 
//Can have different variable types
var myArray3 = new Array() // {undefined length, empty}
var mayArray4 = new Array(5) // {5-item array, empty}.
console.log(myArray2[2])
```

#### 数据类型：对象 Object
+ 类是具有相似内部状态和运动规律的实体的集合（或统称、抽象）。对象是某⼀种类的实例

```javascript
var myObject = {
  propertyName1: value,
  propertyName2: value, 
  ...
};
```

```javascript
ar jedi = {
 name: "Yoda",
 age: 899,
 isCool: true
};
console.log(jedi.name+jedi.age) // Prints: Yoda899
console.log(jedi.isCool) // Prints: true
```



#### 数学运算
```javascript
Math.random(); // returns a random number
Math.min(0, 150, 30, 20, -8); // returns -8
Math.round(4.7); // returns 5
Math.round(4.4); // returns 4
Math.ceil(4.4); // returns 5
Math.floor(4.4); // returns 4
Math.E; // returns Euler's number
Math.PI // returns PI
Math.SQRT2 // returns the square root of 2
Math.SQRT1_2 // returns the square root of 1/2
Math.LN2 // returns the natural logarithm of 2
Math.LN10 // returns the natural logarithm of 10
Math.LOG2E // returns base 2 logarithm of E
Math.LOG10E // returns base 10 logarithm of E
//Math.sin() Math.cos() ... Math.abs()
```

```javascript
// NOW A QUESTION FOR YOU
console.log(Math.abs(Math.floor(-4.5)))
// What should I return?
// answer:5
```

#### 布尔运算
#### 逻辑运算
```javascript
a == b // Equal value 
a === b // Equal value and type 
a != b // Different value
a !== b// Different value or type
! // not
|| // or
&& // and
```

#### 位运算
```javascript
& // and
| // or
~ // not
^ // xor
<< // left shift
>> // right shift
```

#### 条件语句模板
```javascript
if {condition1}{
  codeblock
}else if {condition2}{
  codeblock
}else{
  codeblock
}
```

#### 循环语句模板
```javascript
for (variable=initial_val; condition; increment of var){
  code...
}
```

#### 函数语句模板
```javascript
var name = function(parameter1,parameter2){
  code
  return x
}
```
