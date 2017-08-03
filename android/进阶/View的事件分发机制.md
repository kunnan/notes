**View的事件分发机制**

[TOC]

# 点击事件的传递规则

首先明确我们这里要分析的对象就是MotionEvent，及点击事件。所谓点击事件的事件分发，其实就是对MotionEvent事件的分发过程，即当一个MotionEvent事件产生了以后，系统需要把这个事件传递给一个具体的View，而这个传递的过程就是分发过程。点击事件的分发过程由三个很重要的方法来共同完成：dispatchTouchEvent、onInterceptTouchEvent和onTouchEvent。

*	**public boolean dispatchTouchEvent(MotionEvent event)**

用来进行事件的分发。如果事件能够传递给当前View，那么此方法一定会被调用，返回结果受当前View的onTouchEvent和下级的dispatchTouchEvent方法的影响，表示是否消耗此事件

*	**public boolean onInterceptTouchEvent(MotionEvent event)** //view无此方法，存在于ViewGroup中

用来判断是否拦截某个事件，如果当前View拦截某个事件，那么在同一个事件序列中，次方法将不会被再次调用，返回结果表示是否拦截某个事件。

*	**public boolean onTouchEvent(MotionEvent event)**

在dispatchTouchEvent方法中调用，用来处理点击事件，返回结果表示是否消耗当前事件，如果不消耗，则在同一个事件序列中，当前View无法再次接收到事件

上述三个方法到底有什么区别呢？它们是什么关系呢？下面通过一段伪代码来表示：

```Java
public boolean dispatchTouchEvent(MotionEvent event) {

	public consume = false;

	if(onInterceptTouchEvent(ev)) {

		consume = onTouchEvent(ev);

	} else {

		consume = chiled.dispatchTouchEvent(ev);

	}

	return consume;

}
```

通过上面的伪代码，我们可以大致了解点击事件的传递规则：对已一个跟ViewGroup来说，点击事件产生以后，首先会传递给它，这是它的dispatchTouchEvent就会被调用，如果这个ViewGroup的onInterceptTouchEvent方法返回true就表示它要拦截当前事件，接着事件就会交给这个ViewGroup的onTouchEvent方法就会被调用；如果这个ViewGroup的onInterceptTouchEvent方法返回false就表示它不拦截当前事件，这时当前事件就会继续传递给它的子元素，接着子元素的dispatchTouchEvent方法就会被调用，如此反复直到事件被最终处理。

当一个View需要处理事件时，如果它设置了OnTouchListener，那么OnTouchListener中的OnTouch方法就会被回调，事件如何处理还要看onTouch的返回值，如果返回值为false，则当前View的OnTouchEvent方法会被调用；如果返回为true，则当前View的OnTouchEvent方法不会被调用。由此可见，给View设置的OnTouchListener，其优先级比onTouchEvent要高。在OnTouchEvent方法中，如果当前设置的有OnClickListener，那么它的onClick方法会被调用。可以看出，平时使用的OnClickListener其优先级最低，即处于事件的尾端。

当一个点击事件产生后，它的传递过程遵循如下顺序：Activity > Window > View，即事件总是先传递到Activity。Activity再传递给Window，最后Window再传递给顶级View，顶级View接收到事件后，就会按照事件分发机制去分发事件。

考虑一种情况，如果一个View的OnTouchEvent都返回false，那么它的父容器的OnTouchEvent将会被调用，依次类推，如果所有的子元素都不处理这个事件，那么这个事件将会最终传递给Activity处理，即Activity的OnTouchEvent将会被调用。我们可以用实际生活的例子来描述：假如点击事件是一个难题，这个难题最终被上级领导分给了一个程序员去处理（这是事件分发过程），结果这个程序员搞不定（OnTouchEvent返回false），现在该怎么办呢？程序员只能交给水平更高的程序员去解决（上级的OnTouchEvent被调用），如果上级再搞不定，那只能交给上级的上级去解决，这样就将难题一层层的向上抛，这是公司内部一种很常见的处理问题的过程。

关于事件传递的机制，这里给出一些结论，根据这些揭露可以更好地理解整个传递机制：

*	（1）同一个事件序列是指从手指接触屏幕的那一刻起，到手指离开屏幕的那一刻结束。在这个过程中产生了一系列事件，这个时间序列以down事件开始，中间含有数量不定的move事件，最终以up事件结束。

