### 一、什么是Fragment ###

Fragment是Android3.0以后引入的，起初它是适配大屏幕，更灵活地展现UI所设计的，Fragment是比Activity更轻量化，更节省内存的一个东西，所以有的面试官会把它当做Android的第五大组件。

### 二、Fragment加载方式 ###

**1. 通过FragmentManager加载到指定的布局里面**

	FragmentManager manager = getSupportFragmentManager();
	FragmentTransaction transaction = manager.beginTransaction();
	
	MyFragment fragment = MyFragment.newInstance();
	transaction.add(R.id.container, fragment);
	transaction.addToBackStack(null);
	
	transaction.commit();

**2. 添加Fragment到Activity的布局文件里面**

	<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
	    xmlns:tools="http://schemas.android.com/tools"
	    android:id="@+id/container"
	    android:layout_width="match_parent"
	    android:layout_height="match_parent"
	    tools:context=".MainActivity">
	
	    <fragment
	        android:id="@+id/fragment"
	        android:name="com.test.MyFragment"
	        android:layout_width="match_parent"
	        android:layout_height="match_parent" />
	
	</android.support.constraint.ConstraintLayout>

