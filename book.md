# 《重构,改善既有代码设计》读书笔记

## 第1章 一个重构案例

笔者直接拿了一个 顾客(customer)租借(rental)电影(movie)的例子

分别有三个类  顾客 dependency 租借 (1 to *)， 租借 dependency 电影 (1 to 1)

最开始是一个大又长的大方法（它干了很多件事情），违背了RSP职责以及对扩展开放对修改闭合原则
，于是尝试对大方法进行重构
这个大方法是　statement() ，列出顾客的详单

后面来了一个新需求，需要一个html的顾客详单，在现有框架下只能再复制一个代码出来（坏味道：重复代码）

1、在重构前保证有一个可运行的测试代码(单测)，用于保证你每次的重构是正确的方向（每次小重构都run一遍）
2、先将statement方法中进行拆解，分别做了什么事情，拆解成一个个小方法（增加理解力,单一职责）
3、按照单一职责将这些方法归属到相应的类中(之前在customer下，把方法按照职责放到Movie和rental下)
4、同时修改方法中的变量名称（增加理解力），使得更好理解
5、将if和else改为多态实现，由于可能租借的movie类型不同会计算不同的费率
6、由于在Movie的生命周期内可能会改变电影类型，会影响到费率计算，所以需要引入Price这个概念(类)，
这里采用了state状态模式,所有的类实现这个接口

在如上这6个重构步骤之后，如需要使用html的详单，则完全可以复用如上代码


## 第2章 重构原则
该章节讲述了 重构的时机,好处(benefit)，以及何时不该重构

### 重构时机
1. 在添加功能时
2. CR时
3. 修复Bug时

### 不该重构时机
1. 当项目的尾声时不该重构(新的风险)

### 好处
1. 使代码更容易理解阅读（利他）
2. 修复已存在bug (没有单测需要补)
3. 提高编码速度(更多的是提高将来添加功能速度)

### 不容易重构的地方
1. 已有接口重构,采用新老接口，同时老接口调用新接口实现
2. 重构难度大，ROI低，需要花费大成本,小范围重构即可，增强理解力

### 第3章 代码坏味道

1.  过长的类，意味着需要拆解，一样是单一职责
2.  重复的代码，意味着需要抽取公共的东西,单一职责
3.  过多参数的函数,最多3个[0, 3],更多时抽取对象或考虑拆分
4.  过大的函数，违背单一职责，进一步拆分小的函数
5.  发散式变化,因一个变化修改多个地方，违背SRP
6.  基本类型偏执,比如表达一个范围定义一个Range对象放置to和from比单纯的to和from更具备表达力
7.  没有用的类,单测,以及方法都应该直接删除
8.  过多的switch意味着可以用多态来实现
9.  函数名、类名、临时变量、参数名使人疑惑,不具备清晰的表达
10. 过度耦合的消息链, 由此我想到我们的责任链的命名规范
11. 异常缺少错误信息,出入参原因等
12. 过多的注释, 注释只应该在解释意图原因等时出现,代码更多自解释


### 第四章 构建测试体系

1. 单测缺少边界情况
2. 没有断言,无效的单测
3. 没有断言异常,或只断言异常,未断言错误码
4. void方法就不测试


### 第六章 重新组织函数
从第六章开始，讲述如何开始重构，第六章则将如何将大函数组织成更有表达力的函数

#### 6.1 提取函数(extract method)
1.  提取函数时会有几种情况
-   只有入参

#### 6.2 内联函数(inline method)
与提取函数相反，可能通过变量名就可以区分的无需要再提取

#### 6.3 查询取代临时变量

````
//改动前
double basePrice = quantity * itemPrice;



//改动后,可以复用，同时符合单一职责
double basePrice() {
    return quantity * itemPrice;
}

````

####  6.4 引入解释性变量

````
//改动前
if (platform.toUpperCase().indexOf("MAC") > -1) &&
(browser.toUpperCase().indexOf("IE") > -1) &&
wasInitialized() && resize > 0) 
{
 //do something
}



//改动后,代码更加直观的解释
final boolean isMac = platform.toUpperCase().indexOf("MAC") > -1;
final boolean isIE = browser.toUpperCase().indexOf("IE") > -1;

if (isMac && isIE && wasInitialized() && resize > 0) {
}

````

####  6.5 分解临时变量

````
//改动前 temp这个变量在两个不同逻辑使用，需要分解
double temp = 2 * (height + width);
//这边的打印只是想说明干了别的事
system.out.pring(temp);
//此书又用temp这个变量干了别的事，不符合单一职责
temp = height * width;
system.out.pring(temp);


//改动后
fnal double permeter = 2 * (height + width);
system.out.pring(permeter);
//明确一个变量只干一件事
final area = height * width;
system.out.pring(area);

````

####  6.6 移除对参数的赋值

不要在方法中对入参修改 引用（值或地址）