*	（2）正常情况下，一个事件序列只能被一个View拦截并且消耗。这一条的原因可以参考（3），因为一旦一个元素拦截了某此事件，那么同一个事件序列内的所有事件都会直接交给它处理，因此同一个事件序列中的事件不能分别由两个View同时处理，但是通过特殊手段可以做到，比如一个View将本该自己处理的事件通过OnTouchEvent强行传递给其他View处理。

*	（3）某个View一旦决定拦截，那么这个事件的序列的搜只能由它来处理（如果事件序列能够传递给它的话），并且它的onInterceptTouchEvent方法不会再被调用。就是说当一个View决定拦截一个事件后，那么系统会把同一个事件序列的其他方法都直接交给它来处理，因此就不用再调用这个View的OnInterceptTouchEvent其询问它是否要拦截了。

*	（4）某个View一旦开始处理事件，如果它不消耗ACTION_DOWN事件（onTouchEvent返回了false），那么同一个事件序列中的其他事件都不会再交给它来处理。并且事件将重新交由它的父元素去处理，即父元素的OnTouchEvent会被调用。意思就是事件一旦交给一个View处理，那么它就必须消耗掉，否则同一事件序列中剩下的事件就不再交给它来处理了。

*	（5）如果View不消耗除ACTION_DOWN以外的其他事件，那么这个点击事件将会消失，此时父元素的OnTouchEvent并不会被调用，并且当前View可以持续受到后续的事件，最终这些消失的点击事件会传递给Activity处理。

*	（6）ViewGroup默认不拦截任何事件。Android源码中ViewGroup的OnInterceptTouchEvent方法默认返回false

*	（7）View没有OnInterceptTouchEvent方法，一旦有点击事件传递给它，那么它的OnTouchEvent方法就会被调用

*	（8）View的OnTouchEvent默认都会消耗事件（返回true），除非它是不可点击的（clickable和longClickable同时为false）。View的longClickable属性默认都为false，clickable属性要分情况，比如Button的clickable默认为true，而TextView的clickable属性默认为false。

*	（9）View的enable属性不影响OnTouchEvent的默认返回值，哪怕一个View是disable状态的，只要它的clickable和longClickable有一个为true，那么它的OnTouchEvent就返回true。

*	（10）onClick会发生的前提是当前View是可点击的，并且收到了down和up事件

*	（11）事件传递过程是由外向内的，即事件总是先传递给父元素，然后再有父元素分发给子View，通过requestDisallowInterceptTouchEvent方法可以在子元素中干预父元素的事件分发过程，但是ACTION_DOWN事件除外。


# 事件分发的源码解析

## Activity对点击事件的分发过程

点击事件用MotionEvent来表示，当一个点击操作发生时，事件最先传递给当前Activity，由Activity的dispatchTouchEvent来进行事件分发，具体的工作是由Activity内部的Window来完成的。Window会将事件传递给decor view，decor view一般就是当前界面的底层容器（即setContentView所设置的View的父容器），通过Activity.getWindow。getDecorView()可以获得。先从Activity的dispatchTouchEvent开始分析。

源码：Activity#dispatchTouchEvent

```Java
public boolean dispatchTouchEvent(MotionEvent ev) {
	if (ev.getAction() == MotionEvent.ACTION_DOWN) {
		onUserInteraction();
	}
	if (getWindow().superDispatchTouchEvent(ev)) {
		return true;
	}
	return onTouchEvent(ev);
}
```

分析上面的代码。首先事件交给Activity所附属的Window进行分发，如果返回true，整个事件就循环结束了，返回false以为着事件没人处理，所有View的onTouchEvent都返回了false，那么Activity的onTouchEvent就会被调用

接下里看Window是如何将事件传递给ViewGroup的。Window是是个抽象类，而Window的superDispatchTouchEvent方法也是个抽象方法，因此我们必须找到Window的实现类才行

源码：Window#superDispatchTouchEvent

```Java
public abstract boolean superDispatchTouchEvent(MotionEvent event);
```

到底Window的实现类是什么呢？其实是PhoneWindow，Window的唯一实现是android.policy.Window，接下来看一下PhoneWindow是如何处理点击事件的，

源码：PhoneWindow#superDispatchTouchEvent

```Java
private DecorView mDecor;

public boolean superDispatchTouchEvent(MotionEvent event) {
	return mDecor.superDispatchTouchEvent(event);
}
```

PhoneWindow将事件直接传递给了DecorView，这个DecorView是什么呢？

