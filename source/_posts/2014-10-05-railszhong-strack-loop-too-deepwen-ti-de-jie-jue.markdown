---
layout: post
title: "rails中strack loop too deep 问题的解决"
date: 2014-10-05 10:16:12 +0800
comments: true
categories: rails
---

在将一个Rails3.1.2的应用升级到Rails4.0后，突然发现以前运行的好好的程序无法使用了，抛出"Stack loop too deep"这个异常。
雅美蝶！！！怎么可能，我这个地方又没使用递归，我瞬间打开IDE，找到了出问题的地方。

```
  def transcation_in (detail, operator='系统')
    self.money = self.money + detail.money
    detail.account_type = STATE[:in]
    detail.updateby = operator
    detail.save
    self.account_details << detail
  end
```

如代码所见，逻辑很简单，就是对一个Model对象的简单保存，并加入到关联对象中。这哪里会出现堆栈太深呢。

随后我加入了调试日志

```
  def transcation_in (detail, operator='系统')
    puts 'start transcation_in'
    begin
      self.money = self.money + detail.money
      detail.account_type = STATE[:in]
      detail.updateby = operator
      detail.save
      self.account_details << detail
    rescue Exception => e
      puts "====#{e}"
    end
    puts 'end transcation_in'
  end
```

查看了日志，异常是在执行“detail.save”这个代码的是报的异常。这明明很正常呢。。。。日怪了。

调试日志看不出东西来，那我就进入了debugger模式，发现执行到这一步时，进入了active_record.transactions.rb这个类里面的save方法，但我没开启事务呢。难道是因为方法名字的问题？？？这有点扯蛋了。。。

我查看了我的Mysql数据表，确实是MyISAM引擎，改成InnoDB试试？尼玛，成功了！！！这里果然用了事务，当数据库表不支持事务时，就抛出了异常！！！

这错误异常信息也太扯蛋了嘛，提示和事务一点关系也没有！

希望这篇文章能为遇到同类问题的人提供解决办法，但问题的根源我还是没找到，为什么会用到事务了呢。