````
//改动前 
int discount(int inputVal, int quantity, int yearToDate) {
    if (inputVal > 50) inputVal = 2;
    return inputVal;
}



//改动后,使用临时变量存储入参，在后续使用临时变量做返回
int discount(int inputVal, int quantity, int yearToDate) {
    int result = inputVal;
    if (inputVal > 50) result = 2;
    return result;
}

````

####  6.7 以函数对象取代函数

````
//改动前 
class Order {
  double price() {
    double primaryBasePrice;
    double secondaryBasePrice;
    double teriaryBasePrice;
    //long compute price
  }
}



//改动后 引入PriceCalculator取代price中大函数计算 
class PriceCalculator {
    final Order order;
    double primaryBasePrice;
    double secondaryBasePrice;
    double teriaryBasePrice;
    
    PriceCalculator(Order order) {
      this.order = order;
      ..
    }
   
    double compute() {
     //long compute price
    }
}

class Order {
  double price() {
    return new PriceCalculator(this).compute();
  }
}


````

####  6.8 替换算法

````
//改动前 
String foundPerson(String[] person) {
    for (int i =0; i < person.length; i++) {
      if (person[i].equal("Don")) {
         return "Don";
      }
      if (person[i].equal("John")) {
               return "John";
       }
    }
    return "";
}


//改动后
Set<String> personSet = Sets.newHashSet("Don", "John");
String foundPerson(String[] person) {
    
    for (int i =0; i < person.length; i++) {
        if (personSet.contain(person[i])) {
            return personSet.get(person[i]);
        }   
    }
    return "";
}
````

### 第七章 重新组织对象

其实和函数一样的方式，
类过大、职责过多
提取函数、提取类

类过小，内联类，原则是一样的

### 第八章 重新组织数据

这一章主要想说明一个点，比如一个类，你是直接访问成本变量，还是通过函数访问

#### 8.1 

````
class Order {
  int price;
  int discount;
  
  //直接访问成员变量
  compute() {
    return price * dicount;
  }
  
  //通过函数访问
  compute() {
      return price() * dicount();
    }
    
  int price() {
    return price;
  }  
  
  int discount() {
    return discount;
  }
} 

````

#### 8.2 对象取代数据值

显然和领域建模落地的思路不谋而合，把概念显性化

````

//改动前
class Order {
    String customer;
} 

//改动后

class Order {
    Customer customer;
}

class Customer {
}


````

#### 8.3 将单向关联变为双向关联

    这个是看情况的，只有当需要双向访问时才有必要，原则上还是尽可能简单



#### 第9章 简化条件表达式

1.  将多个表达式抽出成函数，更具表达力
2.  合并表达式，如果多个if分支返回同一个值，且无副作用，则合并
3.  合并重复的条件分支
    ````
    //优化前
    if (a == 1) {
        system.out.println(a);
        send();
    } else if (b == 1) {
        system.out.println(b);
        send();
    }
    
    //优化后
    if (a == 1) {
        system.out.println(a);
       
    } else if (b == 1) {
        system.out.println(b);
    }
    //合并重复的条件分支
     send();
    
    ````
4.  以卫语句代替嵌套的表达式
卫语句（guard clauses）： 如果某个条件罕见，则该条件为真则立即从函数中返回
这边立马能想到的是函数中对于异常情况的提前校验

    ````
    //优化前
    double getPayAmount() {
      double result;
      if (isDead) result = deadAmount();
      else {
         if (isSeperated) {
           result = separeatedAmount();
         } else {
           result = normalPayAmout();
         }
      }
      return result;
    }
    
    //优化后
    double getPayAmount() {
      if (isDead) return deadAmount();
      if (isSeperated)  return separeatedAmount();
      return normalPayAmout();
    }
    
    ````

5.  引入断言

    ````
    //优化前
    double getPayAmount(double amount) {
      if amount == null throws IllegalArgumentException();
      if amount = 0 throws new BizExcpetion();
    }
    
    //优化后 ，但要强调的是不要滥用断言，只有当必须为真的条件才用
    //使用之前问下自己，如果条件表达式不匹配，是否可以继续执行，可以则不可以使用
    double getPayAmount(double amount) {
      Asserts.isFalse(amount == null, "illegal argument : " + amount);
      Asserts.isFalse(amount == 0, "wrong argument : " + amount);
    }
    
    ````



布控链需要关注顺序。
把校验，设置某个参数放到了不同的责任链中，且相互耦合影响
顺序本身就是责任链需要关注的


### 写代码的几个层次（个人思考）

1.  能完成功能,有完整的单测
2.  打印足够的日志，充分考虑边界，代码具备容错降级（防御性编程）
2.  无重复代码,遵循命名等规范
3.  易扩展，符合SRP，DIP等原则
4.  代码自解释，清晰明了,代码极具表达力，像读文章一样
5.  代码兼顾优雅情况下能具备高性能
5.  一个的CRer(影响力更大，自己写得好同时)

