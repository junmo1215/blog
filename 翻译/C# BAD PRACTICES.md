---
date: 2016-07-13 21:37
status: public
title: 'C# 错误实践:通过错误的范例学习如何编写良好的代码'
---

**原文地址：** [http://www.codeproject.com/Articles/1083348/Csharp-BAD-PRACTICES-Learn-how-to-make-a-good-code](http://www.codeproject.com/Articles/1083348/Csharp-BAD-PRACTICES-Learn-how-to-make-a-good-code)

##介绍
我的名字是Radoslaw Sadowski， 是微软的一名软件开发工程师。我的职业生涯一开始就是用微软技术工作着。

多年的工作经验之后我看到了许多糟糕的代码。这些糟糕的编码能供我写一本书来展示了。
这些经验使得我成为一名代码非常简洁的人。

这篇文章的目的之一是通过一些糟糕的示例向大家展示如何编写简洁、扩展性良好以及可维护的代码。我将解释它将带来的问题以及如何修复这些问题——通过好的编程实践和设计模式。

第一部分是针对知晓C#语言基础知识的开发者——将展示一些基础的错误和如何使得代码像书本一样可读的方法
下一部分是针对至少理解基本设计模式的开发者——将展示完全的、可执行单元测试的代码。

为了理解这篇文章，你需要拥有以下知识：
- C#语言
- 依赖注入模式， 工厂方法和设计模式

这篇文章中的示例是实实在在存在的——我不会像修饰模式那样制作披萨或者战略模式那样制作计算器的方式来给你们展示这些示例（这句看不太懂，感觉像是说教科书上的哪些例子是不能用在实际工作上的样子。。。）
<img src="http://7xrop1.com1.z0.glb.clouddn.com/translate/CS_BAD_PRACTICES/Calculator.png" width = "200px" height = "200px" alt="计算器" /><img src="http://7xrop1.com1.z0.glb.clouddn.com/translate/CS_BAD_PRACTICES/Pizza-Pepperoni.png" width = "200px" height = "200px" alt="披萨饼" />

虽然理论上的例子是很好的解释，但是我发现他们很难用在真实的项目上

我们听到过很多次不要使用**this**而是使用**that**代替。但是为什么呢？我将努力解释这一点并且证明良好的实践和设计模式真的能拯救我们的生命！

**注意**：
- 我将不会使用C#语言特性和设计模式（这将使得这篇文章非常长），在网络上有很多非常好的理论示范。我将专注于展示如何将他们使用在我们的日常工作中。
- 例子能非常好的佐证观点——但是我发现从成吨的示例代码中理解一个文章的大意是非常困难的。
- 我不是向你展示下面这些问题的唯一方法，但是毫无疑问的能使你的代码和工作更加高质量。
- 我不在意下面这些代码中错误的异常处理以及日志记录等。这些代码仅仅只用来展示普遍代码问题的解决方案。

让我们继续吧...

## 非常糟糕的class的编写
在我们的世界中存在着下面这种class：
```java
public class Class1
{
  public decimal Calculate(decimal amount, int type, int years)
  {
    decimal result = 0;
    decimal disc = (years > 5) ? (decimal)5/100 : (decimal)years/100; 
    if (type == 1)
    {
      result = amount;
    }
    else if (type == 2)
    {
      result = (amount - (0.1m * amount)) - disc * (amount - (0.1m * amount));
    }
    else if (type == 3)
    {
      result = (0.7m * amount) - disc * (0.7m * amount);
    }
    else if (type == 4)
    {
      result = (amount - (0.5m * amount)) - disc * (amount - (0.5m * amount));
    }
    return result;
  }
}
```
这是非常不好的代码。我们能想象到上面这个类扮演的角色吗？是在做一个计算器吗？

这就是我们目前为止能得到关于它的所有信息...

现在想象这是一个用来当客户在线购物的时候为客户计算折扣的**折扣管理**类
-- 真的嘛？
-- 真是太不幸了
这段代码完全的不可读、不可维护、不可扩展，并且使用了大量糟糕的实践和反对的模式。

这里具体的问题有哪些呢？
1. **命名**——我们仅仅能猜测这个方法在计算什么和这个计算的输入。从这个类中提取算法是非常困难的。
风险：
这种情况中最重要的事情是——浪费时间
<img src="http://7xrop1.com1.z0.glb.clouddn.com/translate/CS_BAD_PRACTICES/ernes-clessidra-hourglass.png" width = "200px" height = "200px" alt="ernes-clessidra-hourglass" />
如果我们需要提供给客户详细的计算方法细节或者需要对这部分代码进行修改，这将花费我们大量的时间去理解我们的计算方法。并且如果我们不使用文档记录或者重构这个代码，下一次我们或者其他人将花费同样多的时间去了解这里究竟发生了什么。同样，我们可能会在修改这个代码的时候犯很低级的错误。

2. **魔法数字**
<img src="http://7xrop1.com1.z0.glb.clouddn.com/translate/CS_BAD_PRACTICES/silvia2k1-nr.png" width = "200px" height = "200px" alt="silvia2k1-nr" />
在我们的例子中**type**这个变量指的是客户账户的状态。你能猜到它吗？**If-else if**语句选择了在折扣后如何计算商品的价格。
现在我们对账户类型1,2,3或者4是什么没有任何思路。现在我们考虑下你不得不修改**有价值的客户（ValuableCustomer）**的折扣方式的情况，你试图从其他代码中找出有价值的客户——这将花费大量的时间并且还容易错误的修改了**普通客户**账户的计算逻辑——像2或者3这样的数字是不具有描述性的。我们犯错之后客户将会非常开心，因为他们讲会得到有价值客户的折扣。

3. **不明显的bug**
由于我们的代码非常混乱以及可读性差，我们可能会很容易的遗漏一些重要的事情。试想一下现在有一个新的客户类型需要加入到我们的系统中——**金牌用户**。现在我们的方法将为这个新类型用户的所有产品返回0作为最终的价格。为什么会这样？因为如果没有满足**if-else if**条件的话（这种情况是一个没有处理的账户类型）方法将总是返回0。我们的老板将会不开心——他在有人意识到有些不对之前免费送出了很多商品。
<img src="http://7xrop1.com1.z0.glb.clouddn.com/translate/CS_BAD_PRACTICES/johnny-automatic-angry-guy.png" width = "200px" height = "200px" alt="johnny-automatic-angry-guy" />

4. **不可读**
我们都不得不承认我们的代码是非常不可读的。
不可读 = 需要花更多的时间来理解代码 + 增加出错的风险

5. **又是魔法数字**
我们了解像0.1,0.7,0.5这样的数字代表着什么吗？答案是否定的，虽然作为代码的拥有者我们应该知道。
让我们想象你不得不改变这一行：
**result = (amount - (0.5m * amount)) - disc * (amount - (0.5m * amount));**
由于方法完全的不可读你仅仅只将第一个0.5改为0.4，确遗留第二个0.5像它以前一样。这种修改也许是个bug，也许能完全满足这个修改的要求。这是由于0.5没有告诉我们任何事情。
同样的状况发生在将**years**转换为**disc**变量的时候：
**decimal disc = (years > 5) ? (decimal)5/100 : (decimal)years/100;**
这是在计算在我们系统中拥有账户时间的折扣百分比。好吧，但是该死的5是什么鬼？你能猜到这是忠实客户能得到的最大折扣吗？

6. **DRY——不要做重复的事情**
虽然不是很明显，但是在我们的方法中有很多地方都存在这重复的代码。
例如：
**disc * (amount - (0.1m * amount));**
与
**disc * (amount - (0.5m * amount));**
有着同样的逻辑。
这里只有一个静态变量有区别——我们能很简单的参数化这个变量。
如果我们不除去重复的代码我们将进入仅仅做一部分任务的处境。因为我们将不会发现我们不得不改变像示例代码里面的5个地方。以上逻辑是计算在系统中成为客户年份的折扣。所以如果我们将改变2或者3处这个逻辑，我们的系统将变得前后矛盾。

7. **一个类中有多种功能**
我们的方法中至少有三个职责：
1、 选择算法，
2、 根据账户状态计算折扣，
3、 根据成为我们客户的年份计算折扣。
这违反了**单一功能原则**。这将带来什么风险呢？如果我们需要这里的一个特性，他将同时影响其余的两个。这意味着将打破一些我们不愿意触及的特性。所以我们将不得不重新测试整个类——**浪费时间**。

## 重构。。。
在下面的9个步骤中我将为你们展示如何能避免上面说描述的风险和不好的编程实践，以实现一个干净、可维护、可单元测试并且能像书本一样可读性强的代码

### 第一步——命名，命名，命名
恕我直言，这是好的代码中最重要的一个方面。我们仅仅是改变了方法、参数和变量的名称，现在我们就能准确的了解下面的这个类代表着什么。

``` cs
public class DiscountManager
{
  public decimal ApplyDiscount(decimal price, int accountStatus, int timeOfHavingAccountInYears)
  {
    decimal priceAfterDiscount = 0;
    decimal discountForLoyaltyInPercentage = (timeOfHavingAccountInYears > 5) ? (decimal)5/100 : (decimal)timeOfHavingAccountInYears/100;
    if (accountStatus == 1)
    {
      priceAfterDiscount = price;
    }
    else if (accountStatus == 2)
    {
      priceAfterDiscount = (price - (0.1m * price)) - (discountForLoyaltyInPercentage * (price - (0.1m * price)));
    }
    else if (accountStatus == 3)
    {
      priceAfterDiscount = (0.7m * price) - (discountForLoyaltyInPercentage * (0.7m * price));
    }
    else if (accountStatus == 4)
    {
      priceAfterDiscount = (price - (0.5m * price)) - (discountForLoyaltyInPercentage * (price - (0.5m * price)));
    }

    return priceAfterDiscount;
  }
}
```
然而我们还是不知道1,2,3,4代表着什么，现在我们针对这点来做些什么！

### 第二步——魔法数字
在C#中避免魔法数字的技巧之一就是使用**枚举（enum）**代替他们。我准备了一个**AccountStatus**枚举来代替在**if-else if**结构中的魔法数字：
``` cs
public enum AccountStatus
{
  NotRegistered = 1,
  SimpleCustomer = 2,
  ValuableCustomer = 3,
  MostValuableCustomer = 4
}
```
现在来看我们重构后的类，我们能很容易的说出针对每个账户状态对应折扣的算法。混淆账户状态的风险迅速减小了。
``` cs
public class DiscountManager
{
  public decimal ApplyDiscount(decimal price, AccountStatus accountStatus, int timeOfHavingAccountInYears)
  {
    decimal priceAfterDiscount = 0;
    decimal discountForLoyaltyInPercentage = (timeOfHavingAccountInYears > 5) ? (decimal)5/100 : (decimal)timeOfHavingAccountInYears/100;

    if (accountStatus == AccountStatus.NotRegistered)
    {
      priceAfterDiscount = price;
    }
    else if (accountStatus == AccountStatus.SimpleCustomer)
    {
      priceAfterDiscount = (price - (0.1m * price)) - (discountForLoyaltyInPercentage * (price - (0.1m * price)));
    }
    else if (accountStatus == AccountStatus.ValuableCustomer)
    {
      priceAfterDiscount = (0.7m * price) - (discountForLoyaltyInPercentage * (0.7m * price));
    }
    else if (accountStatus == AccountStatus.MostValuableCustomer)
    {
      priceAfterDiscount = (price - (0.5m * price)) - (discountForLoyaltyInPercentage * (price - (0.5m * price)));
    }
    return priceAfterDiscount;
  }
}
```

### 第三步——更加可读
在这一步中我们将使用**switch-case**结构来代替**if-else if**结构来提高代码的可读性。

我也将很长的一行代码分隔成两行独立的代码。我们现在已经将“账户状态的折扣计算”从“拥有客户账户年份的折扣计算”分离出来了。

例如：
**priceAfterDiscount = (price - (0.5m * price));
priceAfterDiscount = priceAfterDiscount - (discountForLoyaltyInPercentage * priceAfterDiscount);**
用来代替
**priceAfterDiscount = (price - (0.5m * price)) - (discountForLoyaltyInPercentage * (price - (0.5m * price)));**

下面的这个代码展示了刚刚说的改变：
``` cs
public class DiscountManager
{
  public decimal ApplyDiscount(decimal price, AccountStatus accountStatus, int timeOfHavingAccountInYears)
  {
    decimal priceAfterDiscount = 0;
    decimal discountForLoyaltyInPercentage = (timeOfHavingAccountInYears > 5) ? (decimal)5/100 : (decimal)timeOfHavingAccountInYears/100;
    switch (accountStatus)
    {
      case AccountStatus.NotRegistered:
        priceAfterDiscount = price;
        break;
      case AccountStatus.SimpleCustomer:
        priceAfterDiscount = (price - (0.1m * price));
        priceAfterDiscount = priceAfterDiscount - (discountForLoyaltyInPercentage * priceAfterDiscount);
        break;
      case AccountStatus.ValuableCustomer:
        priceAfterDiscount = (0.7m * price);
        priceAfterDiscount = priceAfterDiscount - (discountForLoyaltyInPercentage * priceAfterDiscount);
        break;
      case AccountStatus.MostValuableCustomer:
        priceAfterDiscount = (price - (0.5m * price));
        priceAfterDiscount = priceAfterDiscount - (discountForLoyaltyInPercentage * priceAfterDiscount);
        break;
    }
    return priceAfterDiscount;
  }
}
```

### 第四步——不明显的bug
最终我们来到了我们隐藏的bug这里！
正如刚刚所说的，我们的**ApplyDiscount**方法将在有新的账户类型存在的时候返回0并作为每个商品最终的价格。悲伤但是真实。。

我们如何调整它呢？通过抛出**NotImplementedException**异常！
<img src="http://7xrop1.com1.z0.glb.clouddn.com/translate/CS_BAD_PRACTICES/cybergedeon-AL-throwing-a-stone.png" width = "200px" height = "200px" alt="cybergedeon-AL-throwing-a-stone" />

你也许会认为——异常不是改变程序逻辑发展的吗？但是并不是这样的！

当我们的方法获取了一个我们不支持的参数**AccountStatus**，我们想立刻注意到这一点并且停止程序运行已防止系统中有非预期的操作。

这种情形**永远都不应该发生**所以我们在这种情形发生的时候不得不抛出异常。

下面被修改的代码中才没有任何情况满足的时候抛出了一个**NotImplementedException**异常——**switch-case**结构中的**default**部分：

```cs
public class DiscountManager
{
  public decimal ApplyDiscount(decimal price, AccountStatus accountStatus, int timeOfHavingAccountInYears)
  {
    decimal priceAfterDiscount = 0;
    decimal discountForLoyaltyInPercentage = (timeOfHavingAccountInYears > 5) ? (decimal)5/100 : (decimal)timeOfHavingAccountInYears/100;
    switch (accountStatus)
    {
      case AccountStatus.NotRegistered:
        priceAfterDiscount = price;
        break;
      case AccountStatus.SimpleCustomer:
        priceAfterDiscount = (price - (0.1m * price));
        priceAfterDiscount = priceAfterDiscount - (discountForLoyaltyInPercentage * priceAfterDiscount);
        break;
      case AccountStatus.ValuableCustomer:
        priceAfterDiscount = (0.7m * price);
        priceAfterDiscount = priceAfterDiscount - (discountForLoyaltyInPercentage * priceAfterDiscount);
        break;
      case AccountStatus.MostValuableCustomer:
        priceAfterDiscount = (price - (0.5m * price));
        priceAfterDiscount = priceAfterDiscount - (discountForLoyaltyInPercentage * priceAfterDiscount);
        break;
      default:
        throw new NotImplementedException();
    }
    return priceAfterDiscount;
  }
}
```

### 第五步——让我们来分析算法
在我们的例子中我们有两个给我们客户折扣的标准：
1. 账户状态
2. 在系统中拥有账户的年份

加入只计算成为客户年份的折扣的话所有的算法都会变得相似：
**(discountForLoyaltyInPercentage * priceAfterDiscount)**

但是，在计算客户常量折扣的时候有一个例外：
**0.7m * price**

所以使它像下面这种情形：
**price - (0.3m * price)**

```cs
public class DiscountManager
{
  public decimal ApplyDiscount(decimal price, AccountStatus accountStatus, int timeOfHavingAccountInYears)
  {
    decimal priceAfterDiscount = 0;
    decimal discountForLoyaltyInPercentage = (timeOfHavingAccountInYears > 5) ? (decimal)5/100 : (decimal)timeOfHavingAccountInYears/100;
    switch (accountStatus)
    {
      case AccountStatus.NotRegistered:
        priceAfterDiscount = price;
        break;
      case AccountStatus.SimpleCustomer:
        priceAfterDiscount = (price - (0.1m * price));
        priceAfterDiscount = priceAfterDiscount - (discountForLoyaltyInPercentage * priceAfterDiscount);
        break;
      case AccountStatus.ValuableCustomer:
        priceAfterDiscount = (price - (0.3m * price));
        priceAfterDiscount = priceAfterDiscount - (discountForLoyaltyInPercentage * priceAfterDiscount);
        break;
      case AccountStatus.MostValuableCustomer:
        priceAfterDiscount = (price - (0.5m * price));
        priceAfterDiscount = priceAfterDiscount - (discountForLoyaltyInPercentage * priceAfterDiscount);
        break;
      default:
        throw new NotImplementedException();
    }
    return priceAfterDiscount;
  }
}

```
现在我们有了根据账户状态计算折扣的一个统一的形式：
**price - ((static_discount_in_percentages/100) * price)**

### 第六步——另一个技巧，除去魔法数字
我们一起来看一个关于账户状态计算折扣方法的静态变量：**(static_discount_in_percentages/100)**，并且用来替换：
0.1m
0.3m
0.5m

这些数字也是同样的神秘——他们没有告诉我们任何关于他们自己的事情。

我们如果需要改变“拥有账户的年限”来计算“由于忠诚所产生的折扣”，也会遇到同样的问题。

数字5是我们的代码非常神奇。

我们需要做些什么来使我们的代码更加具有描述性！

我将使用另外一个技术来避免魔法字符串——**常量（constants）**（在C#中的关键字是**const**）。我强烈推荐在我们的应用程序中为常量创建一个静态类。

例如，我已经创建了下面的这个类：
```cs
public static class Constants
{
  public const int MAXIMUM_DISCOUNT_FOR_LOYALTY = 5;
  public const decimal DISCOUNT_FOR_SIMPLE_CUSTOMERS = 0.1m;
  public const decimal DISCOUNT_FOR_VALUABLE_CUSTOMERS = 0.3m;
  public const decimal DISCOUNT_FOR_MOST_VALUABLE_CUSTOMERS = 0.5m;
}
```

修正之后我们的**DiscountManager**类看起来像下面这样：
```cs
public class DiscountManager
{
  public decimal ApplyDiscount(decimal price, AccountStatus accountStatus, int timeOfHavingAccountInYears)
  {
    decimal priceAfterDiscount = 0;
    decimal discountForLoyaltyInPercentage = (timeOfHavingAccountInYears > Constants.MAXIMUM_DISCOUNT_FOR_LOYALTY) ? (decimal)Constants.MAXIMUM_DISCOUNT_FOR_LOYALTY/100 : (decimal)timeOfHavingAccountInYears/100;
    switch (accountStatus)
    {
      case AccountStatus.NotRegistered:
        priceAfterDiscount = price;
        break;
      case AccountStatus.SimpleCustomer:
        priceAfterDiscount = (price - (Constants.DISCOUNT_FOR_SIMPLE_CUSTOMERS * price));
        priceAfterDiscount = priceAfterDiscount - (discountForLoyaltyInPercentage * priceAfterDiscount);
        break;
      case AccountStatus.ValuableCustomer:
        priceAfterDiscount = (price - (Constants.DISCOUNT_FOR_VALUABLE_CUSTOMERS * price));
        priceAfterDiscount = priceAfterDiscount - (discountForLoyaltyInPercentage * priceAfterDiscount);
        break;
      case AccountStatus.MostValuableCustomer:
        priceAfterDiscount = (price - (Constants.DISCOUNT_FOR_MOST_VALUABLE_CUSTOMERS * price));
        priceAfterDiscount = priceAfterDiscount - (discountForLoyaltyInPercentage * priceAfterDiscount);
        break;
      default:
        throw new NotImplementedException();
    }
    return priceAfterDiscount;
  }
}
```

我希望你会赞同我们的方法比以前更加能自我解释。 :D

### 第七步——不要重复自己！
<img name="Obraz4" src="http://7xrop1.com1.z0.glb.clouddn.com/translate/CS_BAD_PRACTICES/not-repeat-myself.png" style="border-width: 0px; border-style: solid; width: 356px; height: 200px;" />

为了不重复我们的代码我们将提取一部分算法到方法的外面.

我们将使用扩展方法来达到目的。

首先我们将创建两个扩展方法

```cs
public static class PriceExtensions
{
  public static decimal ApplyDiscountForAccountStatus(this decimal price, decimal discountSize)
  {
    return price - (discountSize * price);
  }
 
  public static decimal ApplyDiscountForTimeOfHavingAccount(this decimal price, int timeOfHavingAccountInYears)
  {
     decimal discountForLoyaltyInPercentage = (timeOfHavingAccountInYears > Constants.MAXIMUM_DISCOUNT_FOR_LOYALTY) ? (decimal)Constants.MAXIMUM_DISCOUNT_FOR_LOYALTY/100 : (decimal)timeOfHavingAccountInYears/100;
    return price - (discountForLoyaltyInPercentage * price);
  }
}
```

由于这个方法的名字非常具有描述性，我不需要解释他们代表什么，现在我们在我们的示例中使用心得代码：
```cs
public class DiscountManager
{
  public decimal ApplyDiscount(decimal price, AccountStatus accountStatus, int timeOfHavingAccountInYears)
  {
    decimal priceAfterDiscount = 0;
    switch (accountStatus)
    {
      case AccountStatus.NotRegistered:
        priceAfterDiscount = price;
        break;
      case AccountStatus.SimpleCustomer:
        priceAfterDiscount = price.ApplyDiscountForAccountStatus(Constants.DISCOUNT_FOR_SIMPLE_CUSTOMERS)
          .ApplyDiscountForTimeOfHavingAccount(timeOfHavingAccountInYears);
        break;
      case AccountStatus.ValuableCustomer:
        priceAfterDiscount = price.ApplyDiscountForAccountStatus(Constants.DISCOUNT_FOR_VALUABLE_CUSTOMERS)
          .ApplyDiscountForTimeOfHavingAccount(timeOfHavingAccountInYears);
        break;
      case AccountStatus.MostValuableCustomer:
        priceAfterDiscount = price.ApplyDiscountForAccountStatus(Constants.DISCOUNT_FOR_MOST_VALUABLE_CUSTOMERS)
          .ApplyDiscountForTimeOfHavingAccount(timeOfHavingAccountInYears);
        break;
      default:
        throw new NotImplementedException();
    }
    return priceAfterDiscount;
  }
}
```
扩展方法是非常好的并且能使我们的代码简洁，但是在这个静态方法使我们的单元测试很难实现甚至是不可能实现。由于这一点我们将去除最后一步。我这么做仅仅是向你展示如何使得单元测试更简单，但是我不是单元测试的狂热粉丝。

无论如何，你不赞同我们的代码现在看起来更棒了吗？

现在我们跳到下一步！

### 第八步——去除一些不必要的行
我们应该尽可能的短的和简单的代码。更短的代码 = 更少可能的bug， 更少的时间去理解业务逻辑。
让我们使我们的示例更加简化一些吧。

我们很容易的发现我们针对客户账户有三个相同的方法调用：
**.ApplyDiscountForTimeOfHavingAccount(timeOfHavingAccountInYears);**

我们能只做一次吗？不能，因为我们有针对**未注册（NotRegistered）**用户的异常，这是由于针对年份的折扣不能适用于未注册的用户。这是对的，但是没有注册的用户的折扣时间是多少呢？
-- 0年

这种情况下的折扣总是0，所以我们能安全的将这个折扣用在未注册的用户上，现在让我们试试！

```cs
public class DiscountManager
{
  public decimal ApplyDiscount(decimal price, AccountStatus accountStatus, int timeOfHavingAccountInYears)
  {
    decimal priceAfterDiscount = 0;
    switch (accountStatus)
    {
      case AccountStatus.NotRegistered:
        priceAfterDiscount = price;
        break;
      case AccountStatus.SimpleCustomer:
        priceAfterDiscount = price.ApplyDiscountForAccountStatus(Constants.DISCOUNT_FOR_SIMPLE_CUSTOMERS);
        break;
      case AccountStatus.ValuableCustomer:
        priceAfterDiscount = price.ApplyDiscountForAccountStatus(Constants.DISCOUNT_FOR_VALUABLE_CUSTOMERS);
        break;
      case AccountStatus.MostValuableCustomer:
        priceAfterDiscount = price.ApplyDiscountForAccountStatus(Constants.DISCOUNT_FOR_MOST_VALUABLE_CUSTOMERS);
        break;
      default:
        throw new NotImplementedException();
    }
    priceAfterDiscount = priceAfterDiscount.ApplyDiscountForTimeOfHavingAccount(timeOfHavingAccountInYears);
    return priceAfterDiscount;
  }
}
```
我们能将这一行移到switch-case结构的外面。好处——减少代码量

### 第九步——进一步——最终得到简洁的代码
好了！现在我们能像书本一样阅读我们的代码了，但是这对我们来说仍然不够！我们现在想进一步简化我们的代码！
<img name="Obraz4" src="http://7xrop1.com1.z0.glb.clouddn.com/translate/CS_BAD_PRACTICES/cleaner.png" style="border-width: 0px; border-style: solid; width: 356px; height: 200px;" />
现在做一些改动就能实现最终的目标了。我们将使用**工厂模式**的**依赖注入**和**策略**。

现在我们的code变成了下面这样：
``` java
public class DiscountManager
{
  private readonly IAccountDiscountCalculatorFactory _factory;
  private readonly ILoyaltyDiscountCalculator _loyaltyDiscountCalculator;
 
  public DiscountManager(IAccountDiscountCalculatorFactory factory, ILoyaltyDiscountCalculator loyaltyDiscountCalculator)
  {
    _factory = factory;
    _loyaltyDiscountCalculator = loyaltyDiscountCalculator;
  }
 
  public decimal ApplyDiscount(decimal price, AccountStatus accountStatus, int timeOfHavingAccountInYears)
  {
    decimal priceAfterDiscount = 0;
    priceAfterDiscount = _factory.GetAccountDiscountCalculator(accountStatus).ApplyDiscount(price);
    priceAfterDiscount = _loyaltyDiscountCalculator.ApplyDiscount(priceAfterDiscount, timeOfHavingAccountInYears);
    return priceAfterDiscount;
  }
}

public interface ILoyaltyDiscountCalculator
{
  decimal ApplyDiscount(decimal price, int timeOfHavingAccountInYears);
}
 
public class DefaultLoyaltyDiscountCalculator : ILoyaltyDiscountCalculator
{
  public decimal ApplyDiscount(decimal price, int timeOfHavingAccountInYears)
  {
    decimal discountForLoyaltyInPercentage = (timeOfHavingAccountInYears > Constants.MAXIMUM_DISCOUNT_FOR_LOYALTY) ? (decimal)Constants.MAXIMUM_DISCOUNT_FOR_LOYALTY/100 : (decimal)timeOfHavingAccountInYears/100;
    return price - (discountForLoyaltyInPercentage * price);
  }
}

public interface IAccountDiscountCalculatorFactory
{
  IAccountDiscountCalculator GetAccountDiscountCalculator(AccountStatus accountStatus);
}
 
public class DefaultAccountDiscountCalculatorFactory : IAccountDiscountCalculatorFactory
{
  public IAccountDiscountCalculator GetAccountDiscountCalculator(AccountStatus accountStatus)
  {
    IAccountDiscountCalculator calculator;
    switch (accountStatus)
    {
      case AccountStatus.NotRegistered:
        calculator = new NotRegisteredDiscountCalculator();
        break;
      case AccountStatus.SimpleCustomer:
        calculator = new SimpleCustomerDiscountCalculator();
        break;
      case AccountStatus.ValuableCustomer:
        calculator = new ValuableCustomerDiscountCalculator();
        break;
      case AccountStatus.MostValuableCustomer:
        calculator = new MostValuableCustomerDiscountCalculator();
        break;
      default:
        throw new NotImplementedException();
    }
 
    return calculator;
  }
}

public interface IAccountDiscountCalculator
{
  decimal ApplyDiscount(decimal price);
}
 
public class NotRegisteredDiscountCalculator : IAccountDiscountCalculator
{
  public decimal ApplyDiscount(decimal price)
  {
    return price;
  }
}
 
public class SimpleCustomerDiscountCalculator : IAccountDiscountCalculator
{
  public decimal ApplyDiscount(decimal price)
  {
    return price - (Constants.DISCOUNT_FOR_SIMPLE_CUSTOMERS * price);
  }
}
 
public class ValuableCustomerDiscountCalculator : IAccountDiscountCalculator
{
  public decimal ApplyDiscount(decimal price)
  {
    return price - (Constants.DISCOUNT_FOR_VALUABLE_CUSTOMERS * price);
  }
}
 
public class MostValuableCustomerDiscountCalculator : IAccountDiscountCalculator
{
  public decimal ApplyDiscount(decimal price)
  {
    return price - (Constants.DISCOUNT_FOR_MOST_VALUABLE_CUSTOMERS * price);
  }
}
```

（未完）