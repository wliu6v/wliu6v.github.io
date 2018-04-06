title: ListView in ScrollView
date: 2014-10-11 23:31:37
tags: [Android]

---

##	本篇内容已过时

Update 2015.04.06

发现了新的方式，实现的效果更好，代码更简练而且 bug 更少。请查阅 [NestedScrollView](http://wliu6v.github.io/2015/NestedScrollView/)

##	场景描述

对于 Scroll 嵌套 Scroll 这种情况来说，最理想的情况是避免进行这种布局设计。但有时候并非太容易做。比如下图：

![Nested Scroll 示意图](/img/scroll_1.png)

其中，Header 需要能够随着 ListView 的滚动离开屏幕，但 Tab 会留在界面上面。而且通过点击 Tab，需要可以在不同界面中进行切换。

如果不需要可以切换的 Tab 效果，那么我们可以将 Header 与 Tab 都作为 ListView 的 HeaderView 来实现。但是如果要能够实现 Tab 切换的话，显然我们不能将 Header 对于每个 Tab 中的 ListView 都作为 HeaderView。因此只能将 Tab 与下面的 ListView 视作是同级的控件，然后在外面嵌套一层 ScrollView。

<!-- more -->

##	实现思路

###	1. 展开内层的 ListView

关于这个问题，网上有一种现成的解决方案：计算整个内层 ListView 的高度，将其全部展开，然后就可以将其作为一个 LinearLayout 放在 ScrollView 中了。

这种做法的优点是处理起来非常简单，不需要考虑 Touch 事件。但有下列缺点：

1. 展开整个 ListView 并计算其高度这个操作需要对每个元素执行一次 getView 操作，比较耗时。如果只计算一个元素的高度并乘以元素数目，则在每个元素不等高时可能出现问题。
2. 展开整个 ListView 将导致 ListView 原有的重用机制无法正常起作用，当需要加载大量资源时，可能导致性能问题。
3. 对于通常的 loadMore，我们可以监测 onScroll 事件，判断其是否滚动到了最后一个元素。如果将 ListView 全部展开，则即使最底部元素没有显示在屏幕中，也会执行 loadMore。
4. 无法与 DragSoftListView 同时使用。

###	2. 根据内层与外层的滚动位置分发 touch 事件

我们可以在滚动的时候令其先滚动外层的 ScrollView，滚到底的时候再滚动 ListView。

具体思路，首先根据上右图，我们需要将 ListView 设置为整个屏幕的高度，并放置在外层的 ScrollView 中。也就是说， ScrollView 的全部高度为：TopView 的高度 + Tab 的高度 + 整个屏幕的高度（即下面的 ListView）。

当屏幕由初始状态向下滚动时，我们需要监测外层 ScrollView 滚动的位置，并与 TopView 的高度作比较。如果 TopView 还在视野内，则滚动事件由外层 ScrollView 来处理。如果 TopView 完全消失，则将滚动事件由内层的 ListView 来处理。

如果是向上滚动，则首先需要判断 ListView 是否已经被滚动到了顶端。如果否，则将滚动事件给 ListView 处理。如果 ListView 的第一个元素已经到顶，则将滚动事件交给外层的 ScrollView 处理。

部分代码如下：

外层：

	mRootScrollView.getViewTreeObserver().addOnScrollChangedListener(
					new ViewTreeObserver.OnScrollChangedListener() {
						@Override
						public void onScrollChanged() {
							if (mTabTopPosition == 0 || mTabHeight == 0) {
								setTabPositionValue();
							}
							if (mRootScrollView.getScrollY() >= (mTabTopPosition - 5)) {
								mTopTabView.setVisibility(View.VISIBLE);
								for (InnerFragment innerFragment : mFragmentList) {
									innerFragment.setOutCanDown(false);
								}
							} else if (mRootScrollView.getScrollY() <= mTabTopPosition
									+ mTabHeight) {
								mTopTabView.setVisibility(View.INVISIBLE);
								for (InnerFragment innerFragment : mFragmentList) {
									innerFragment.setOutCanDown(true);
								}
							}
						}
					});

	/**
	 * Help for controling the Top Tab's visibility when scrolling
	 */
	private void setTabPositionValue() {
		mTabHeight = mTabView.getHeight();
		mTabTopPosition = mTabView.getTop();
		mScrollViewTop = mRootScrollView.getTop();

		Handler handler = new Handler();
		handler.postDelayed(new Runnable() {
			@Override
			public void run() {
				int height = mFragmentContainer.getHeight();
				LinearLayout.LayoutParams params = (LinearLayout.LayoutParams) mFragmentContainer
						.getLayoutParams();
				params.height += mHeaderLayout.getHeight();
				params.height += height;
				mFragmentContainer.setLayoutParams(params);
			}
		}, 100);
	}


内层：

	private View.OnTouchListener mTouchListener = new View.OnTouchListener() {
			@Override
			public boolean onTouch(View v, MotionEvent event) {
				int action = event.getAction();
				switch (action) {
				case MotionEvent.ACTION_DOWN: {
					mTouchBeginY = event.getY();
					v.getParent().requestDisallowInterceptTouchEvent(
							!mOutCanDown || mInCanUp);
					break;
				}
				case MotionEvent.ACTION_UP: {
					v.getParent().requestDisallowInterceptTouchEvent(false);
					break;
				}
	
				case MotionEvent.ACTION_MOVE:
					if (event.getY() > mTouchBeginY + 4) {
						v.getParent().requestDisallowInterceptTouchEvent(mInCanUp);
					} else if (event.getY() < mTouchBeginY - 4) {
						v.getParent().requestDisallowInterceptTouchEvent(
								!mOutCanDown);
					}
					break;
				}
				v.onTouchEvent(event);
				return true;
			}
		};

	mGalleryView.setOnScrollListener(new AbsListView.OnScrollListener() {
				@Override
				public void onScrollStateChanged(AbsListView view, int scrollState) {
				}
	
				@Override
				public void onScroll(AbsListView view, int firstVisibleItem,
						int visibleItemCount, int totalItemCount) {

					if (firstVisibleItem == 0) {
						View v = mGalleryView.getChildAt(0);
						if (v != null) {
							setInnerCanUp(v.getBottom() < (v.getHeight() - 8));
						}
	
					} else {
						setInnerCanUp(true);
					}
				}
			});


###	需要注意的地方

对于 setTabPositionValue 方法，每次屏幕高度发生变化时（也就是横竖屏切换时）都需要重新调用一遍。

但是，怎么处理，我还没想好。

###	一些其他问题

如果因为分发 Touch 事件而导致多点触控导致 crash 的话，可以考虑禁用多点触控。参照 [http://blog.csdn.net/yy958836746/article/details/21536969](http://blog.csdn.net/yy958836746/article/details/21536969)。在 layout 中禁用掉多点触控并不能解决该问题。

如果有 Sliding Menu，则可以在内层的 TouchListener 的 ACTION_MOVE 中判断横向的移动，代码如下：

	if (mHasSlidingMenu && event.getX() < mTouchBeginX - 28) {
		v.getParent().requestDisallowInterceptTouchEvent(false);
		break;
	}


----

###	新问题

当内层的 ListView 的元素上有子 View 的 onClick 事件时，处理起来会有问题。需要对点击事件进行额外的判断。