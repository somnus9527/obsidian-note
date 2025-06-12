---
tags:
  - 知识笔记/基础知识
---
#### 关系图
![[Pasted image 20250110131633.png]]

#### 核心规则

- 函数对象才有prototype，**proto**是所有对象都有的属性
- **proto**对象，有两个属性，_constructor_和 ___proto___
- prototype只有一个属性 _constructor_
>[!Example] 示例
> ```javascript
> function Person(country) {
>   this.country = country;
> }
> 
> Person.prototype.getCountry = function() {
>   return this.country;
> }
> 
> let hanMeiMei = new Person("United States");
> 
> console.assert(hanMeiMei instanceof Person);
> console.assert(hanMeiMei.__proto__ === Person.prototype);
> console.assert(hanMeiMei.__proto__.__proto__ === Object.prototype);
> console.assert(Person.prototype.__proto__ === Object.prototype);
> console.assert(Function.__proto__ === Function.prototype);
> console.assert(Object.prototype.__proto__ === null);
> ```

#### 几个特殊情况
- Function.**proto** === Function.prototype _这个就离谱，鸡生蛋还是蛋生鸡？？_
- Object.prototype.**proto** === null
- Function.prototype.bind 方法没有prototype
#### 小扩展
>这里有个小场景，叫做原型污染

>[!Example] 比如将接口返回改造成这样:
>`{__proto__: {admin: 1}}`

那么当代码将这个json存下来，使用这个对象时，就会出现误用，造成系统出问题

处理方式有几种：

1. 处理外来JSON时先用Object.create(null)，将原型链限死 （可以参见**对象的创建**）
2. 通过Object.freeze冻结属性，使之不可扩展
3. 合并对象时，过滤掉 **proto**, prototype这些key
4. 基于原型链的继承

>[!Example] 示例
> ```javascript
> function Student(name,age,sex) {
>   // 继承Person的属性和方法,如果只需要继承Person的属性和方法这一行就够了
>   Person.call(this,name,age);
>   this.sex = sex;
> }
> // 继承Person原型链上的属性和方法
> Student.prototype = Object.create(Person.prototype);
> // 标识Student的constructor是它自己,方便判断实例是父类构建还是子类构建,如果不关注创建者,这一行也不关键
> Student.prototype.constructor = Student;
> ```

