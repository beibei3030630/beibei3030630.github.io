---
title: 理解对象--Object.keys 、Object.defineProperty、Proxy、Reflect
catalog: true
date: 2022-11-07 11:00:54
subtitle:
header-img: /img/header_img/lml_bg.jpg
lang: cn
tags:
 - js对象
 - Object.keys
 - Object.defineProperty
 - Proxy
 - Reflect
categories:
 - javascript
---
#### Object.keys&Object.defineProperty
**Object.keys**     可以拿到对象的所有可枚举属性，组成一个字符串数组。
<br/>

**Object.defineProperty**   
    直接在一个对象上定义一个新属性，或者修改一个对象的现有属性，并返回此对象。
    get 在读取属性时会触发get
    set 在写入属性的时候调用的函数
    **get与set中的this对象都指向托管的对象，就是第一个参数**


```javascript
   function observe(data) {
        Object.keys(data).forEach((key) => {
          if (data[key] && typeof data[key] == "object") {
            //如果对象属性为对象继续往下去递归
            arguments.callee(data[key]);
          }
          defineReactive(data, key, data[key]);
        });
      }
      function defineReactive(data, key, val) {
        Object.defineProperty(data, key, {
          enumerable: true,
          configurable: true,
          get() {
            console.log(val);
            return val;
          },
          set(newVal) {
            if (val === newVal) {
              return;
            }
            val = newVal;
            console.log(`属性值${key}被监控，现在值为${newVal.toString()}`);
          },
        });
      }
      let obj = {
        book1: {
          info: {
            name: "vue指南",
          },
        },
        book2: "react指南",
      };
      //托管对象obj下面的所有属性
      observe(obj);

      //如果把属性值赋值给变量，会触发get方法,
      //***前后触发三次，分别是obj.book1的get方法，
      //obj.book1.info的get方法，以及obj.book1.name的get方法
      let test = obj.book1.info.name;

      //即使什么操作都不做，只是调用一下 obj.book1.info.name;
      //***同样也是前后触发三次，分别是obj.book1的get方法，
      //obj.book1.info的get方法，以及obj.book1.name的get方法
      obj.book1.info.name;

      //给属性值赋值会触发到上次对象属性的get方法为止，
      //不触发本层get方法，因为到本层就换成触发set方法了
      //****先触发obj.book1的get方法，   {info:{...}}
      //然后触发obj.book1.info的get方法   {name:{...}}
      //最后触发obj.book1.info.name的set方法   属性值name被监控，现在值为vue权威指南
      obj.book1.info.name = "vue权威指南";

      //同样的因为obj.book2没有上层的对象属性，所以直接是触发obj.book2的set方法
      //******只触发obj.book2的set方法   属性值book2被监控，现在值为react权威指南
      obj.book2 = "react权威指南";
```
资料：[Object.defineProperty()详解](https://www.cnblogs.com/ldq678/p/13854113.html)


 #### Proxy
      Proxy 可以理解成，在目标对象之前架设一层“拦截”，外界对该对象的访问，都必须先通过这层拦截，因此提供了一种机制，可以对外界的访问进行过滤和改写。Proxy 这个词的原意是代理，用在这里表示由它来“代理”某些操作，可以译为“代理器”。


 #### Reflect
    Reflect对象的方法与Proxy对象的方法一一对应，只要是Proxy对象的方法，就能在Reflect对象上找到对应的方法。这就让Proxy对象可以方便地调用对应的Reflect方法，完成默认行为，作为修改行为的基础。也就是说，不管Proxy怎么修改默认行为，你总可以在Reflect上获取默认行为。


```javascript
  Proxy(target, {
    set: function(target, name, value, receiver) {
    var success = Reflect.set(target, name, value, receiver);
    if (success) {
      console.log('property ' + name + ' on ' + target + ' set to ' + value);
    }
    return success;
  }
});
```

```javascript
 let obj = {
  name: 'Eason',
  age: 30
}

 let handler = {
  get (target, key, receiver) {
    console.log('get', key)
    return Reflect.get(target, key, receiver)
  },
  set (target, key, value, receiver) {
    console.log('set', key, value)
    return Reflect.set(target, key, value, receiver)
  }
}
  let proxy = new Proxy(obj, handler)

  proxy.name = 'Zoe' // set name Zoe
  proxy.age = 18 // set age 18

```


#### Object.defineProperty()与Proxy区别
**相同点：**
  都可以单独使用set或者是get不需要成对出现
<br/>

**不同点：**
  Object.defineProperty()只针对对象的某个属性，所以需要使用Object.keys+递归遍历对obj进行遍历。
  Proxy针对的是整个对象。

#### 除了vue源码以外用到的地方
![](image.png)
![](image2.png)
