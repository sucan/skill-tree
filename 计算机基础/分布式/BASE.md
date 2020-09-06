# BASE 理论

## 简介

BASE 这个缩写同样由Eric Brewer定义（就是提出CAP定理的那个），含义如下：
> **B**asically **A**vailable, **S**oft state, **E**ventual consistency

BASE系统通过牺牲强一致性来保证可用性和分区一致性，通常被应用于分区数据库

## Basically Available

基本可用：

也就是说BASE系统会保证CAP理论中的A


## Soft state

软状态：

表明当前系统的状态有可能会一直变化，即使没有任何输入


## Eventual consistency

最终一致性：

当系统没有任何输入之后，系统将会随着时间的推移变得一致

## 参考资料

1. [stackoverflow](https://stackoverflow.com/questions/3342497/explanation-of-base-terminology#:~:text=The%20BASE%20acronym%20was%20defined%20by%20Eric%20Brewer%2C,tolerance%3B%20A%20BASE%20system%20gives%20up%20on%20consistency.)
2. [Base: An Acid Alternative](https://queue.acm.org/detail.cfm?id=1394128)