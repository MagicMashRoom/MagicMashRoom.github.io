---
layout: post
title: 可能是第十好的Android 开源 日历 Calendar 仿小米(重大更新)
date: 2017-06-27
categories: blog
tags: [android,calendar]
description: 可能是第十好的Android 开源 日历 Calendar 仿小米(重大更新)
---

# SuperCalendar

## 简介
* 博主现在工作在一家教育公司，最近公司的产品狗扔过来一个需求，说要做一个可以周月切换的课表，可以展示用户在某一天的上课安排。接到这个任务之后我研究了很多的日历控件，并且抽出了一个calenderlib。先看一下最后的项目中的效果：

<figure class="half">
    <img src="http://osnftsiae.bkt.clouddn.com/syllabus_1.png" width="250">
    <img src="http://osnftsiae.bkt.clouddn.com/syllabus_2.png" width="250">
</figure>

* 看到本篇文章的同学估计也是实验课或者项目需求中需要一个日历表，当我接到这个需求的时候，当时脑子压根连想都没想，这么通用的控件，GitHub上一搜一大堆不是嘛。可是等到真正做起来的时候，扎心了老铁，GitHub上的大神居然异常的不给力，都是实现了基本功能，能够滑动切换月份，找实现了周月切换功能的开源库很难。终于我费尽千辛万苦找到一个能够完美切换的项目时，你周月切换之后的数据乱的一塌糊涂啊！！！
* 算了，自己撸一个！！！

## 项目链接 [SuperCalendar][Tags]

[Tags]: https://github.com/MagicMashRoom/SuperCalendar
* 如果你感觉到对你有帮助，欢迎star
* 如果你感觉对代码有疑惑，或者需要修改的地方，欢迎issue

## 主要特性
* 日历样式完全自定义，拓展性强
* 左右滑动切换上下周月，上下滑动切换周月模式
* 抽屉式周月切换效果
* 标记指定日期（marker）
* 跳转到指定日期

