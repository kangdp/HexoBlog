---
title: ViewPager在5.0设备以上引起的问题
date: 2018-03-15
updated: 2018-03-15
tags:
categories: Android
---

# 遇到的问题
当手指向上滑动页面,ViewPager被隐藏,然后向下滑动将ViewPager显示出来，此时ViewPager第一次切换没有动画效果
此问题是5.0以上的一个小bug,经过查看ViewPager源码发现,ViewPager最终是在`setCurrentItemInternal()`方法中做切换的,其源码如下：

        if (mFirstLayout) {
            mCurItem = item;
            if (dispatchSelected) {
                dispatchOnPageSelected(item);
            }
            requestLayout();
        } else {
            populate(item);
            scrollToItem(item, smoothScroll, velocity, dispatchSelected);
        }

上述中的if代码块是没有切换效果的,而else代码块中的scrollToItem()方法就是ViewPager切换动画的代码,也就是说只有mFirstLayout为真时，才可触发动画效果，而在ViewPager在可见时会回调onAttachedToWindow()方法,在此方法中只有一行代码：mFirstLayout = true, 所以当ViewPager可见时的第一次切换是无动画效果的。

# 针对这个解决方法有两种:

## 重写ViewPager并覆盖父类的`onAttachedToWindow()`方法，不让其执行父类的代码

	
	 @Override
         protected void onAttachedToWindow() {
           // super.onAttachedToWindow();
         }

## 在`onAttachedToWindow()`方法中使用反射设置`mFirstLayout `属性为`false`

	
	private void setFirstLayout(boolean isFirstLayout){		
	 try {
            Class<?> clazz = Class.forName("android.support.v4.view.ViewPager");
            Field field = clazz.getDeclaredField("mFirstLayout");
            field.setAccessible(true);
            field.setBoolean(this,isFirstLayout);
          } catch (ClassNotFoundException e) {
            e.printStackTrace();
          } catch (NoSuchFieldException e) {
            e.printStackTrace();
          } catch (IllegalAccessException e) {
            e.printStackTrace();
          }
      	}


 然后在`onAttachedToWindow()`方法中调用该方法

	
	@Override
        protected void onAttachedToWindow() {
             super.onAttachedToWindow();
             setFirstLayout(false);
      	}