```Java
private final class DecorView extends FrameLayout implements RootViewSurfaceTaker

@Override
public final View getDecorView() {
	if (mDecor == null) {
		installDecor();
	}
	return mDecor;
}
```

可以看出，这个mDecor就是通过getWindow().getDecor()返回的View，而我们通过setContentView设置的View是它的一个子View。目前事件传递到了DecorView这里，由于DecorView继承自FrameLayout且是父View，所有最终事件会传递到View。换句话说，事件肯定会传递到View，不然应用如何响应点击事件呢？重点是事件到了View以后该如何传递。从这里开始，事件已经传递到顶级View了，即在Activity中通过setContentView所设置的View，另外顶级View也叫根View，顶级View一般来说都是ViewGroup。


## 顶级View对点击事件的分发过程

点击事件达到顶级View以后，会调用ViewGroup的dispatchTouchEvent方法，然后逻辑是这样的：如果顶级ViewGroup拦截事件即OnInterceptTouchEvent返回true，则事件由ViewGroup处理，这时如果ViewGroup的OnTouchListener被设置，则onTouch被调用，否则onTouchEvent被调用，也就是说两者都设置的话，onTouch会屏蔽掉OnTouchEvent。在OnTouchEvent中如果设置了OnClickListener，则OnClick会被调用。如果顶级ViewGroup不拦截事件，则事件会传递给它所在的点击事件链上的子View，这时子View的dispatchTouchEvent会被调用。到此，事件已经从顶级View传递给下一层View，接下来的传递过程和顶级View是一致的，如此循环完成整个事件的分发。

首先看ViewGroup对点击事件的分发过程，其主要实现在ViewGroup的dispatchTouchEvent方法中，这个方法较长。先看下面一段，它描述的是View是否拦截点击事件这个逻辑

