---
layout: post
title:  "ViewGroup事件分发"
date:   2018-05-26 14:53:00 +0800
categories: Android
tags: 自定义ViewGroup
author: pepe
description: 『 View事件分发 』
---

自定义ViewGroup的四个方法：
* ① onInterceptTouchEvent(MotionEvent ev)
* ② dispatchTouchEvent(MotionEvent ev)
* ③ onTouchEvent(MotionEvent event)
* ④ requestDisallowInterceptTouchEvent(boolean disallowIntercept)

* `MotionEvent.ACTION_DOWN`:

    。 `mMotionTarget = null`;
        
    。 如果不拦截，能找到对应的子view，清除子`view.mPrivateFlags` 的 `CANCEL_NEXT_UP_EVENT`;  
    
        - 如果该子view消费事件：mMotionTarget = child，return  true;
        - 如果该子view不消费事件：mMotionTarget 仍然为 null
    
    。 不能找到对应的子view，view.mPrivateFlags 包含 CANCEL_NEXT_UP_EVENT，同时 mMotionTarget 仍然为 null
    
    。 继续往下看，
    
    。 `final View target = mMotionTarget;`
    
    。 `return super.dispatchTouchEvent(ev);`
    
* `MotionEvent.ACTION_MOVE`:

    。 `final View target = mMotionTarget;`
    
    。 如果 `target == null ,return super.dispatchTouchEvent(ev);`
    
    。 如果 `target != null` ,再次判断是否拦截
    
        - 如果拦截， `mMotionTarget = null，return true;` 那么下一部 up 也就是一样的了，交给 super
        - 如果不拦截，判断 view.mPrivateFlags 是否包含 CANCEL_NEXT_UP_EVENT
            - 如果包含，也就是说要取消咯，那么 `mMotionTarget = null;` 下一步 交给 super
            - 如果不包含，`return target.dispatchTouchEvent(ev);`  正常执行
            
* `MotionEvent.ACTION_UP`:

    。 `final View target = mMotionTarget;`
    
    。 如果 `target == null ,return super.dispatchTouchEvent(ev);`
    
    。 如果 target != null ,再次判断是否拦截
    
        - 如果拦截， `mMotionTarget = null，return true;`
        - 如果不拦截，`mMotionTarget = null;`
        - 然后 `return target.dispatchTouchEvent(ev);`  正常执行
 
总结一下：
 
* 1、`ACTION_DOWN`中，`ViewGroup`捕获到事件，然后判断是否拦截，如果没有拦截，则找到包含当前x,y坐标的子View，赋值给`mMotionTarget`，然后调用	`mMotionTarget.dispatchTouchEvent`

* 2、`ACTION_MOVE`中，`ViewGroup`捕获到事件，然后判断是否拦截，如果没有拦截，则直接调用`mMotionTarget.dispatchTouchEvent(ev)`

* 3、`ACTION_UP中`，`ViewGroup`捕获到事件，然后判断是否拦截，如果没有拦截，则直接调用`mMotionTarget.dispatchTouchEvent(ev)`

当然了在分发之前都会修改下坐标系统，把当前的x，y分别减去`child.left`和`child.top`，然后传给child;

* 4、如果`MotionEvent`最终都没有被消费，例如`dispatchTouchEvent`在`Action_Down`返回了false，那么系统便不会再分发`Action_Move`或者`Action_UP`
* 5、如果`View`不消耗除`Action_Down`以为的其他事件，那么这个点击事件会消失，此时父元素的`onTouchEvent`并不会被调用，并且当前View可以持续受到后续的事件，最终这些消失的点击事件会传递给`Activity`处理。

说明：

* 一、只要我们的`onInterceptTouchEvent return true `那么我们的`MotionEvent`与ChildView 无缘
* 二、如果我们的`onInterceptTouchEvent  return false;`那么我们的ChildView  会优先获得`MotionEvent`。

    。 但是当我们的`ChildView`并不在`TouchTarget`上，我们的ViewGroup依然有机会得到本次`MotionEvent`。
    
    。 获得之后执行`super.dispatchTouchEvent(ev)`；也就是把`ViewGroup`当做一个View了。
    
* 三、`onInterceptTouchEvent`并不会在每一次`MotionEvent`事件（`ACTION_DOWN、ACTION_MOVE、ACTION_UP` 等）调用，

    。 例如果在`ACTION_DOWN `时，`onInterceptTouchEvent` 拦截，`return true`交给了`ViewGroup` 而View没有得到的话，`ACTION_MOVE`时就不会调用，
    
    。 但是如果`return false`，`ChildView`得到了`ACTION_MOVE`时，`onInterceptTouchEvent`就会再次调用


    
    

* 1、进入ViewGroup：onInterceptTouchEvent,判断是否拦截，两个参数：`disallowIntercept || !onInterceptTouchEvent(ev)` 。

    。 `disallowIntercept` 默认是false，主要是根据 `onInterceptTouchEvent(ev)`来判断，这个默认也是false，！之后就是true了。也就是不拦截。
    
    。 `disallowIntercept` 可以通过`requestDisallowInterceptTouchEvent(true)`方法进行修改，修改flag。
    
    。 默认不拦截：查找是哪个 child 的坐标范围，进入该 child 的 dispatchTouchEvent。
    
        - 如果 dispatchTouchEvent 返回true，设置 `mMotionTarget = child`。同时返回true。
        - 拦截不成功，`mMotionTarget = null`，那么  `return super.dispatchTouchEvent(ev)`;  也就是把ViewGroup 当做一个View了。
        
    。 如果拦截：`onInterceptTouchEvent(ev)` 返回 `true`。`mMotionTarget = null`。
    
        - 直接进入：`return super.dispatchTouchEvent(ev)`;  也就是把ViewGroup 当做一个View了。
        
