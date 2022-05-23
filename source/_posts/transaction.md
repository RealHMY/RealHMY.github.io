---
title: PROPAGATION_NESTED 和 PROPAGATION_REQUIRES_NEW 的使用
author: RealHMY
top: false
cover: false
toc: true
mathjax: false
date: 2022-05-23 14:12:26
img:
coverImg:
password:
summary:
tags:
categories:
---

## 种类
* 编程式事务：使用 TransactionTemplate 来完成
* 声明式事务：通过 AOP 来完成，一般推荐使用声明式事务

## 声明式事务
1. 以 AOP 的方式完成事务
2. 在配置文件中声明
3. 通过@Transactional注解的方式，便可以将事务规则应用到业务逻辑中
4. 最细粒度只能作用到方法级别，无法做到像编程式事务那样可以作用到代码块级别

## @Transactional 的参数
| 属性                   | 类型                                     | 描述                                                                                                                                        |
| ---------------------- | ---------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| propagation            | 枚举型：Propagation                      | 可选的传播性设置                                                                                                                            |
| isolation              | 枚举型：Isolation                        | 可选的隔离性级别（默认值：ISOLATION_DEFAULT）                                                                                               |
| readOnly               | 布尔型                                   | 读写型事务 vs. 只读型事务                                                                                                                   |
| timeout                | int型（以秒为单位）                      | 事务超时                                                                                                                                    |
| rollbackFor            | 一组Class类的实例，必须是Throwable的子类 | 一组异常类，遇到时必须进行回滚。默认情况下 checked exceptions不进行回滚，仅unchecked exceptions（即RuntimeException的子类）才进行事务回滚。 |
| rollbackForClassname   | 一组Class类的名字，必须是Throwable的子类 | 一组异常类名，遇到时必须进行回滚                                                                                                            |
| noRollbackFor          | 一组Class类的实例，必须是Throwable的子类 | 一组异常类，遇到时必须不回滚。                                                                                                              |
| noRollbackForClassname | 一组Class类的名字，必须是Throwable的子类 | 一组异常类，遇到时必须不回滚                                                                                                                |

## 传播行为 isolation
当多个事务同时存在的时候，spring如何处理这些事务的行为。

个别事务需要捕获异常的，使用try-catch来捕获异常，即在try块中调用子事务

### ROPAGATION_REQUIRED
* 如果没有事务，则创建一个新事务
* 如果存在事务，则加入该事务
* 默认事务

### PROPAGATION_REQUIRES_NEW
* 无论存不存在事务，都创建新事务
* 如果方法已运行在一个事务中，则原有事务被挂起，新的事务被创建，直到方法结束，新事务才结束，原先的事务才会恢复执行
* 内层事务因为创建了一个新的事务，所以数据与外层事务不一致

### PROPAGATION_SUPPORTS
* 如果没有事务，就以非事务执行
* 如果存在事务，则加入该事务

### PROPAGATION_NOT_SUPPORTED
* 以非事务方式执行操作
* 如果存在事务，则把当前事务挂起

### PROPAGATION_MANDATORY
* 如果当前没有事务，则抛出异常
* 如果存在事务，则加入该事务

### PROPAGATION_NEVER
* 以非事务方式执行
* 如果当前存在事务，则抛出异常

### PROPAGATION_NESTED
* 如果没有事务，则按REQUIRED属性执行
* 如果存在事务，则在嵌套事务内执行

## 事务不生效的情况
* 数据库不支持事务
  * 如：MySQL的mylsam引擎不支持，而innodb支持
* 没有配置事务管理器
* Service类没有被Spring管理
  * 如：没标注@Service
* 没有事务的方法调用有事务的方法
* 非RuntimeException
  * 默认支持的是RuntimeException，只有发生该异常时或该子类异常时才回滚
  * 手动抛出非运行时异常也不会回滚
  * 如果需要解决这个问题，通过设置@Transactional的 rollbackFor 属性即可
* 方法不是public
* 多线程
  * 如：子线程抛异常，主线程无法捕获，导致事务不生效
* 被捕获了，又没有抛出来
  * 正常情况下，需要try-catch来捕获，然后在catch块或finally块中抛出运行时异常
* final 方法
  * AOP无法重写该方法，从而添加事务功能

通常来说，for循环内调用带有事务的方法，若想事务之间不受影响，for循环调用的事务方法可以加上REQUIRES_NEW

注意：传播行为基于事务，如果是controller调用事务方法，则不一定起效，必须是事务方法通过容器调用事务方法，使得事务生效才行

### REQUIRES_NEW 和 NESTED 的测试

测试例子如下：
```java
class Entity {
  private String name;
  ... setter getter
}

@Transactional
public void saveOutter(Entity entity) {
    dao.save(entity);
}

@Transactional
// @Transactional(propagation = Propagation.REQUIRES_NEW)
// @Transactional(propagation = Propagation.NESTED)
public void saveInner(Entity entity) {
    if ("inner2".equals(entity.getName())) {
        throw new RuntimeException("抛出内部异常");
    }
    dao.save(entity);
}

@Transactional
public void testSave() {
    // 模拟从容器获取Service
    // 不能直接调用同个类的方法，需要通过注入的bean去调用，这样才受spring的控制
    EntityService entityService =  SpringContextHolder.getBean(EntityService.class);

    Entity outter1 = new Entity();
    outter1.setName("outter1");
    violationService.saveOutter(outter1);

    for (int i = 1; i <= 3; i++) {
        Entity inner = new Entity();
        inner.setName("inner" + i);
        // try {
            entityService.saveInner(inner);
        // } catch (Exception ignored) {}
    }
    // if(true) {
    //     throw new RuntimeException("抛出外出异常");
    // }

    Entity outter2 = new Entity();
    outter2.setName("outter2");
    entityService.saveOutter(outter2);
}
```

例子代码说明：一共需要保存5个实体：outter1、inner1、inner2、inner3、outter2，其中保存inner2时抛出内部异常，保存outter2前抛出外部异常

实验结果：

条件 \ 对象 | outter1 | inner1 | inner2 | inner3 | outter2 | 运行结果
-|-|-|-|-|-|-
无传播行为 | x | x | x | x | x | 抛出内部异常
无传播行为+catch内部异常 | x | x | x | x | x | rollback-only异常
REQUIRES_NEW | x | o | x | x | x | 抛出内部异常
REQUIRES_NEW+捕获内部异常 | o | o | x | o | o | 运行结束
REQUIRES_NEW+捕获内部异常+抛出外部异常 | x | o | x | o | x | 抛出外部异常
NESTED | x | x | x | x | x | 抛出内部异常<br>（因为是同一事务，故一起回滚）
NESTED+捕获内部异常 | o | o | x | o | o | 运行结束

总结：

| |REQUIRED|REQUIRES_NEW|NESTED|
|-|-|-|-|
|外层正常，内层正常|一起提交|内层先提交，外层再提交|一起提交
|外层正常，内层异常|一起回滚|**内外层属于不同事务**<br>若外层捕获内层异常，则内层回滚，外层再提交<br>若外层没捕获内层异常，则内层回滚，外层再回滚|**内外层属于同一事务**<br>若外层捕获内层异常，则内层回滚，外层再提交<br>若外层没捕获内层异常，则内层回滚，外层再回滚|
|外层异常，内层正常|一起回滚|外层回滚，内层提交|一起回滚
|外层异常，内层异常|一起回滚|内层先回滚，外层再回滚|一起回滚