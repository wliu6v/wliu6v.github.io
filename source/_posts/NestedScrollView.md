title: NestedScrollView
date: 2015-04-06 12:00:45
tags: [Android]

---

##	场景描述

参照 [之前的内容 : ListView in ScrollView](http://wliu6v.github.io/2014/ListView-in-ScrollView)

<!-- more -->

![Nested Scroll 示意图](/img/scroll_1.png)

其中，Header 需要伴随下面 Content 的滚动而离开屏幕，但 Tab 滚动到屏幕顶部后会固定在顶部。我们可以通过点击 Tab 切换不同的 ListView（或 GridView）。

##	实现思路

对于该种场景，首先我们不能将 Header 部分作为 ListView 的 HeaderView。因为若如此做，那么点击 Tab 切换 ListView 时，Header 部分需要随之发生变化，这就给 Header 的管理带来了麻烦。

然后是之前我使用的思路：用一个 ScrollView 在外面嵌套着 ListView，同时根据内外层的 ListView 与 ScrollView 各自的滚动位置分发 Touch 事件。后来在使用过程中，发现此种方式会引入很多 Bug，比如在横竖屏切换或者进入 Activity 销毁并重建后都有可能导致滚动位置发生异常。因此需要考虑新的方式。

后来考虑，可以监听 ListView 的滚动事件，并通过代码使 Header 与 Tab 移动相同的距离，直至 Header 从屏幕顶端滑出而 Tab 恰好停在顶端为止。大致分为下列几步：

1. 将 Header 与 Tab 以 `layout_alignTop` 的方式叠放在 ListView 上，而不是以 `layout_above` 的方式。
2. 为 ListView 添加一个空白的 HeaderView 作为占位符，其高度与 Header 相同。
3. 在 ListView 添加 OnScrollListener，在其 onScroll 事件中根据当前滚动的位置改变 Header 和 Tab 的位置。（通过 `view.setTranslationY(floatY)` 方法实现）。
4. 因为 Tab 需要居顶，因此 Header 滚动到了适当的高度后就不应该再继续滚动，而是停在如下图所示的位置：

![Nested Scroll 示意图2](/img/scroll_2.png)

##	代码实现

代码分为以下 3 个部分：

1. 接口 ScrollTabHolder
2. 外层 Activity 与其布局
3. 内层 Fragment 与其布局

###	接口 ScrollTabHolder


	public interface ScrollTabHolder {
	
		void adjustScroll(int scrollHeight);
	
		void onScroll(AbsListView view, int firstVisibleItem, int visibleItemCount, int totalItemCount, int pagePosition);
	}


### 外层 Activity 布局

其中， pager 作为 Fragment 的容器，header_layout 包含了上文所述的 Header 与 Tab 这两部分。


	<?xml version="1.0" encoding="utf-8"?>
	<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
		android:layout_width="match_parent"
		android:layout_height="match_parent" >
	
		<android.support.v4.view.ViewPager
			android:id="@+id/pager"
			android:layout_width="match_parent"
			android:layout_height="match_parent" />
	
		<LinearLayout
			android:id="@+id/header_layout"
			android:layout_width="match_parent"
			android:layout_height="wrap_content"
	        android:orientation="vertical">
	
			<View
				android:id="@+id/header"
				android:layout_width="match_parent"
				android:layout_height="250dp"
	            android:background="#FF8888"/>
	
			<com.astuetz.PagerSlidingTabStrip
				android:id="@+id/tabs"
				android:layout_width="match_parent"
				android:layout_height="48dip"
				android:layout_gravity="bottom"
				android:background="@android:color/holo_red_dark" />
		</LinearLayout>
	
	</FrameLayout>


###	外层 Activity

省略了很多内容，只保留了 scroll 相关的内容

	public class MainActivity extends ActionBarActivity implements ScrollTabHolder,
		ViewPager.OnPageChangeListener {
	
		private View mHeaderLayout;
		private View mHeader;
		
		private int mActionBarHeight;
		private int mHeaderLayoutHeight;
		private int mHeaderHeight;
	
		private int mMinHeaderTranslation;
		private TypedValue mTypedValue = new TypedValue();
	
		@Override
		protected void onCreate(Bundle savedInstanceState) {
			super.onCreate(savedInstanceState);
	
			setContentView(R.layout.activity_main);
			mHeaderLayout = findViewById(R.id.header_layout);
	
			mHeaderLayout.getViewTreeObserver().addOnGlobalLayoutListener(
					new ViewTreeObserver.OnGlobalLayoutListener() {
						@Override
						public void onGlobalLayout() {
							if (mHeaderLayoutHeight == 0) {
								mHeaderLayoutHeight = mHeaderLayout.getHeight();
								mHeaderHeight = mHeader.getHeight();
								// removeOnGlobalLayoutListener should be removeGlobalOnLayoutListener when SDK_VERSION < SDK_JELLY_BEAN
								mHeaderLayout.getViewTreeObserver()
										.removeOnGlobalLayoutListener(this);
							}
						}
					});
	
			mHeader = findViewById(R.id.header);
			mPagerAdapter = new PagerAdapter(getSupportFragmentManager());
			mPagerAdapter.setTabHolderScrollingContent(this);
			mViewPager.setAdapter(mPagerAdapter);
			mPagerSlidingTabStrip.setViewPager(mViewPager);
			mPagerSlidingTabStrip.setOnPageChangeListener(this);
		}
	
		@Override
		public void onPageSelected(int position) {
			SparseArrayCompat<ScrollTabHolder> scrollTabHolders = mPagerAdapter
					.getScrollTabHolders();
			ScrollTabHolder currentHolder = scrollTabHolders.valueAt(position);
	
			currentHolder.adjustScroll((int) (mHeaderLayout.getHeight() + ViewHelper
					.getTranslationY(mHeaderLayout)));
		}
	
		@Override
		public void onScroll(AbsListView view, int firstVisibleItem,
				int visibleItemCount, int totalItemCount, int pagePosition) {
	
			mHeaderHeight = mHeader.getHeight();
			mMinHeaderTranslation = getActionBarHeight() - mHeaderHeight;
			int scrollY = getScrollY(view);
	
			mHeaderLayout.setTranslationY(Math.max(-scrollY, mMinHeaderTranslation)); // SDK < HONEYCOMB should use NineOldAnimation 
		}
	
		@Override
		public void adjustScroll(int scrollHeight) {
			// nothing
		}
	
		public int getScrollY(AbsListView view) {
			View c = view.getChildAt(0);
			if (c == null) {
				return 0;
			}
	
			int firstVisiblePosition = view.getFirstVisiblePosition();
			int top = c.getTop();
	
			int headerHeight = 0;
			if (firstVisiblePosition >= 1) {
				headerHeight = mHeaderLayoutHeight;
			}
	
			return -top + firstVisiblePosition * c.getHeight() + headerHeight;
		}
	
		@TargetApi(Build.VERSION_CODES.HONEYCOMB)
		public int getActionBarHeight() {
			if (mActionBarHeight != 0) {
				return mActionBarHeight;
			}
	
			if (Build.VERSION.SDK_INT > Build.VERSION_CODES.HONEYCOMB) {
				getTheme().resolveAttribute(android.R.attr.actionBarSize,
						mTypedValue, true);
			} else {
				getTheme()
						.resolveAttribute(R.attr.actionBarSize, mTypedValue, true);
			}
	
			mActionBarHeight = TypedValue.complexToDimensionPixelSize(
					mTypedValue.data, getResources().getDisplayMetrics());
	
			return mActionBarHeight;
		}
	
		public class PagerAdapter extends FragmentPagerAdapter {
	
			private SparseArrayCompat<ScrollTabHolder> mScrollTabHolders;
			private ScrollTabHolder mListener;
	
			public PagerAdapter(FragmentManager fm) {
				super(fm);
				mScrollTabHolders = new SparseArrayCompat<ScrollTabHolder>();
			}
	
			public void setTabHolderScrollingContent(ScrollTabHolder listener) {
				mListener = listener;
			}
	
			@Override
			public Fragment getItem(int position) {
				ScrollTabHolderFragment fragment = (ScrollTabHolderFragment) SampleListFragment
						.newInstance(position);
				mScrollTabHolders.put(position, fragment);
				if (mListener != null) {
					fragment.setScrollTabHolder(mListener);
				}
	
				return fragment;
			}
	
			public SparseArrayCompat<ScrollTabHolder> getScrollTabHolders() {
				return mScrollTabHolders;
			}
		}
	}

###	内层 Fragment

内层 Fragment 仅包含一个 ListView

	public class SampleListFragment extends ScrollTabHolderFragment implements OnScrollListener {
		private ListView mListView;
	
		@Override
		public void onActivityCreated(Bundle savedInstanceState) {
			super.onActivityCreated(savedInstanceState);
	
			mListView.setOnScrollListener(this);
	
			View placeHolderView = getActivity().getLayoutInflater().inflate(R.layout.view_header_placeholder, mListView, false);
			if (mListView.getHeaderViewsCount() > 0) {
				mListView.removeHeaderView(placeHolderView);
			}
			mListView.addHeaderView(placeHolderView);
		}
	
		@Override
		public void adjustScroll(int scrollHeight) {
			if (scrollHeight == 0 && mListView.getFirstVisiblePosition() >= 1) {
				return;
			}
	
			mListView.setSelectionFromTop(1, scrollHeight);
		}
	
		@Override
		public void onScroll(AbsListView view, int firstVisibleItem, int visibleItemCount, int totalItemCount) {
			if (mScrollTabHolder != null)
				mScrollTabHolder.onScroll(view, firstVisibleItem, visibleItemCount, totalItemCount, mPosition);
		}
	}


### 总结

1. 若 Tab 不需要置顶而是需要随 Header 一起滚动离开屏幕，则需要修改外层 Activity 中 onScroll 方法中的 mMinHeaderTranslation 与 scrollY 值。
2. 当切换 Tab 时，最好将 ListView 滚动至顶部，同时是 Header 位置复位，从而避免切换 Tab 时带来的部分位置空白的问题。
3. 如果 Header 部分的高度会动态变化，则应将外层 Activity 中的 onGlobalLayout 相关内容进行修改，去除 removeOnGlobalLayout 的相关代码，同时对 HeaderLayoutHeight 的高度进行判断，如果高度发生了变化就进入 if 代码段。
4. 需要注意添加了 HeaderView 之后会导致 ListView 的 position 发生变化，ListView 的 onItemClick 方法会受到影响。应采用 `parent.getAdapter().getItem(postion)` 或其他方式进行处理。

##	参考自

[Android-ParallaxHeaderViewPager](https://github.com/kmshack/Android-ParallaxHeaderViewPager)