* 2、整个ViewGroup的事件分发流程：

    。 Android事件分发是先传递到ViewGroup，再由ViewGroup传递到View的。
    
    。 在ViewGroup中可以通过 onInterceptTouchEvent 方法对事件传递进行拦截，onInterceptTouchEvent方法返回true代表不允许事件继续向子View传递，返回false代表不对事件进行拦截，默认返回false。
    
    。 子View中如果将传递的事件消费掉，ViewGroup中将无法接收到任何事件。
* 3、
    。 ACTION_DOWN中，ViewGroup捕获到事件，然后判断是否拦截，如果没有拦截，则找到包含当前x,y坐标的子View，赋值给mMotionTarget，然后调用	mMotionTarget.dispatchTouchEvent
    
        - 如果被拦截，`mMotionTarget = null`，执行 ViewGroup 的 `super.dispatchTouchEvent(ev)`。
        
    。 ACTION_MOVE中，ViewGroup捕获到事件，然后判断是否拦截，如果没有拦截，则直接调用mMotionTarget.dispatchTouchEvent(ev)
    
        - 如果被拦截，mMotionTarget执行完dispatchTouchEvent后， mMotionTarget = null，然后执行一次 MotionEvent.ACTION_CANCEL。
        
    。 ACTION_UP中，ViewGroup捕获到事件，然后判断是否拦截，如果没有拦截，则直接调用mMotionTarget.dispatchTouchEvent(ev)
    
        - 如果被拦截，mMotionTarget执行完dispatchTouchEvent后， mMotionTarget = null,然后执行一次 MotionEvent.ACTION_CANCEL。
        
    当然了在分发之前都会修改下坐标系统，把当前的x，y分别减去child.left 和 child.top ，然后传给child;
    
* 4、如何不被拦截？

    。 如果ViewGroup的`onInterceptTouchEvent(ev) `当ACTION_MOVE时return true ，即拦截了子View的MOVE以及UP事件；
    
    。 此时子View希望依然能够响应MOVE和UP时该咋办呢？
    
    。 Android给我们提供了一个方法：requestDisallowInterceptTouchEvent(boolean) 用于设置是否允许拦截。
    
    。 `getParent().requestDisallowInterceptTouchEvent(true);`
    
    。 当我们把disallowIntercept设置为true时，`!disallowIntercept`直接为false，于是拦截的方法体就被跳过了~
    
    。 子View可以通过调用getParent().requestDisallowInterceptTouchEvent(true);  阻止ViewGroup对其MOVE或者UP事件进行拦截。




参考：

[Andriod 从源码的角度详解View,ViewGroup的Touch事件的分发机制 - CSDN博客](https://blog.csdn.net/xiaanming/article/details/21696315)

[Android ViewGroup事件分发机制 - Hongyang - CSDN博客](http://blog.csdn.net/lmj623565791/article/details/39102591/)

[Android事件分发机制完全解析，带你从源码的角度彻底理解(下) - 郭霖的专栏 - CSDN博客](http://blog.csdn.net/guolin_blog/article/details/9153747)

上面三篇文章足矣！

[Android 手把手教您自定义ViewGroup（一） - Hongyang - CSDN博客](http://blog.csdn.net/lmj623565791/article/details/38339817/)

[Android 自定义控件打造史上最简单的侧滑菜单 - Hongyang - CSDN博客](http://blog.csdn.net/lmj623565791/article/details/39185641)


dispatchTouchEvent分析：
* 1、如果是down  先清空上次down的信息
* 2、判断是否拦截：

    。 a、如果是down，或者mFirstTouchTarget != null
    
    。 b、判断group的tag，是否允许拦截
    
        - 如果允许拦截，执行intercepted = onInterceptTouchEvent(ev);
        - 如果不允许拦截，intercepted = false;
        
    。 c、如果不是down，并且 或者mFirstTouchTarget == null，那么intercepted = true;
    
* 3、如果不是取消，并且不是拦截



[Android事件分发机制--ViewGroup(二) - CSDN博客](http://blog.csdn.net/dmk877/article/details/49055815)
    
[Android中的dispatchTouchEvent()、onInterceptTouchEvent()和onTouchEvent() - 张兴业的博客 - CSDN博客](
http://blog.csdn.net/xyz_lmn/article/details/12517911)

[onInterceptTouchEvent 与 onTouchEvent 分析与MotionEvent在ViewGroup与View中的分发 - 大叔的愤怒，你驾驭不了 - CSDN博客](
http://blog.csdn.net/jaysong2012/article/details/46909959)

[Android View 事件分发机制源码详解(ViewGroup篇) - CSDN博客](https://blog.csdn.net/a553181867/article/details/51287844)



