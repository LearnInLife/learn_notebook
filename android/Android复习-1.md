[TOC]



## Activity

### 1.Activity的启动流程



### 2.app的启动流程



### 3.启动模式相关知识

* standard：标准启动模式（默认启动模式），每次都会启动一个新的activity实例。
* singleTop：单独使用使用这种模式时，如果**Activity实例位于当前任务栈顶**，就重用栈顶实例，而不新建，并回调该实例onNewIntent()方法，否则走新建流程。
* singleTask：这种模式启动的Activity**只会存在相应的Activity的taskAffinit任务栈中**，同一时刻系统中只会存在一个实例，已存在的实例被再次启动时，会重新唤起该实例，并清理当前Task任务栈该实例之上的所有Activity，同时回调onNewIntent()方法。
* singleInstance：这种模式启动的Activity独自占用一个Task任务栈，同一时刻系统中只会存在一个实例，已存在的实例被再次启动时，只会唤起原实例，并回调onNewIntent()方法。



#### **taskAffinity**属性

* 若activity没有设置taskAffinity属性，则activity的启动task名为应用包名。
* 若activity设置了taskAffinity属性，但启动flag必须要设置为**FLAG_ACTIVITY_NEW_TASK**，才会生效。



#### Intent.FLAG_ACTIVITY_NEW_TASK

* 非Activity启动的Activity（比如Service或者通知中启动的Activity）需要显示的设置Intent.FLAG_ACTIVITY_NEW_TASK

* singleTask及singleInstance在AMS中被预处理后，隐形的设置了Intent.FLAG_ACTIVITY_NEW_TASK

* FLAG_ACTIVITY_NEW_TASK与taskAffinity各种启动情况

  * A、B、C均默认taskAffinity值，并在task1中启动，D设置了新的taskAffinity值。若在C中使用FLAG_ACTIVITY_NEW_TASK标志启动D，会新建D，放在新的task2中。

    ![1553593394564](assets/1553593394564.png)

    

  * A、B、D均默认taskAffinity值，并在task1中启动，C设置了新的taskAffinity值，若在B中采用FLAG_ACTIVITY_NEW_TASK标志启动C，C会在新的task2中启动。

    * 在C中启动D，这时会新建D并进入task2。

    * 若在C中使用FLAG_ACTIVITY_NEW_TASK标志启动D，由于D的taskAffinity为默认值，则回去查找task1是否已经存在，则将task1栈移到最前面，并新建D放入task1中。

    ![1553594476932](assets/1553594476932.png)

    

  * A、B为默认taskAffinity值，并在task1中启动，C、D设置了新的taskAffinity值。采用FLAG_ACTIVITY_NEW_TASK标志启动C和D，C、D会在新的task2中启动。若在D中再次启动D，则会新建D并放入到task2中。

    ![1553596311386](assets/1553596311386.png)

    

  * A、B、C为默认taskAffinity值，并在task1中启动，D设置了新的taskAffinity值。采用FLAG_ACTIVITY_NEW_TASK标志启动D,D会在新的task2中启动。若在D中再次启动D，这个时候并不会再新建D，而是直接使用D，不会回调任何方法。

    ![1553596705010](assets/1553596705010.png)

* **总结：** 

  * 目标栈的确定，如果未使用FLAG_ACTIVITY_NEW_TASK标志启动，目标栈为启动它的当前栈；若使用了标志启动，目标栈与taskAffinity相关。
  * 如果新启动的Activity找不到目标Task自然会启动Task，
  * 如果目标task栈的栈根Activit的intent同将要启动的Activit相同，就不启动新Activity（不做任何操作），否则启动Activity。



#### Intent.FLAG_ACTIVITY_CLEAR_TASK

* FLAG_ACTIVITY_CLEAR_TASK属性必须同FLAG_ACTIVITY_NEW_TASK属性配合使用。

* 如果设置了FLAG_ACTIVITY_NEW_TASK|FLAG_ACTIVITY_CLEAR_TASK，若目标task已存在，则清空已存在的目标task，然后新建Activity作为根Activity。

