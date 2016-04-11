# AndroidDecompiledAndFix

### 1. `setColor(-16777216)`
反编译的代码中会有很多`setColor(int)`的情况，比如`setColor(-16777216)`，这个值比较特别，能轻易的查到Android文档中对这个整数的定义：
> public static final int BLACK.  
> Added in API level 1  
> Constant Value: -16777216 ( 0xff000000). 

也就是说`setColor(-16777216)`中`-16777216 `对应的颜色是**BLACK**(0xff000000)，那么其他系统未定义成某个颜色名的值呢？

```
-16777216 对应 0xff000000
       -1 对应 0xffffffff
 0xffffff 的值 16777215
   
那么对任意的 setColor(int)中的int值，我们可以：
0xffffffff+(int)+1 或 0xffffffff-(-int+1) 

则对于 ：setColor(-16777216)
可写成 ：setColor(0xffffffff - 16777215)) 或 setColor(-16777216 + 1 + 0xffffffff))

这样，我们就不用查文档寻找特定的颜色值，也能解决任意颜色的设置。
```

> [Stackoverflow : How to set color using integer?](http://stackoverflow.com/questions/8489990/how-to-set-color-using-integer)


### 2.`MeasureSpec.makeMeasureSpec(xx, int)`

反编译的代码中`MeasureSpec.makeMeasureSpec(xx, int)`的第二个参数是个`int`类型的数，这个比较简单，直接看文档或者源码即可找到：

源码：

```java

public static class MeasureSpec {
        public static final int UNSPECIFIED = 0;
        public static final int EXACTLY = 1073741824;
        public static final int AT_MOST = -2147483648;
        ...
        }
```

文档：

```
public static final int AT_MOST
Added in API level 1
Measure specification mode: The child can be as large as it wants up to the specified size.
Constant Value: -2147483648 (0x80000000)

public static final int EXACTLY
Added in API level 1
Measure specification mode: The parent has determined an exact size for the child. The child is going to be given those bounds regardless of how big it wants to be.
Constant Value: 1073741824 (0x40000000)

public static final int UNSPECIFIED
Added in API level 1
Measure specification mode: The parent has not imposed any constraint on the child. It can be whatever size it wants.
Constant Value: 0 (0x00000000)
```

则对于：  
`MeasureSpec.makeMeasureSpec(xx, 0)`  
我们应该修改为 
`MeasureSpec.makeMeasureSpec(xx, View.MeasureSpec.UNSPECIFIED)`

其他依次类推。

### 3.`setVisibility(int)`

这个同**[2]**,看文档或者看源码：

```java
public static final int VISIBLE = 0;
public static final int INVISIBLE = 4;
public static final int GONE = 8;
```

则对于：`setVisibility(0)` ==> `setVisibility(View.VISIBLE)`

其他依次类推。

### 4.`new Runnable()...`

反编译的代码中：

```java
new Runnable() {
    final /* synthetic */ AbstractButton a;
		{
       	this.a = r1;
       }

       public final void run() {
          this.a.xxxxx();
       }
};
```

可直接去掉成员变量：

```java
new Runnable() {
       public final void run() {
          xxxxx();
       }
};

```

### 5.`new Handler()...`

同`[4]`,直接去掉成员变量:

```java
new Handler() {
            final /* synthetic */ ButtonSave a;

            {
                this.a = r1;
            }

            public final void handleMessage(Message message) {
            		this.a.xxx();
            }
        };
//修改为
new Handler() {
            public final void handleMessage(Message message) {
                xxx();
            }
        };
```

### 6.`context.getSystemService("layout_inflater")`

直接看源码即可：

```java
public static final String POWER_SERVICE = "power";
public static final String WINDOW_SERVICE = "window";
public static final String LAYOUT_INFLATER_SERVICE = "layout_inflater";
public static final String ACCOUNT_SERVICE = "account";
public static final String ACTIVITY_SERVICE = "activity";
public static final String ALARM_SERVICE = "alarm";
public static final String NOTIFICATION_SERVICE = "notification";
public static final String ACCESSIBILITY_SERVICE = "accessibility";
...
```

则`context.getSystemService("layout_inflater")` ==> `context.getSystemService(Context.LAYOUT_INFLATER_SERVICE)`

其他依次类推。

### 7.`intent.setFlags(335544320)`

先看源码：

```java
Intent implements Parcelable, Cloneable  {
    public static final int FLAG_GRANT_READ_URI_PERMISSION = 1;
    public static final int FLAG_GRANT_WRITE_URI_PERMISSION = 2;
    public static final int FLAG_FROM_BACKGROUND = 4;
    public static final int FLAG_DEBUG_LOG_RESOLUTION = 8;
    public static final int FLAG_EXCLUDE_STOPPED_PACKAGES = 16;
    public static final int FLAG_INCLUDE_STOPPED_PACKAGES = 32;
    public static final int FLAG_ACTIVITY_NO_HISTORY = 1073741824;
    public static final int FLAG_ACTIVITY_SINGLE_TOP = 536870912;
    public static final int FLAG_ACTIVITY_NEW_TASK = 268435456;
    public static final int FLAG_ACTIVITY_MULTIPLE_TASK = 134217728;
    public static final int FLAG_ACTIVITY_CLEAR_TOP = 67108864;
    public static final int FLAG_ACTIVITY_FORWARD_RESULT = 33554432;
    public static final int FLAG_ACTIVITY_PREVIOUS_IS_TOP = 16777216;
    public static final int FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS = 8388608;
    public static final int FLAG_ACTIVITY_BROUGHT_TO_FRONT = 4194304;
    public static final int FLAG_ACTIVITY_RESET_TASK_IF_NEEDED = 2097152;
    public static final int FLAG_ACTIVITY_LAUNCHED_FROM_HISTORY = 1048576;
    public static final int FLAG_ACTIVITY_CLEAR_WHEN_TASK_RESET = 524288;
    public static final int FLAG_ACTIVITY_NO_USER_ACTION = 262144;
    public static final int FLAG_ACTIVITY_REORDER_TO_FRONT = 131072;
    public static final int FLAG_ACTIVITY_NO_ANIMATION = 65536;
    public static final int FLAG_ACTIVITY_CLEAR_TASK = 32768;
    public static final int FLAG_ACTIVITY_TASK_ON_HOME = 16384;
    public static final int FLAG_RECEIVER_REGISTERED_ONLY = 1073741824;
    public static final int FLAG_RECEIVER_REPLACE_PENDING = 536870912;
    public static final int FLAG_RECEIVER_FOREGROUND = 268435456;
```

那么对于`intent.setFlags(int);` 中 `int`值是上面四种之一的话就比较简单，例如：

`intent.setFlags(536870912);` ==> `intent.setFlags(PendingIntent.FLAG_NO_CREATE);` 

但是遇到一个比较特别的：`intent.setFlags(335544320);`

源码里根本没有这样一个值啊，其实`intent.setFlags( A | B )`是可以使用`|(或运算)`的，那么：

```

10000000000000000000000000000 = 268435456
					|				|
  100000000000000000000000000 =  67108864
10100000000000000000000000000 = 335544320

即 268435456 | 67108864 = 335544320
```

从而:

`intent.setFlags(335544320);`==>

`intent.setFlags( FLAG_ACTIVITY_NEW_TASK | FLAG_ACTIVITY_CLEAR_TOP )`

或者

`intent.setFlags( FLAG_RECEIVER_FOREGROUND | FLAG_ACTIVITY_CLEAR_TOP )`

从 [Codota](https://www.codota.com) 中搜索`intent.setFlags(335544320);`看到的是第一种情况，结合`intent.setFlags()`的用法，应该也是第一种情况。

相关资料：
> http://farwmarth.com/2013/04/23/android%20反编译和代码解读/
 

PS:
> 欢迎提交更多代码！
> 你可以关注的我[Github](https://github.com/likfe)、[CSDN](http://blog.csdn.net/ys743276112)和[微博](http://weibo.com/zyansen)