```Java
// Check for interception.
final boolean intercepted;
if (actionMasked == MotionEvent.ACTION_DOWN
		|| mFirstTouchTarget != null) {
	final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
	if (!disallowIntercept) {
		intercepted = onInterceptTouchEvent(ev);
		ev.setAction(action); // restore action in case it was changed
	} else {
		intercepted = false;
	}
} else {
	// There are no touch targets and this action is not an initial down
	// so this view group continues to intercept touches.
	intercepted = true;
}
````

从上面的代码可以看出，ViewGroup在如下两种情况下会判断是否要拦截当前事件：事件类为ACTION_DOWN或者mFirstTouchTarget ！= null，mFirstTouchTarget ！= null是什么意思了？当ViewGroup不拦截事件并将事件交由子元素处理时mFirstTouchTarget ！= null 。反过来，一旦事件由当前ViewGroup拦截时，mFirstTouchTarget ！= null就不成立，那么当ACTION_DWON和ACTION_UP事件到来时，由于(actionMasked == MotionEvent.ACTION_DOWN|| mFirstTouchTarget != null)为false，将导致ViewGroup的OnInterceptTouchEvent不会再调用，并且同一序列中的其他事件都会默认交由给它处理。

这里有一种特使情况，那就是FLAG_DISALLOW_INTERCEPT标记位，这个标记位是通过requestDisallowInterceptTouchEvent设置的，一般用于子View中。FLAG_DISALLOW_INTERCEPT一旦设置后，ViewGroup将无法拦截除了ACTION_DOWN以外的其他事件。为什么说是除了ACTION_DOWN意外的其他事件了？这是因为ViewGroup在分发事件时，如果是ACTION_DOWN就会重置FLAG_DISALLOW_INTERCEPT这个标记位，将导致子View中设置的这个标记位无效。因此，当面对ACTION_DOWN事件时，ViewGroup总是会调用自己的OnInterceptTouchEvent方法来询问自己是否要拦截事件。在下面的代码中，ViewGroup会在ACTION_DOWN事件到来时做重置状态的操作，而在resetTouchState方法中会对FLAG_DISALLOW_INTERCEPT进行重置，因此子View调用requestDisallowInterceptTouchEvent方法并不能影响ViewGroup对ACTION_DOWN的处理

```Java
if (actionMasked == MotionEvent.ACTION_DOWN) {
	// Throw away all previous state when starting a new touch gesture.
	// The framework may have dropped the up or cancel event for the previous gesture
	// due to an app switch, ANR, or some other state change.
	cancelAndClearTouchTargets(ev);
	resetTouchState();
}
```

从上面的源码分析，我们可以得出结论：当ViewGroup决定拦截事件后，那么后续的点击事件将会默认交给它处理，并且不再调用它的OnInterceptTouchEvent方法。FLAG_DISALLOW_INTERCEPT这个标志的作用是ViewGroup不再拦截事件，当前前提是ViewGroup不拦截ACTION_DOWN事件。那么这段分析有什么价值呢？总结起来有两点：第一，OnInterceptTouchEvent不是每次事件都会被调用的，如果我们想提前处理所有的点击事件，要选择dispatchTouchEvent方法，只有这个方法能确保每次都会调用，当然前提是事件能够传递到当前的ViewGroup；第二点：FLAG_DISALLOW_INTERCEPT标记位的作用给我们提供了一个思路，当面对滑动冲突时，我们可以是不是考虑用这种方法去解决问题？

接着再看当ViewGroup不拦截事件的时候，事件会向下分发交由它的子View进行处理，这段源码如下：

```Java
final int childrenCount = mChildrenCount;
if (newTouchTarget == null && childrenCount != 0) {
	final float x = ev.getX(actionIndex);
	final float y = ev.getY(actionIndex);
	// Find a child that can receive the event.
	// Scan children from front to back.
	final ArrayList<View> preorderedList = buildOrderedChildList();
	final boolean customOrder = preorderedList == null
			&& isChildrenDrawingOrderEnabled();
	final View[] children = mChildren;
	for (int i = childrenCount - 1; i >= 0; i--) {
		final int childIndex = customOrder
				? getChildDrawingOrder(childrenCount, i) : i;
		final View child = (preorderedList == null)
				? children[childIndex] : preorderedList.get(childIndex);

		// If there is a view that has accessibility focus we want it
		// to get the event first and if not handled we will perform a
		// normal dispatch. We may do a double iteration but this is
		// safer given the timeframe.
		if (childWithAccessibilityFocus != null) {
			if (childWithAccessibilityFocus != child) {
				continue;
			}
			childWithAccessibilityFocus = null;
			i = childrenCount - 1;
		}

		if (!canViewReceivePointerEvents(child)
				|| !isTransformedTouchPointInView(x, y, child, null)) {
			ev.setTargetAccessibilityFocus(false);
			continue;
		}

		newTouchTarget = getTouchTarget(child);
		if (newTouchTarget != null) {
			// Child is already receiving touch within its bounds.
			// Give it the new pointer in addition to the ones it is handling.
			newTouchTarget.pointerIdBits |= idBitsToAssign;
			break;
		}

		resetCancelNextUpFlag(child);
		if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)) {
			// Child wants to receive touch within its bounds.
			mLastTouchDownTime = ev.getDownTime();
			if (preorderedList != null) {
				// childIndex points into presorted list, find original index
				for (int j = 0; j < childrenCount; j++) {
					if (children[childIndex] == mChildren[j]) {
						mLastTouchDownIndex = j;
						break;
					}
				}
			} else {
				mLastTouchDownIndex = childIndex;
			}
		}
		mLastTouchDownX = ev.getX();
		mLastTouchDownY = ev.getY();
		newTouchTarget = addTouchTarget(child, idBitsToAssign);
		alreadyDispatchedToNewTouchTarget = true;
		break;
	}
}
```

首先遍历ViewGroup的所有子元素，然后判断子元素是否能够接收到点击事件。是否能够接收到点击事件主要由两点来衡量：子元素是否在播动画和点击事件的坐标是否落在子元素的区域内。如果某个子元素满足这两个条件，那么事件就会传递给它来处理。可以看到dispatchTransformedTouchEvent实际上调用的就是子元素的dispatchTouchEvent方法，在它的内部有一段内容：而在上面的代码中child传递的不是null，因此它会直接调用子元素的dispatchTouchEvent，这样事件就交由子元素处理，从而完成了一轮事件分发。

```Java
if (child == null) {
	handled = super.dispatchTouchEvent(event);
} else {
	handled = child.dispatchTouchEvent(event);
}
```

如果子元素的dispatchTouchEvent返回true，那么，FirstTouchTarget就会被赋值同时跳出for循环，如下所示：

```Java
newTouchTarget = addTouchTarget(child, idBitsToAssign);
alreadyDispatchedToNewTouchTarget = true;
break;
```

这几行代码就完成了mFirstTouchTarget的赋值并终止对子元素的遍历。如果子元素的dispatchTouchEvent返回false，ViewGroup就会把事件分发给下一个子元素。

其实mFirstTouchTarget真正的赋值过程是在addTouchTarget内部完成的，从下面的addTouchTarget方法的内部结构可以看出，mFirstTouchTarget其实是一种单链表结构。mFirstTouchTarget是否被赋值，将直接影响到ViewGroup对事件的拦截策略，如果mFirstTouchTarget为null，那么ViewGroup就默认拦截接下来同一序列中所有的点击事件

```Java
private TouchTarget addTouchTarget(View child, int pointerIdBits) {
	TouchTarget target = TouchTarget.obtain(child, pointerIdBits);
	target.next = mFirstTouchTarget;
	mFirstTouchTarget = target;
	return target;
}
```

如果遍历所有的子元素后事件都没有合适地处理，这包含两种情况：第一种是ViewGroup没有子元素；第二种是子元素处理了点击事件，但是再dispatchTouchEvent中返回了false，这一般式因为子元素在OnTouchEvent中返回了false。在这两种情况下ViewGroup会自己处理点击事件。

```Java
// Dispatch to touch targets.
if (mFirstTouchTarget == null) {
	// No touch targets so treat this as an ordinary view.
	handled = dispatchTransformedTouchEvent(ev, canceled, null,
			TouchTarget.ALL_POINTER_IDS);
}
```

第三个参数child为null，它会调用super.dispatchTouchEvent(event)，很显然，这里就转到了View（不包括ViewGroup）的dispatchTouchEvent方法，即点击事件开始交由View来处理。

## View对点击事件的处理过程

View点击事件的处理过程简单一些，这里的View不包含ViewGroup。先看它的dispatchTouchEvent方法：

```Java
public boolean dispatchTouchEvent(MotionEvent event) {
	// If the event should be handled by accessibility focus first.
	...

	if (onFilterTouchEventForSecurity(event)) {
		//noinspection SimplifiableIfStatement
		ListenerInfo li = mListenerInfo;
		if (li != null && li.mOnTouchListener != null
				&& (mViewFlags & ENABLED_MASK) == ENABLED
				&& li.mOnTouchListener.onTouch(this, event)) {
			result = true;
		}

		if (!result && onTouchEvent(event)) {
			result = true;
		}
	}

	...

	return result;
}
```

View对点击事件的处理就比较简单了因为View是一个单独的元素，它没有子元素因此无法向下传递事件，所以它只能自己处理事件。从上面的源码可以看出View对点击事件的处理过程，首先会判断有没有设置OnTouchListener，如果OnTouchListener中的OnTouch方法返回true，那么OnTouchEvent就不会被调用，可见OnTouchListener的优先级高于OnTouchEvent。

接着再分析OnTouchEvent的实现。先看当View出于不可用状态下的点击事件的处理过程，如下所示，不可用状态下的View照样会消耗点击事件，尽管它看起来不可用。

```Java
if ((viewFlags & ENABLED_MASK) == DISABLED) {
	if (event.getAction() == MotionEvent.ACTION_UP && (mPrivateFlags & PFLAG_PRESSED) != 0) {
		setPressed(false);
	}
	// A disabled view that is clickable still consumes the touch
	// events, it just doesn't respond to them.
	return (((viewFlags & CLICKABLE) == CLICKABLE ||
			(viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE));
}
```

接着，如果View设置有代理，那么还会执行TouchDelegate的onTouchEvent方法，这个OnTouchEvent的工作机制看起来和OnTouchListener类似

```Java
if (mTouchDelegate != null) {
	if (mTouchDelegate.onTouchEvent(event)) {
		return true;
	}
}
```

下面再看一下OnTouchEvent中堆点击事件的具体处理

```Java
	  if (((viewFlags & CLICKABLE) == CLICKABLE ||
	            (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE)) {
	        switch (event.getAction()) {
	            case MotionEvent.ACTION_UP:
	                boolean prepressed = (mPrivateFlags & PFLAG_PREPRESSED) != 0;
	                if ((mPrivateFlags & PFLAG_PRESSED) != 0 || prepressed) {
	                    // take focus if we don't have it already and we should in
	                    // touch mode.
	                    boolean focusTaken = false;
	                    if (isFocusable() && isFocusableInTouchMode() && !isFocused()) {
	                        focusTaken = requestFocus();
	                    }
	
	                    if (prepressed) {
	                        // The button is being released before we actually
	                        // showed it as pressed.  Make it show the pressed
	                        // state now (before scheduling the click) to ensure
	                        // the user sees it.
	                        setPressed(true, x, y);
	                   }
	
	                    if (!mHasPerformedLongPress) {
	                        // This is a tap, so remove the longpress check
	                        removeLongPressCallback();
	
	                        // Only perform take click actions if we were in the pressed state
	                        if (!focusTaken) {
	                            // Use a Runnable and post this rather than calling
	                            // performClick directly. This lets other visual state
	                            // of the view update before click actions start.
	                            if (mPerformClick == null) {
	                                mPerformClick = new PerformClick();
	                            }
	                            if (!post(mPerformClick)) {
	                                performClick();
	                            }
	                        }
	                    }
	
	                    if (mUnsetPressedState == null) {
	                        mUnsetPressedState = new UnsetPressedState();
	                    }
	
	                    if (prepressed) {
	                        postDelayed(mUnsetPressedState,
	                                ViewConfiguration.getPressedStateDuration());
	                    } else if (!post(mUnsetPressedState)) {
	                        // If the post failed, unpress right now
	                        mUnsetPressedState.run();
	                    }
	
	                    removeTapCallback();
	                }
	                break;
	
	            case MotionEvent.ACTION_DOWN:
	                mHasPerformedLongPress = false;
	
	                if (performButtonActionOnTouchDown(event)) {
	                    break;
	                }
	
	                // Walk up the hierarchy to determine if we're inside a scrolling container.
	                boolean isInScrollingContainer = isInScrollingContainer();
	
	                // For views inside a scrolling container, delay the pressed feedback for
	                // a short period in case this is a scroll.
	                if (isInScrollingContainer) {
	                    mPrivateFlags |= PFLAG_PREPRESSED;
	                    if (mPendingCheckForTap == null) {
	                        mPendingCheckForTap = new CheckForTap();
	                    }
	                    mPendingCheckForTap.x = event.getX();
	                    mPendingCheckForTap.y = event.getY();
	                    postDelayed(mPendingCheckForTap, ViewConfiguration.getTapTimeout());
	                } else {
	                    // Not inside a scrolling container, so show the feedback right away
	                    setPressed(true, x, y);
	                    checkForLongClick(0);
	                }
	                break;
	
	            case MotionEvent.ACTION_CANCEL:
	                setPressed(false);
	                removeTapCallback();
	                removeLongPressCallback();
	                break;
	
	            case MotionEvent.ACTION_MOVE:
	                drawableHotspotChanged(x, y);
	
	                // Be lenient about moving outside of buttons
	                if (!pointInView(x, y, mTouchSlop)) {
	                    // Outside button
	                    removeTapCallback();
	                    if ((mPrivateFlags & PFLAG_PRESSED) != 0) {
	                        // Remove any future long press/tap checks
	                        removeLongPressCallback();
	
	                        setPressed(false);
	                    }
	                }
	                break;
	        }
	
	        return true;
	    }
```

只要View的Clickable和longClickable有一个为true，那么就会消耗这个事件，即OnTouchEvent返回true，不管它是不是disable状态。然后就是当ACTION_UP事件发生时会触发performClick方法，如果View设置了OnClickListener，那么performClick方法内部会调用它的onClick方法，如下所示

```Java
public boolean performClick() {
	final boolean result;
	final ListenerInfo li = mListenerInfo;
	if (li != null && li.mOnClickListener != null) {
		playSoundEffect(SoundEffectConstants.CLICK);
		li.mOnClickListener.onClick(this);
		result = true;
	} else {
		result = false;
	}

	sendAccessibilityEvent(AccessibilityEvent.TYPE_VIEW_CLICKED);
	return result;
}
```

View的LONG_CLICKABLE属性默认为false，而CLICKABLE属性是否为false和具体的View有关，确切来说是可点击的View其CLICKABLE为true，不可点击的View其CLICKABLE为false。通过setClickable和setLongClickable可以改变View的LONG_CLICKABLE、CLICKABLE属性。另外，setOnClickListener和setOnLongClickListener会自动将View的LONG_CLICKABLE、CLICKABLE属性设置为true。

```Java
public void setOnClickListener(OnClickListener l) {
	if (!isClickable()) {
		setClickable(true);
	}
	getListenerInfo().mOnClickListener = l;
}

public void setOnLongClickListener(OnLongClickListener l) {
	if (!isLongClickable()) {
		setLongClickable(true);
	}
	getListenerInfo().mOnLongClickListener = l;
}
```

# 参考文献

Android开发艺术探索