* FLAG_ACTIVITY_CLEAR_TASK的优先级最高，基本可以无视所有的配置，包括启动模式及Intent Flag，哪怕是singleInstance也会被finish，并重建。

  ![1553653384217](assets/1553653384217.png)



#### Intent.FLAG_ACTIVITY_CLEAR_TOP

##### 只设置了FLAG_ACTIVITY_CLEAR_TOP标志

分两种情况：

* 若当前栈中存在将要新建的Activity，则将该Activity上面的所有Activity出栈，该Activity也finish掉，之后创建新的入栈。

  ![1553654696543](assets/1553654696543.png)

  

* 若当前栈中没有要新建的Activity，则新建入栈。

  ![1553654785216](assets/1553654785216.png)

  

##### FLAG_ACTIVITY_CLEAR_TOP与FLAG_ACTIVITY_SINGLE_TOP配合使用

* 当前栈中存在要新建的Activity，则将该Activity上面的所有Activity出栈，但不会finish该Activity，而是直接回调B的onNewIntent()。

![1553654944490](assets/1553654944490.png)



##### FLAG_ACTIVITY_CLEAR_TOP与FLAG_ACTIVITY_NEW_TASK配合使用

* 若将要启动的Activity的目标栈存在，则将该task移到前面。

* 若目标栈中存在要新建的Activity，则将该Activity上面的所有Activity出栈，该Activity也finish掉，之后创建新的入栈。

  ![1553656359850](assets/1553656359850.png)

  

##### FLAG_ACTIVITY_CLEAR_TOP、FLAG_ACTIVITY_NEW_TASK、FLAG_ACTIVITY_SINGLE_TOP配合使用

分两种情况：

* 如果当前栈的topActivity不是将要启动的Activity，会去目标Task中去找，并将该Activity上面的所有Activit以及自身都finish掉，之后创建新的入栈。（同FLAG_ACTIVITY_CLEAR_TOP与FLAG_ACTIVITY_NEW_TASK一样）

* 如果当前栈的topActivity是将要启动的Activity，就直接回调topActivity的onNewIntent，无论topActivity是不是在目标Task中。

  ![1553657662127](assets/1553657662127.png)



#### Intent.FLAG_ACTIVITY_SINGLE_TOP

* FLAG_ACTIVITY_SINGLE_TOP多用来做辅助作用，和singleTop启动模式一样。



**【参考】**：

