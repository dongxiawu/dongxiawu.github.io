---
title: Intent与IntentFilter
date: 2018-05-04 16:20:51
tags:
- Android
- Intent
- IntentFilter
layout:
updated:
comments: true
categories:
- Android
permalink:
---

{% asset_img Intent与IntentFilter.jpg %}

> 版权声明：本文为 *冬夏* 原创文章，可以随意转载，但请注明出处。

<!-- more -->

# 前言 #
Intent 这个类相信大家都不陌生。这个类的中文名称为 *意图* ,主要用于 Android 四大组件之间传递信息。那么这个类有什么值得注意的地方呢？还有 Intent 和 IntentFilter 之间是如何匹配的呢？

# Intent #

了解一个类最直接的方式就是查看源码，所以我们通过源码首先看看 Intent 这个类有哪些成员变量。

    private String mAction;
    private Uri mData;
    private String mType;
    private String mPackage;
    private ComponentName mComponent;
    private int mFlags;
    private ArraySet<String> mCategories;
    private Bundle mExtras;
    private Rect mSourceBounds;
    private Intent mSelector;
    private ClipData mClipData;
    private int mContentUserHint = UserHandle.USER_CURRENT;
    private String mLaunchToken;

其实从 Intent 的成员变量中我们已经可以大概知道 Intent 的作用了。
我们知道 Intent 有显式和隐式之分，其中，显式 Intent 主要设置的就是 ComponentName mComponent 这个成员变量，比如：

    Intent intent = new Intent(MainActivity.this,SecondActivity.class);
    startActivity(intent);

而隐式 Intent 主要设置的就是以下的几个成员变量

    private String mAction;
    private Uri mData;
    private String mType;
    private ArraySet<String> mCategories;

从中我们可以得出，Action、Data、Type 都最多只有一个值，而 Category 可以有多个值。

隐式 Intent 的主要使用方式为：

    Intent intent = new Intent();
    intent.setAction(Intent.ACTION_SEND);
    intent.addCategory(Intent.CATEGORY_APP_BROWSER);
    startActivity(intent);

注意，若需要同时设置Data和Type两个属性时，必须使用setDataAndType()方法，不要同时调用setData()和setType()方法，因为这两个方法设置的值会相互覆盖

    public Intent setData(Uri data) {
        mData = data;
        mType = null;
        return this;
    }

    public Intent setType(String type) {
        mData = null;
        mType = type;
        return this;
    }

# IntentFilter #

当设置完 Intent 对象后，要启动的是哪个组件呢？这就由 IntentFilter 决定了。

IntentFilter 通常在 AndroidManifest.xml 中设置，如：

    <activity android:name=".MainActivity">
      <intent-filter>
            <action android:name="android.intent.action.MAIN" />
            <category android:name="android.intent.category.LAUNCHER" />
      </intent-filter>
    </activity>

与 Intent 相同，我们先看一下 IntentFilter 的成员变量如下：


    private int mPriority;
    private int mOrder;
    private final ArrayList<String> mActions;
    private ArrayList<String> mCategories = null;
    private ArrayList<String> mDataSchemes = null;
    private ArrayList<PatternMatcher> mDataSchemeSpecificParts = null;
    private ArrayList<AuthorityEntry> mDataAuthorities = null;
    private ArrayList<PatternMatcher> mDataPaths = null;
    private ArrayList<String> mDataTypes = null;
    private boolean mHasPartialTypes = false;

从中我们可以看到，由于Action，Category，DataType 等都是链表，所以 IntentFilter 中，Action、Data、Type 都允许有多个值。

而且每一个组件都允许存在多个 IntentFilter。

至于 Intent 和 IntentFilter 之间的匹配规则为：

显式 Intent：直接通过类名查找，故不需要 IntentFilter。

隐式 Intent：它必须有一个IntentFilter 的<category>属性包含CATEGORY_DEFAULT，因为startActivity()和startActivityForResult()方法处理 隐式Intent 时候，默认的认为接受组件有一个<category>属性为CATEGORY_DEFAULT的过滤器。如果一个Activity组件不声明这样一个过滤器，它就接收不到 隐式Intent。

IntentFilter 必须包含 Intent 的 Action 属性（如果 Intent 有）

IntentFilter 必须包含 Intent 的所有 Category 属性（如果 Intent 有），同时必须有
一个 Category 的属性值为CATEGORY_DEFAULT。

每个Data属性都可以指定数据的URI结构和数据MIME类型。URI包括scheme、host、port 和path四个部分，host和port合起来也成authority（host:port）部分。

`<scheme>://<host>:<port>/<path>`

例如：

content://192.168.0.1:8080/folder/subfolder/etc


在这个URI中，scheme是content，host是192.168.0.1，port是8080，path是folder/subfolder/etc。我们平时使用的网络url就是这种格式。

匹配规则：

在URI中，每个组成部分都是可选的，但是有线性的依赖关系

如果没有scheme部分，那么host部分会被忽略

如果没有host部分，那么port部分会被忽略

如果host部分和port部分都没有，那么path部分会被忽略

当进行URI匹配时候，并不是比较全部，而是局部对比，以下是URI匹配规则。

如果一个URI仅声明了scheme部分，那么所有拥有与其相同的scheme的URI都会通过匹配，其他部分不做匹配

如果一个URI声明了scheme部分和authority部分，那么拥有与其相同scheme和authority的URI才能匹配成功，path部分不做匹配

如果一个URI所有的部分都声明了，那么只有所有部分都相同的URI才能匹配成功

注意：path部分可以使用通配符`*`，也就是path其中的一部分进行匹配。

Data匹配时候，MIME类型和URI两者都会进行匹配，匹配规则如下：

如果过滤器未声明URI和MIME类型，则只有不含URI和MIME类型的隐式Intent才能匹配成功

如果过滤器中声明URI但是未声明MIME类型（也不能从URI中分析出MIME类型），则只有URI与过滤器URI相同且不包含IME类型的隐式Intent才能匹配成功

如果过滤器声明MIME类型但是未声明URI，只有包含相同MIME类型但是不包含URI的隐式Intent才能匹配成功

如果过滤器声明了URI和MIME类型（既可以是直接设置，也可以是从URI分析出来），只有包含相同的URI和MIME类型的隐式Intent才能匹配成功