## 思路
![CalendarMindMap](http://osnftsiae.bkt.clouddn.com/CalendarMindMap.png )

* Calendar的绘制由CalendarRenderer完成，IDayRenderer实现自定义的日期效果，CalendarAttr中存储日历的属性。
* 首先看一下Calendar的代码,Calendar主要是初始化Renderer和Attr，然后接受View的生命周期
* 在OnDraw的时候调用Renderer的onDraw方法，在点击事件onTouchEvent触发时，调用Renderer的点击处理逻辑

```java
	private void initAttrAndRenderer() {
		calendarAttr = new CalendarAttr();
		calendarAttr.setWeekArrayType(CalendarAttr.WeekArrayType.Monday);
		calendarAttr.setCalendarType(CalendarAttr.CalendayType.MONTH);
		renderer = new CalendarRenderer(this , calendarAttr , context);
		renderer.setOnSelectDateListener(onSelectDateListener);
	}

	@Override
	protected void onDraw(Canvas canvas) {
		super.onDraw(canvas);
		renderer.draw(canvas);
	}
	private float posX = 0;
	private float posY = 0;
	/*
     * 触摸事件为了确定点击的位置日期
     */
	@Override
	public boolean onTouchEvent(MotionEvent event) {
		switch (event.getAction()) {
			case MotionEvent.ACTION_DOWN:
				posX = event.getX();
				posY = event.getY();
				break;
			case MotionEvent.ACTION_UP:
				float disX = event.getX() - posX;
				float disY = event.getY() - posY;
				if (Math.abs(disX) < touchSlop && Math.abs(disY) < touchSlop) {
					int col = (int) (posX / cellWidth);
					int row = (int) (posY / cellHeight);
					onAdapterSelectListener.cancelSelectState();
					renderer.onClickDate(col, row);
					onAdapterSelectListener.updateSelectState();
					invalidate();
				}
				break;
		}
		return true;
	}
```

* 然后看一下CalendarRenderer的代码,Renderer承担了Calendar的绘制任务，首先renderer根据种子日期seedDate填充出Calendar包含的Date数据，calendar中持有一个6*7二维数组来存放日期数据。然后在onDraw的时候通过IDayRenderer来完成对日历的绘制。当点击日期改变了日期的状态时，首先改变对应日期的状态State，然后重绘Calendar。

```java
	private void instantiateMonth() {
        int lastMonthDays = Utils.getMonthDays(seedDate.year, seedDate.month - 1);	// 上个月的天数
        int currentMonthDays = Utils.getMonthDays(seedDate.year, seedDate.month);	// 当前月的天数
        int firstDayPosition = Utils.getFirstDayWeekPosition(seedDate.year, seedDate.month , CalendarViewAdapter.weekArrayType);

        int day = 0;
        for (int row = 0; row < Const.TOTAL_ROW; row++) {
            day = fillWeek(lastMonthDays, currentMonthDays, firstDayPosition, day, 				row);
        }
    }

    private int fillWeek(int lastMonthDays, int currentMonthDays, int firstDayWeek, 				int day, int row) {
        for (int col = 0; col < Const.TOTAL_COL; col++) {
            int position = col + row * Const.TOTAL_COL;	// 单元格位置
            if (position >= firstDayWeek && position < firstDayWeek + 				currentMonthDays) {	// 本月的
                day ++;
                fillCurrentMonthDate(day, row, col);
            } else if (position < firstDayWeek) { //last month
                instantiateLastMonth(lastMonthDays, firstDayWeek, row, col, 				position);
            } else if (position >= firstDayWeek + currentMonthDays) {//next month
                instantiateNextMonth(currentMonthDays, firstDayWeek, row, col, 				position);
            }
        }
        return day;
    }
    
    public void draw(Canvas canvas) {
        for (int row = 0; row < Const.TOTAL_ROW; row++) {
			if (weeks[row] != null) {
                for (int col = 0; col < Const.TOTAL_COL; col ++) {
                    if (weeks[row].days[col] != null) {
                        dayRenderer.drawDay(canvas , weeks[row].days[col]);
                    }
                }
            }
		}
    }
    
    public void onClickDate(int col, int row) {
        if (col >= Const.TOTAL_COL || row >= Const.TOTAL_ROW)
            return;
        if (weeks[row] != null) {
            if(attr.getCalendarType() == CalendarAttr.CalendayType.MONTH) {
                if(weeks[row].days[col].getState() == State.CURRENT_MONTH){
                    weeks[row].days[col].setState(State.SELECT);
                    selectedDate = weeks[row].days[col].getDate();
                    CalendarViewAdapter.saveDate(selectedDate);
                    onSelectDateListener.onSelectDate(selectedDate);
                    seedDate = selectedDate;
                } else if (weeks[row].days[col].getState() == State.PAST_MONTH){
                    selectedDate = weeks[row].days[col].getDate();
                    CalendarViewAdapter.saveDate(selectedDate);
                    onSelectDateListener.onSelectOtherMonth(-1);
                    onSelectDateListener.onSelectDate(selectedDate);
                } else if (weeks[row].days[col].getState() == State.NEXT_MONTH){
                    selectedDate = weeks[row].days[col].getDate();
                    CalendarViewAdapter.saveDate(selectedDate);
                    onSelectDateListener.onSelectOtherMonth(1);
                    onSelectDateListener.onSelectDate(selectedDate);
                }
            } else {
                weeks[row].days[col].setState(State.SELECT);
                selectedDate = weeks[row].days[col].getDate();
                CalendarViewAdapter.saveDate(selectedDate);
                onSelectDateListener.onSelectDate(selectedDate);
                seedDate = selectedDate;
            }
        }
    }
```

* 调用Renderer的draw方法时使用dayRenderer.drawDay(canvas , weeks[row].days[col])，dayRenderer是一个接口，在lib中有一个DayView 的抽象类实现该接口。 其中的drawDay方法完成了对该天到calendar的canvas上的绘制

```java
	@Override
    public void drawDay(Canvas canvas , Day day) {
        this.day = day;
        refreshContent();
        int saveId = canvas.save();
        canvas.translate(day.getPosCol() * getMeasuredWidth(),
                day.getPosRow() * getMeasuredHeight());
        draw(canvas);
        canvas.restoreToCount(saveId);
    }
```

* 使用继承自ViewPager的MonthPager来存放calendar的view

```java
	viewPageChangeListener = new ViewPager.OnPageChangeListener() {}
	//新建viewPagerChangeListener
	@Override
    protected void onSizeChanged(int w, int h, int oldW, int oldH) {
        cellHeight = h / 6;
        super.onSizeChanged(w, h, oldW, oldH);
    }//重写onSizeChanged，获取dayView的高度
	public int getTopMovableDistance() {
        CalendarViewAdapter calendarViewAdapter = (CalendarViewAdapter) 			getAdapter();
        rowIndex = calendarViewAdapter.getPagers().get(currentPosition % 			3).getSelectedRowIndex();
        return cellHeight * rowIndex;
    }//计算周月切换时在到达选中行之前MonthPager收起的距离
    public int getRowIndex() {
        CalendarViewAdapter calendarViewAdapter = (CalendarViewAdapter) 			getAdapter();
        rowIndex = calendarViewAdapter.getPagers().get(currentPosition  % 			3).getSelectedRowIndex();
        Log.e("ldf","getRowIndex = " + rowIndex);
        return rowIndex;
    }//计算选中日期所在的行数
```
* 使用CalendarViewAdapter为MonthPager填充calendar的实例

```java
	@Override
	public void setPrimaryItem(ViewGroup container, int position, Object object) {
		super.setPrimaryItem(container, position, object);
		this.currentPosition = position;
	}

	@Override
	public Object instantiateItem(ViewGroup container, int position) {

		if(position < 2){
			return null;
		}
		Calendar calendar = calendars.get(position % calendars.size());
		if(calendarType == CalendarAttr.CalendayType.MONTH) {
			CalendarDate current = seedDate.modifyMonth(position - MonthPager.CURRENT_DAY_INDEX);
			current.setDay(1);//每月的种子日期都是1号
			calendar.showDate(current);
		} else {
			CalendarDate current = seedDate.modifyWeek(position - MonthPager.CURRENT_DAY_INDEX);
			if(weekArrayType == 1) {
				calendar.showDate(Utils.getSaturday(current));
			} else {
				calendar.showDate(Utils.getSunday(current));
			}//每周的种子日期为这一周的最后一天
			calendar.updateWeek(rowCount);
		}
        if (container.getChildCount() == calendars.size()) {
            container.removeView(calendars.get(position % 3));
        }
        if(container.getChildCount() < calendars.size()) {
            container.addView(calendar, 0);
        } else {
            container.addView(calendar, position % 3);
        }
		return calendar;
	}
```
* 日历在切换周月时切换日历中填充的数据

```java
	public void switchToMonth() {
		if(calendars != null && calendars.size() > 0 && calendarType != 			CalendarAttr.CalendayType.MONTH){
			calendarType = CalendarAttr.CalendayType.MONTH;
			MonthPager.CURRENT_DAY_INDEX = currentPosition;
			Calendar v = calendars.get(currentPosition % 3);//0
			seedDate = v.getSeedDate();

			Calendar v1 =  calendars.get(currentPosition % 3);//0
			v1.switchCalendarType(CalendarAttr.CalendayType.MONTH);
			v1.showDate(seedDate);

			Calendar v2 = calendars.get((currentPosition - 1) % 3);//2
			v2.switchCalendarType(CalendarAttr.CalendayType.MONTH);
            CalendarDate last = seedDate.modifyMonth(-1);
			last.setDay(1);
			v2.showDate(last);

			Calendar v3 = calendars.get((currentPosition + 1) % 3);//1
			v3.switchCalendarType(CalendarAttr.CalendayType.MONTH);
            CalendarDate next = seedDate.modifyMonth(1);
			next.setDay(1);
			v3.showDate(next);
		}
	}

	public void switchToWeek(int rowIndex) {
		rowCount = rowIndex;
		if(calendars != null && calendars.size() > 0 && calendarType != 			CalendarAttr.CalendayType.WEEK){
			calendarType = CalendarAttr.CalendayType.WEEK;
			MonthPager.CURRENT_DAY_INDEX = currentPosition;
			Calendar v = calendars.get(currentPosition % 3);
			seedDate = v.getSeedDate();

			rowCount = v.getSelectedRowIndex();

			Calendar v1 =  calendars.get(currentPosition % 3);
			v1.switchCalendarType(CalendarAttr.CalendayType.WEEK);
			v1.showDate(seedDate);
			v1.updateWeek(rowIndex);

			Calendar v2 = calendars.get((currentPosition - 1) % 3);
			v2.switchCalendarType(CalendarAttr.CalendayType.WEEK);
			CalendarDate last = seedDate.modifyWeek(-1);
			if(weekArrayType == 1) {
				v2.showDate(Utils.getSaturday(last));
			} else {
				v2.showDate(Utils.getSunday(last));
			}//每周的种子日期为这一周的最后一天
			v2.updateWeek(rowIndex);

			Calendar v3 = calendars.get((currentPosition + 1) % 3);
			v3.switchCalendarType(CalendarAttr.CalendayType.WEEK);
			CalendarDate next = seedDate.modifyWeek(1);
			if(weekArrayType == 1) {
				v3.showDate(Utils.getSaturday(next));
			} else {
				v3.showDate(Utils.getSunday(next));
			}//每周的种子日期为这一周的最后一天
			v3.updateWeek(rowIndex);
		}
	}
```
* 使用CoordinateLayout的特性来做周月模式切换
* 1.RecyclerViewBehavior
```java
	@Override
    public boolean onStartNestedScroll(CoordinatorLayout coordinatorLayout, RecyclerView child,
                                       View directTargetChild, View target, int nestedScrollAxes) {
        LinearLayoutManager linearLayoutManager = (LinearLayoutManager) child.getLayoutManager();
        if(linearLayoutManager.findFirstCompletelyVisibleItemPosition() > 0) {
            return false;
        }

        boolean isVertical = (nestedScrollAxes & ViewCompat.SCROLL_AXIS_VERTICAL) != 0;
        int firstRowVerticalPosition =
                (child == null || child.getChildCount() == 0) ? 0 : child.getChildAt(0).getTop();
        boolean recycleviewTopStatus = firstRowVerticalPosition >= 0;
        return isVertical && (recycleviewTopStatus || !Utils.isScrollToBottom()) && child == directTargetChild;
    }

    @Override
    public void onNestedPreScroll(CoordinatorLayout coordinatorLayout, RecyclerView 			child,
                                  View target, int dx, int dy, int[] consumed) {
        super.onNestedPreScroll(coordinatorLayout, child, target, dx, dy, consumed);
        if (child.getTop() <= initOffset && child.getTop() >= minOffset) {
            consumed[1] = Utils.scroll(child, dy, minOffset, initOffset);
            saveTop(child.getTop());
        }
    }

    @Override
    public void onStopNestedScroll(final CoordinatorLayout parent, final 		RecyclerView child, View target) {
        Log.e("ldf","onStopNestedScroll");
        super.onStopNestedScroll(parent, child, target);
        if (!Utils.isScrollToBottom()) {
            if (initOffset - Utils.loadTop() > Utils.getTouchSlop(context)){
                scrollTo(parent, child, minOffset, 200);
            } else {
                scrollTo(parent, child, initOffset, 80);
            }
        } else {
            if (Utils.loadTop() - minOffset > Utils.getTouchSlop(context)){
                scrollTo(parent, child, initOffset, 200);
            } else {
                scrollTo(parent, child, minOffset, 80);
            }
        }
    }
```
* (2)MonthPagerBehavior 当recyclerView滑动式，MonthPager做相应的变化。

```java
@Override
    public boolean onDependentViewChanged(CoordinatorLayout parent, MonthPager child, View dependency) {
        Log.e("ldf","onDependentViewChanged");
        CalendarViewAdapter calendarViewAdapter = (CalendarViewAdapter) child.getAdapter();
        if (dependentViewTop != -1) {
            int dy = dependency.getTop() - dependentViewTop;    //dependency对其依赖的view(本例依赖的view是RecycleView)

            int top = child.getTop();

            if( dy > touchSlop){
                calendarViewAdapter.switchToMonth();
            } else if(dy < - touchSlop){
                calendarViewAdapter.switchToWeek(child.getRowIndex());
            }

            if (dy > -top){
                dy = -top;
            }

            if (dy < -top - child.getTopMovableDistance()){
                dy = -top - child.getTopMovableDistance();
            }

            child.offsetTopAndBottom(dy);
        } else {
            initRecyclerViewTop = dependency.getTop();
        }

        dependentViewTop = dependency.getTop();
        top = child.getTop();

        if((initRecyclerViewTop - dependentViewTop) >= child.getCellHeight()) {
            Utils.setScrollToBottom(false);
            calendarViewAdapter.switchToWeek(child.getRowIndex());
            initRecyclerViewTop = dependentViewTop;
        }
        if((dependentViewTop - initRecyclerViewTop) >= child.getCellHeight()) {
            Utils.setScrollToBottom(true);
            calendarViewAdapter.switchToMonth();
            initRecyclerViewTop = dependentViewTop;
        }

        return true;
        // TODO: 16/12/8 dy为负时表示向上滑动，dy为正时表示向下滑动，dy为零时表示滑动停止
    }
```

* 使用IDayRender来实现自定义的日历效果

DayView实现IDayRenderer，我们新建一个CustomDayView继承自DayView，在里面作自定义的显示
 ```java
 public CustomDayView(Context context, int layoutResource) {
        super(context, layoutResource);
        dateTv = (TextView) findViewById(R.id.date);
        marker = (ImageView) findViewById(R.id.maker);
        selectedBackground = findViewById(R.id.selected_background);
        todayBackground = findViewById(R.id.today_background);
    }

    @Override
    public void refreshContent() {
        renderToday(day.getDate());
        renderSelect(day.getState());
        renderMarker(day.getDate(), day.getState());
        super.refreshContent();
    }
 ```


## 使用方法

#### XML布局
* 新建XML布局

RecyclerView的layout_behavior为com.ldf.calendar.behavior.RecyclerViewBehavior

```xml
 <android.support.design.widget.CoordinatorLayout
        android:id="@+id/content"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_weight="1">

        <com.ldf.calendar.view.MonthPager
            android:id="@+id/calendar_view"
            android:layout_width="match_parent"
            android:layout_height="300dp"
            android:background="#fff">
        </com.ldf.calendar.view.MonthPager>

        <android.support.v7.widget.RecyclerView
            android:id="@+id/list"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            app:layout_behavior="com.ldf.calendar.behavior.RecyclerViewBehavior"
            android:background="#c2c2c2"
            android:layout_gravity="bottom"/>

    </android.support.design.widget.CoordinatorLayout>
    
```
#### 自定义日历样式
* 新建CustomDayView继承自DayView并重写refreshContent 和 copy 两个方法

```java
	@Override
    public void refreshContent() {
        //你的代码 你可以在这里定义你的显示规则
        super.refreshContent();
    }

    @Override
    public IDayRenderer copy() {
        return new CustomDayView(context , layoutResource);
    }
```

* 新建CustomDayView实例，并作为参数构建CalendarViewAdapter


```java
	CustomDayView customDayView = new CustomDayView(
        	context , R.layout.custom_day);
	calendarAdapter = new CalendarViewAdapter(
                context ,
                onSelectDateListener ,
                Calendar.MONTH_TYPE ,
                customDayView);
```
#### 初始化View

* 目前来看 相比于Dialog选择日历 我的控件更适合于Activity/Fragment在Activity的`onCreate`   或者Fragment的`onCreateView`  你需要实现这两个方法来启动日历并装填进数据

```java
@Override
   protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_syllabus);
        initCalendarView();
    }
    
    private void initCalendarView() {
        initListener();
        CustomDayView customDayView = new CustomDayView(
        	context , R.layout.custom_day);
		calendarAdapter = new CalendarViewAdapter(
                context ,
                onSelectDateListener ,
                Calendar.MONTH_TYPE ,
                customDayView);
        initMarkData();
        initMonthPager();
    } 
```

使用此方法回调日历点击事件
```java
private void initListener() {
        onSelectDateListener = new OnSelectDateListener() {
            @Override
            public void onSelectDate(CalendarDate date) {
                //your code
            }

            @Override
            public void onSelectOtherMonth(int offset) {
                //偏移量 -1表示上一个月 ， 1表示下一个月
                monthPager.selectOtherMonth(offset);
            }
        };
    }
```
 
 使用此方法初始化日历标记数据
```java
private void initMarkData() {
       HashMap markData = new HashMap<>();
        //1表示红点，0表示灰点
       markData.put("2017-8-9" , "1");
       markData.put("2017-7-9" , "0");
       markData.put("2017-6-9" , "1");
       markData.put("2017-6-10" , "0");
       calendarAdapter.setMarkData(markData);
   }
```
 使用此方法给MonthPager添加上相关监听
```java
monthPager.addOnPageChangeListener(new MonthPager.OnPageChangeListener() {
            @Override
            public void onPageScrolled(int position, float positionOffset, int positionOffsetPixels) {
            }

            @Override
            public void onPageSelected(int position) {
                mCurrentPage = position;
                currentCalendars = calendarAdapter.getAllItems();
                if(currentCalendars.get(position % currentCalendars.size()) instanceof Calendar){
                    //you code
                }
            }

            @Override
            public void onPageScrollStateChanged(int state) {
            }
        });
```

重写onWindowFocusChanged方法，使用此方法得知calendar和day的尺寸

```java
	@Override
    public void onWindowFocusChanged(boolean hasFocus) {
        super.onWindowFocusChanged(hasFocus);
        if(hasFocus && !initiated) {
            CalendarDate today = new CalendarDate();
        	calendarAdapter.notifyDataChanged(today);
            initiated = true;
        }
    }
```

* 大功告成，如果还不清晰，请下载DEMO

## Download
--------
Gradle:
Step 1. Add it in your root build.gradle at the end of repositories:
```groovy
allprojects {
    repositories {
	...
	maven { url 'https://www.jitpack.io' }
    }
}
```
Step 2. Add the dependency

```groovy
	dependencies {
	        compile 'com.github.MagicMashRoom:SuperCalendar:v1.3.1'
	}

```
[![](https://www.jitpack.io/v/MagicMashRoom/SuperCalendar.svg)](https://www.jitpack.io/#MagicMashRoom/SuperCalendar)

## Licence

      Copyright 2017 MagicMashRoom, Inc.