[Android面试官装逼失败之：Activity的启动模式](https://www.jianshu.com/p/b3a95747ee91)



### 4.android获取文件存储路径

![1554274812164](assets/1554274812164.png)



## Service





## View

### 1.View绘制流程





### 2.View事件分发机制

* 一次事件序列包括：一次**ACTION_DOWN**、一次**ACTION_UP**、以及数量不定的**ACTION_MOVE**

  

* 事件分发过程通过**dispatchTouchEvent**、**onInterceptTouchEvent**、**onTouchEvent**三个方法共同完成

  * dispatchTouchEvent：此方法的返回结果受当前View的onTouchEvent和下级View的dispatchTouchEvent方法的影响，表示是否消耗当前时间。
  * onInterceptTouchEvent：如果当前View拦截了某个事件，那么同一个事件序列当中，此方法不会被再次调用，且交由当前View的onTouchEvent处理。
  * onTouchEvent：在dispatchTouchEvent进行调用，返回true表示事件被消费，本次事件终止，不在向下传递。返回false表示事件不消费，交给父级View的onTouchEvent处理，并且当前View无法接收到当前事件同一事件序列的后续事件。

  

* 事件发生时，最先由Activity接收，然后再一层层的向下层传递，直到找到合适的处理控件。

  * > ACTIVITY －> PHONEWINDOW －> DECORVIEW －> VIEWGROUP －> … －> VIEW

  * 如果事件传递到最后的View还是没有找到合适的View消费事件，那么事件就会向相反的方向传递，最终传递给Activity，如果最后 Activity 也没有处理，本次事件才会被抛弃.

    > ACTIVITY <－ PHONEWINDOW <－ DECORVIEW <－ VIEWGROUP <－ … <－ VIEW

    

* 事件分发流程图。

  ![1553673610572](assets/1553673610572.png)

#### View事件分发源码

```java
class ViewGroup:
    public boolean dispatchTouchEvent(MotionEvent ev) {
        ...
        final int action = ev.getAction();
        final int actionMasked = action & MotionEvent.ACTION_MASK;

        if (actionMasked == MotionEvent.ACTION_DOWN) {
            cancelAndClearTouchTargets(ev);
            //清除FLAG_DISALLOW_INTERCEPT设置并且mFirstTouchTarget 设置为null
            resetTouchState();
        }
        // Check for interception.
        final boolean intercepted;//是否拦截事件
        if (actionMasked == MotionEvent.ACTION_DOWN
                || mFirstTouchTarget != null) {
            //FLAG_DISALLOW_INTERCEPT是子View通过
            //requestDisallowInterceptTouchEvent方法进行设置的
            final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
            if (!disallowIntercept) {
                //调用onInterceptTouchEvent方法判断是否需要拦截
                intercepted = onInterceptTouchEvent(ev);
                ev.setAction(action); // restore action in case it was changed
            } else {
                intercepted = false;
            }
        } else {
            intercepted = true;
        }
        ...
    }
```

****

**结论：**

1. 当ViewGroup不拦截事件，交由子元素去处理，那么mFirstTouchTarget不为null。反之，mFirstTouchTarget=null，当MOVE、UP事件到来，不会再调用onInterceptTouchEvent方法来判断是否拦截，而且事件都交给它自身处理。

2. 如果事件由子View去处理时mFirstTouchTarget 会被赋值并指向子View

3. 子View可以通过requestDisallowInterceptTouchEvent方法干预父View的事件分发过程（**ACTION_DOWN事件除外**）。（联想：滑动冲突事件的解决，[见3](#View滑动冲突)）

   原因：ACTION_DOWN事件，拦截标志重置，mFirstTouchTarget 设置为null。DOWN事件ViewGroup都会调用自己的onInterceptTouchEvent来询问是否拦截。

   

**当ViewGroup不拦截事件，那么事件将下发给子View进行处理。**

```java
class ViewGroup:
    public boolean dispatchTouchEvent(MotionEvent ev) {
        final View[] children = mChildren;
        //对子View进行遍历
        for (int i = childrenCount - 1; i >= 0; i--) {
            final int childIndex = getAndVerifyPreorderedIndex(
                    childrenCount, i, customOrder);
            final View child = getAndVerifyPreorderedView(
                    preorderedList, children, childIndex);

            if (childWithAccessibilityFocus != null) {
                if (childWithAccessibilityFocus != child) {
                    continue;
                }
                childWithAccessibilityFocus = null;
                i = childrenCount - 1;
            }

            //判断1，View可见并且没有播放动画。2，点击事件的坐标落在View的范围内
            //如果上述两个条件有一项不满足则continue继续循环下一个View
            if (!canViewReceivePointerEvents(child)
                    || !isTransformedTouchPointInView(x, y, child, null)) {
                ev.setTargetAccessibilityFocus(false);
                continue;
            }

            newTouchTarget = getTouchTarget(child);
            //如果有子View处理即newTouchTarget 不为null则跳出循环。
            if (newTouchTarget != null) {
                newTouchTarget.pointerIdBits |= idBitsToAssign;
                break;
            }

            resetCancelNextUpFlag(child);
            //dispatchTransformedTouchEvent第三个参数child这里不为null
            //实际调用的是child的dispatchTouchEvent方法
            if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)) {
                mLastTouchDownTime = ev.getDownTime();
                if (preorderedList != null) {
                    for (int j = 0; j < childrenCount; j++) {
                        if (children[childIndex] == mChildren[j]) {
                            mLastTouchDownIndex = j;
                            break;
                        }
                    }
                } else {
                    mLastTouchDownIndex = childIndex;
                }
                mLastTouchDownX = ev.getX();
                mLastTouchDownY = ev.getY();
                //当child处理了点击事件，那么会设置mFirstTouchTarget 在addTouchTarget被赋值
                newTouchTarget = addTouchTarget(child, idBitsToAssign);
                alreadyDispatchedToNewTouchTarget = true;
                //子View处理了事件，然后就跳出了for循环
                break;
            }
        }
    }
```



**结论：**



















**【参考】**：

[一文读懂Android View事件分发机制](https://www.jianshu.com/p/238d1b753e64)



### 3.View滑动冲突



## 进程与线程





## 热修复与插件化

### 1.AndFix

#### 介绍

* AndFix支持2.3-7.0版本，ARM和X86架构，Dalvik和ART运行环境，32位和64位
* AndFix提供apkpatch工具生成*.apatch的补丁包，然后分发到客户端修复bug
* AndFix是在运行时在Native修改属性指针的方式，实现方法方法的替换，及时生效无需重启
* AndFix在生成补丁的包的时候，会添加上@MethodReplace注解，然后在运行时通过判断是否存在该注解去hook替换
* **核心思想：直接在native层进行方法的结构体信息对换，从而实现完美的方法新旧替换，从而实现热修复功能**

#### 补丁文件

![1554276393203](assets/1554276393203.png)

#### 方法分析

> 1. Initialize PatchManager,
>
> ```java
> patchManager = new PatchManager(context);
> patchManager.init(appversion);//current version
> ```
>
> 2. Load patch,
>
> ```java
> patchManager.loadPatch();
> ```
>
> 3. Add patch,
>
> ```java
> patchManager.addPatch(path)
> ```

**PatchManager.init(String appVersion)**

* 对初始化传入的版本号进行判断是否一致，如果版本不一致清除之前的补丁文件
* 如果一致则调用`addPatch()`加载本地的补丁文件



**PatchManager.addPatch(File file)**

* 将补丁文件封装为Patch对象，然后添加进一个排序patch集合中去



**PatchManager.loadPatch()**

* 对本地的patch进行遍历，获取每个patch的信息，然后调用AndFixManager的fix()方法进行逐一修复
* fix()的入参classes为patch中配置文件Patch.MF的Patch-Classes字段对应的所有类，即为要修复的类



**PatchManager.addPatch(String path)**

* 将从服务器上下发的补丁文件，拷贝到/data/data/application package/files/apatch文件夹下
* 然后封装成Patch对象，调用loadPatch(Patch patch)方法，单独对该补丁文件进行修复



**AndFixManager.fix(File file, ClassLoader classLoader, List<String> classes)**

* 验证补丁文件的签名、指纹信息等
* 调用DexFile.loadDex()方法加载补丁文件中的dex文件，返回DexFile 对象
* 创建匿名的类加载器，加载补丁文件中的类
* 遍历DexFile中的元素，然后与classes集合中的类进行比对，在Dex文件中找到需要修改的类
* 调用匿名的类加载器加载获得该类的Class对象
* 最后调用fixClass(String, ClassLoader)修复这个类



**AndFixManager.fixClass(Class<?> clazz, ClassLoader classLoader)**

* 获取该类所有的方法
* 遍历方法集合，找到存在@MethodReplace注解的方法，bug方法会在生成补丁类的时候打上此注解
* 获取注解中的clazz的值，即要修改的类的全类名
* 获取注解中的method的值，即要修改的方法名
* 最后调用replaceMethod()方法去替换原来的方法



**AndFixManager.replaceMethod(ClassLoader classLoader, String clz,String meth, Method method）**

* 从mFixedClass哈希map中获取需要修复的类，若不存在，则加载，调用AndFix.initTargetClass()初始化
* 根据反射获取原类中存在bug的方法
* 最后调用AndFix.addReplaceMethod(src, method)，传入原方法对象和修改后的方法对象
* AndFix.addReplaceMethod()中调用了native方法replaceMethod(src, dest)进行方法替换



**AndFix**

AndFix是Java层进行方法替换的核心类，在该类中提供了Native层的接口，加载了andfix.cpp，主要进行了Native层的初始化，以及目标修复类的替换工作。



**Native层**

![1554280388234](assets/1554280388234.png)

* Dalvik：调用dalvik_replaceMethod()，通过meth->nativeFunc = target->nativeFunc完成方法 的替换

* art：调用art_replaceMethod()完成方法替换，但是art由于版本不同，需要根据不同的版本号进行适配处理；

  通过smeth->declaring_class_ = dmeth->declaring_class_完成方法 的替换

  每一个java方法在art种都对应一个ArtMethod, ArtMethod记录了这个java方法的所有信息，包括所属类，访问权限、代码执行地址

  通过evn->FromReflectedMethod,可以由Method对象得到这个方法对应的ArtMethod的真正其实地址，然后就可以把它强转为ArtMethod指针，从而对其所有的成员进行修改。将指针指向替换方法的真实地址。以后调用这个方法时就会直接走到新方法的实现中了。



#### AndFix总体过程

1. 初始化patch管理器，加载补丁；
2. 检查手机是否支持，判断ART、Dalvik；
3. 进行md5，指纹的安全检查
4. 验证补丁的配置，通过patch-classes字段得到要替换的所有类
5. 通过注解从类中得到具体要替换的方法
6. 得到指向新方法和被替换目标方法的指针，将新方法指向目标方法，完成方法的替换。



#### AndFix优缺点

优势：

1. BUG修复的即时性
2. 补丁包同样采用差量技术，生成的PATCH体积小

3. 对应用无侵入，几乎无性能损耗

不足：

1. 不支持新增字段，以及修改方法，也不支持对资源的替换。

2. 由于厂商的自定义ROM，对少数机型暂不支持。兼容性差。





**【参考】**：

[Andfix热修复框架原理及源码解析](<https://blog.csdn.net/u011176685/article/details/50984885>)

[AndFix原理分析](<https://www.jianshu.com/p/b3aa96e449ec>)

[Android中热修复框架AndFix原理解析及案例使用](<https://blog.csdn.net/u011277123/article/details/53282381>)

[阿里最新热修复Sophix与QQ超级补丁和Tinker的实现与总结](<https://www.jianshu.com/p/0a31d145cad2>)



### 2.Tinker

#### Tinker中比较关键的几个类

* `PatchListener`  Patch监听者，负责执行获取到补丁包后的后续操作

*  `LoadReporter`  Load记录者，负责记录加载补丁包过程的状态信息

* `PatchReporter` Patch记录者，负责记录合成补丁过程的状态信息

*  `AbstractPatch` Patch执行者，负责执行补丁的合成

  

   其中，`PatchListener`,`LoadReporter`,`PatchReporter`均为接口，`AbstractPatch`为抽象类，在Tinker中他们都有默认的实现类，对应关系如下：

*  `PatchListener`  默认实现类为`DefaultPatchListener` 
* `LoadReporter`  默认实现类为`DefaultLoadReporter` 
*  `PatchReporter` 默认实现类为`DefaultPatchReporter` 
* `AbstractPatch` 默认实现类为`UpgradePatch`



#### 补丁的合成

补丁合成调用的方法:

```java
TinkerInstaller.onReceiveUpgradePatch(getApplicationContext(), Environment.getExternalStorageDirectory().getAbsolutePath() + "/patch_signed_7zip.apk");
```

合成流程：

> TinkerInstaller.onReceiveUpgradePatch()-->
>
> DefaultPatchListener.onPatchReceived()-->
>
> TinkerPatchService.runPatchService()-->
>
> UpgradePatch.tryPatch()

* 在onPatchReceived方法中对补丁文件路径进行校验，成功后启动IntentServic进行补丁合成，失败则交给LoadReporter加载记录者进行状态记录
* 在TinkerPatchService的onHandleIntent中会调用到increasingPriority()方法，采用Notification方式提高服务等级，进行保活
* tryPatch方法中首先进行一些安全性的检验，例如检查tinkerId，签名，md5值等等。最后调用



**【参考】**：

[Tinker源码解析系列（二）—补丁合成与加载](<https://www.jianshu.com/p/04528a26de9d>)