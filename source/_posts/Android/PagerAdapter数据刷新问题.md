---
title: PagerAdapter数据刷新的问题
date: 2017-03-14
updated: 2017-03-14
tags:
- PagerAdapter
categories: Android
---

# 遇到的问题
在做轮播图时，当后台的轮播图内容发生更改时，调用`adapter.notifyDataSetChanged`不起作用，通过查资料发现了两种解决方法。

# 解决方法
## 最简单的一种
PagerAdapter中有个方法`getItemPosition(Object object)`，此方法默认返回 `POSITION_UNCHANGED`，意为不做处理；当返回`POSITION_NONE`时，那么就会去调用`destroyItem(...)`方法移除当前所有已添加的View，然后调用`instantiateItem(...)`方法来重新添加View；这种方法有一个弊端，就是我们不需要更新的View也被重新加载了，这样的话就浪费了系统资源。因此这种方法适用于数据量少的情况下。

## 优化后，只刷新当前显示的页面
在`instantiateItem(...)`方法中给每个View设置Tag，将当前的`position`传入，然后在`getItemPosition(Object object)`方法中通过`((View)object).getTag()`方法获取`position`，然后与轮播图当前所显示的View的位置下标(当前位置可以通过回调接口从Activity中传进来)做比较，如果等于，那么就让`getItemPosition(Object object)`返回当前`position`，这样的话PagerAdapter就只会移除当前位置的View，并重新创建此View，而并不会刷新所有的View，避免了系统资源的浪费



