# Android 开发的最正确做法

翻译力求信达雅。如果有什么不准确的地方，欢迎大家提出来。有些地方进行了修改，添加了我自己的一些看法。如果需要交流之类的可以邮件我 `lichen900210#gmail.com`
这是我们从[Futurice](http://www.futurice.com) 开发者得到的经验教训. 希望大家可以通过阅读下面的tips 避免造重复的轮子（Do not repeat yourself）. 如果你对ios和winphone的开发也感兴趣可以参考 [**iOS Good Practices**](https://github.com/futurice/ios-good-practices) 和 [**Windows App Development Best Practices**](https://github.com/futurice/windows-app-development-best-practices) 这两个文档（尚未翻译）.

[![Android Arsenal](https://img.shields.io/badge/Android%20Arsenal-android--best--practices-brightgreen.svg?style=flat)](https://android-arsenal.com/details/1/1091)

## 总结

#### 使用Gradle 推荐的 项目结构方式
#### 把密码和隐秘的信息放在  gradle.properties 文件里
#### 不要自己搭建http架构, 使用 Volley  OkHttp 这种开源库
#### 使用 Jackson library to 来解析json数据 （PS：我个人比较推荐gson  国内的fastjson也不错好像是阿里巴巴的）
#### 不要使用 Guava 库 因为 *65k 方法的限制* 建议使用一些轻量的库（PS：我用过这个Guava的库，确实很方便 也很坑，处理一些大数据分组排序这种东西很爽,但是太重量级了）
#### 使用Fragment 来重新规划你的视图 （避免视图重复利用）
#### 仅仅使用activity来管理这些fragment （就是不要在activity 干太多视图等等奇葩的事，交给fragment 尽量分出去）
#### 布局xml layout 也是代码，请你重视它，合理规划它们，组织好它们
#### 尽量使用style来建立 主题，不要定义一大堆的 属性值 （还是那点，避免重复）
#### 样式文件尽量建多个，别堆在一个里面
#### colors.xml 颜色文件定义要短，不要重复, 尽量只定义色块
#### dimens.xml 数值文件也尽量不要重复，尽量弄成全局使用
#### ViewGroups 树形结构不要定义的太深 (做视图的时候不要嵌套太深，很容易出现 over draw 浪费性能 关于性能优化相关视频 [youtube](https://www.youtube.com/playlist?list=PLWz5rJ2EKKc9CBxr3BVjPTPoDPLdPIFCE) [优酷](http://v.youku.com/v_show/id_XODY3NjY5NTg0.html?from=s1.8-1-1.2))
#### 避免客户端处理网页视图，防止内存泄露
#### 使用 Robolectric 来做单元测试, 用 Robotium 来做UI方面测试
#### 用 Genymotion 作为模拟器 （其实不建议模拟器开发，不差那点钱吧）
#### 一定记得混淆  用 ProGuard or DexGuard


----------

### Android SDK

把 [Android SDK](https://developer.android.com/sdk/installing/index.html?pkg=tools) 放在主目录下面或者应用独立的一个目录. 有些开发工具会更新甚至修改这些sdk，容易出问题。（简言之，sdk单独出来，怕eclipse  android studio 影响sdk 不然重装sdk 很麻烦）. linux用户不要放在需要系统级别的文件夹，不然可能需要sudo权限。

### 编译环境

默认选择 [Gradle](http://tools.android.com/tech-docs/new-build-system). Ant 局限性大还墨迹. 用 Gradle, 很容易干这些事:

- 编译不同风格多样的app
- 简单的脚本风格任务
- 容易组织下载依赖包
- 自定义 签名方式加密方式 keystores
- 等等

Gradle 插件也是google比较推崇的 比较火的 一个新的编译系统 所以建议使用.

### 项目结构

有两个选择: 老的 Ant + Eclipse ADT 项目结构, 和新的 Gradle + Android Studio 项目结构.你需要选择新的结构方式. 如果您还在用老的，建议您调整适配的新的结构中去.

老的结构:

```
老结构
├─ assets
├─ libs
├─ res
├─ src
│  └─ com/futurice/project
├─ AndroidManifest.xml
├─ build.gradle
├─ project.properties
└─ proguard-rules.pro
```

新的结构:

```
新结构
├─ library-foobar
├─ app
│  ├─ libs
│  ├─ src
│  │  ├─ androidTest
│  │  │  └─ java
│  │  │     └─ com/futurice/project
│  │  └─ main
│  │     ├─ java
│  │     │  └─ com/futurice/project
│  │     ├─ res
│  │     └─ AndroidManifest.xml
│  ├─ build.gradle
│  └─ proguard-rules.pro
├─ build.gradle
└─ settings.gradle
```

最大的去别就是新的结构里面把 代码集合source set(`main`, `androidTest`)很清楚地分开了, 这是Gradle的理念. 举个例子, 你可以加两个新的标签代码库  'paid' 和 'free' 到 `src` 目录下 你就有了关于 付钱代码和免费代码这两个属性的app了.

有一个高级别的 `app` 文件夹把你和库项目 (比如., `library-foobar`被你主项目引用的库)区分开来  `settings.gradle` 存了 `app/build.gradle` 可以引用的库的数据.

### Gradle 配置

**通用的文件结构.** 参考 [Google's guide on Gradle for Android](http://tools.android.com/tech-docs/new-build-system/user-guide)

**小的脚本任务.** 跟 (shell, Python, Perl, etc) 这些脚本不同, 你可以在gradle里面建任务. 参考 [Gradle's documentation](http://www.gradle.org/docs/current/userguide/userguide_single.html#N10CBF) 看看具体怎么做.

**密码.** 当你弄发布版本的时候，在`build.gradle` 文件中你会需要定义 加密方式`signingConfigs` . 下面是你需要避免的事:

_别这么干_. 这个会提交代码大家全看见了.

```groovy
signingConfigs {
    release {
        storeFile file("myapp.keystore")
        storePassword "password123"
        keyAlias "thekey"
        keyPassword "password789"
    }
}
```

取而代之, 建立一个不会加入版本控制系统的 `gradle.properties` 文件 :

```
KEYSTORE_PASSWORD=password123
KEY_PASSWORD=password789
```

这个文件会被 gradle自动引用, 所以你可以这么用 `build.gradle`:

```groovy
signingConfigs {
    release {
        try {
            storeFile file("myapp.keystore")
            storePassword KEYSTORE_PASSWORD
            keyAlias "thekey"
            keyPassword KEY_PASSWORD
        }
        catch (ex) {
            throw new InvalidUserDataException("You should define KEYSTORE_PASSWORD and KEY_PASSWORD in gradle.properties.")
        }
    }
}
```

**尽量用 Maven 来管理jar包 的依赖 不要引用 jar 文件.** 单独引进来是一个单独的版本，会导致没法更新jar，用Maven很容易解决这个问题. 比如:

```groovy
dependencies {
    compile 'com.squareup.okhttp:okhttp:2.2.0'
    compile 'com.squareup.okhttp:okhttp-urlconnection:2.2.0'
}
```    

**避免 Maven 肢解开来动态引用**
不要用这种 `2.1.+` 模糊不清的动态引用，会导致使用不稳定版本编译和些细微的错误偏差之类的, 编译的时候根本不知道怎么会代码没改，东西不一样了. 用静态版本号 `2.1.1` 来建立稳定的, 可以预测，可以控制可以重复的开发环境.

### 开发工具和编辑器

**不管用什么编辑器，最好能和你的文件目录恰当匹配.** 编辑器看你自己的喜好来，但是需要注意的是你有责任让编辑器和你的编译系统，项目结构很好的结合运行.

强烈推荐 [Android Studio](https://developer.android.com/sdk/installing/studio.html), 由google开发, 和Gradle紧密结合, 默认用了新的项目结构, 还发布了稳定版, 专门适配了 Android 的开发.

如果想的话，你也可以用 [Eclipse ADT](https://developer.android.com/sdk/installing/index.html?pkg=adt) , 但还得配环境, 用的是老的结构还有ant的编译器. 夸张点你也能用 Vim, Sublime Text, or Emacs 这种纯文本编辑器. 如果非那么干，估计你要用命令行来跑 Gradle and `adb` . 如果gradle和eclipse 结合对你来说不合适, 你可以选择命了行编译, 或者整合到Android studio. 这是ADT插件过时后最好的解决方案了。

不管用什么, 请保证 Gradle 和新的项目结构和官方的编译程序方式匹配，别把编辑器的配置文件放进版本控制器了.比如 ant 的 `build.xml` 文件. 特别注意，改ant配置后，记得保证 `build.gradle` 最新并且可以工作. 最后，每个人都有自己喜好，别老逼别人换IDE.

### 库

**[Jackson](http://wiki.fasterxml.com/JacksonHome)** JSON 和对象互转用的. [Gson](https://code.google.com/p/google-gson/) 也是干这个用的, 但是发现 Jackson  性能更好自从它支持更多可选择的方式处理json: 流, 内存树模型, 传统的 JSON-POJO 数据绑定. 记住, 尽管Jackson 比GSON大, 按情况分析取舍,你可能更喜欢 GSON 轻量. 其他的方案: [Json-smart](https://code.google.com/p/json-smart/) 和 [Boon JSON](https://github.com/RichardHightower/boon/wiki/Boon-JSON-in-five-minutes)

**网络, 缓存, 和图片.** 处理请求和后端服务器有挺多不错的方案. 用 [Volley](https://android.googlesource.com/platform/frameworks/volley) 或者 [Retrofit](http://square.github.io/retrofit/). Volley 也能引用来加载缓存图片. 如果你用 Retrofit, 可以考虑配合 [Picasso](http://square.github.io/picasso/) 来加载缓存图片, 配合 [OkHttp](http://square.github.io/okhttp/) 做高效请求. 这三个Retrofit, Picasso 和 OkHttp 都是一个公司出的所以结合起来比较好. [OkHttp can also be used in connection with Volley](http://stackoverflow.com/questions/24375043/how-to-implement-android-volley-with-okhttp-2-0/24951835#24951835).

**RxJava** 是做响应式编程的, 换句话说就是处理异步任务的. 很强大而且用了 promising 理念（关于这点有兴趣的可以查下，是一种设计方式，用起来很舒服）, 不过也可能让你迷惑毕竟这东西挺特别的. 建议你用这个库搭整个项目时候考虑清楚了. 有些项目用了 RxJava, 需要帮忙可以问这些人: Timo Tuominen, Olli Salonen, Andre Medeiros, Mark Voit, Antti Lammi, Vera Izrailit, Juha Ristolainen. 我们已经写了些博客关于这个的: [[1]](http://blog.futurice.com/tech-pick-of-the-week-rx-for-net-and-rxjava-for-android), [[2]](http://blog.futurice.com/top-7-tips-for-rxjava-on-android), [[3]](https://gist.github.com/staltz/868e7e9bc2a7b8c1f754), [[4]](http://blog.futurice.com/android-development-has-its-own-swift).

如果用 Rx这个没什么经验, 可以尝试仅仅在api返回响应用一下. 或者试下在简单的界面事件处理上用下, 比如搜索框里的点击事件和字符串变化事件. 如果你打算在整个项目中用 Rx技术 记得在这些取巧的地方写Java文档 . 记住不熟悉 RxJava 的维护这个项目可能很费事. 尽力帮助他们读懂代码和 理解Rx的概念.

**[Retrolambda](https://github.com/evant/gradle-retrolambda)** 是个在Android和JDK8预览版平台用lambda表达式的库 . 使用RxJava 会让你的代码可读性很强，并且优雅富有美感. 使用它, 安装 JDK8,在Android Studio Project 结构对话框里配置你的SDK 位置 , 然后设置 `JAVA8_HOME` 和 `JAVA7_HOME` 环境变量, 然后再项目文件 build.gradle加上:

```groovy
dependencies {
    classpath 'me.tatarka:gradle-retrolambda:2.4.1'
}
```

在每个module 中的 build.gradle文件添加

```groovy
apply plugin: 'retrolambda'

android {
    compileOptions {
    sourceCompatibility JavaVersion.VERSION_1_8
    targetCompatibility JavaVersion.VERSION_1_8
}

retrolambda {
    jdk System.getenv("JAVA8_HOME")
    oldJdk System.getenv("JAVA7_HOME")
    javaVersion JavaVersion.VERSION_1_7
}
```

Android Studio 提供了代码提示用于 Java8 lambdas 表达式. 如果你是新手看看下面的东西:

- 只用一个方法的接口是非常兼容lambda 特性的  也更方便整到 更紧密的表达式中
- 碰到参数拿不准可以写一个匿名内部类 让Android Studio 帮你整合出到一个lambda中.

**注意 dex 方法限制, 避免使用太多的库.** Android 应用在打包成dex的时候有个强硬限制,引用的方法 要在65536大小之内 [[1]](https://medium.com/@rotxed/dex-skys-the-limit-no-65k-methods-is-28e6cb40cf71) [[2]](http://blog.persistent.info/2014/05/per-package-method-counts-for-androids.html) [[3]](http://jakewharton.com/play-services-is-a-monolith/). 你要是无视这个限制编译的时候可能会挂掉. 就因为这样, 尽量使用小的库, 也顺便用 [dex-method-counts](https://github.com/mihaip/dex-method-counts) 这个工具来测测那些库可以用. 特别注意不要用 Guava 库, 因为这货超过了13K.

### Activities 和 Fragments
在社区论坛里，怎么着能最好地组织Fragments 和 Activities 的android 结构 还没有共识。Square 甚至还出了 [一个库](https://github.com/square/mortar)通过传递fragment来用来建立更好的视图, 但也没推行起来，大家用的并不多.

由于Android API's 历史原因, 你可以简单的用 Fragments 作为展示视图的块. 换句话说, Fragments 更贴近UI. Activities 可以简单的 当做controllers, 他们在维护生命周期和状态的时候特别重要. 不过你可能见过泽泻角色变换: activities 可能管理UI 去了([delivering transitions between screens](https://developer.android.com/about/versions/lollipop.html)), 而 [fragments 又去当控制器去了](http://developer.android.com/guide/components/fragments.html#AddingWithoutUI). 不管怎么说，希望你们用的时候小心点，因为如果只用fragment 只用activity 和只用view都会有弊端。 下面是一些建议，你看和你的想法一致就采纳不一定非要套用:

- 避免大规模 [嵌套 fragments](https://developer.android.com/about/versions/android-4.2.html#NestedFragments) , 因为 会有这种[matryoshka bugs](http://delyan.me/android-s-matryoshka-problem/) . 仅仅当嵌套fragments 有意义的时候才弄 (比如, fragments 在水平滑动的 ViewPager里面 又在 像屏幕的fragment里面) or 除非你非常清楚自己要做什么.
- 在activity里面别扔太多代码. 记得不论在什么时候尽量让你的它只管理生命周期和一些重要的Android API 交互的事情. 最好用单个 fragment activity 代替简单的activity - 把UI的代码放在这个fragment里面. 这样的话当你放在选项卡布局页面，或者平板多fragment交互页面的时候 可以重用. 尽量避免一个activity 不加任何fragment 除非你下定了要这么干.
- 不要烂用 Android-级别的 APIs 比如特别依赖app Intent的工作机制. 这样会影响 Android 系统和其他程序, 产生bug或者延迟. 比如, 如果你用Intents 放在包之间的内部交互，当app在系统刚启动的时候，用户会感觉产生重复的二次延迟。（这点我也不是很理解，有深入了解的可以探讨下发issue 或者mail我 可能翻译的不当）

### Java 包结构

 Android 应用的java结构的用 MVC  [Model-View-Controller](http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller) 表示有点难. 在 Android中, [Fragment 和 Activity 实际上是控制器（controller）](http://www.informit.com/articles/article.aspx?p=2126865). 另一方面, 他们也有点像UI交互, 也可以说是view.（面试有时候会问mvc这块）

从这点上说, 很难下定义说fragments (或者 activities) 就是 controllers 或者 views. 所以最好还是让他们呆在自己的包里 fragment 在 `fragments` 包中.只要你遵照刚才的建议， Activities 可以放在高层的包中. 如果有多个activity 可以 组织到`activity` 的一个包中.

还有一种, 你的包结构可以是典型的 MVC。 用一个  `models` 包 放些 通过json 解析器 返回的 POJOs , 把自定义视图 通知  action bar , widgets,等等放在 `views` 包里面. Adapters 是个连接器, 放在  数据和视图之间. 然而, 他们很明显需要通过 `getView()` 方法获取view, 所以可以把 `adapters` 包放在 `views` 包里面.

有些controller 的类 全局都要用 更贴近 Android 系统. 他们可以放在  `managers` 包里面. 乱七八糟的数据处理类可以类似 "DateUtils",放在 `utils` 包里面. 和后台交互的一些类 则放在`network` 包里面.

总而言之，要贴近后台贴近用户:

```
com.futurice.project
├─ network
├─ models
├─ managers
├─ utils
├─ fragments
└─ views
   ├─ adapters
   ├─ actionbar
   ├─ widgets
   └─ notifications
```

### 资源文件

**命名.** 遵循约定俗成的下划线写法, 就像 `type_foo_bar.xml`. 比如: `fragment_contact_details.xml`, `view_primary_button.xml`, `activity_main.xml`.

**组织 layout XMLs文件.** 如果你不知道怎么排版一个layout 文件 下面这些约定俗成的东西挺适合你看看.

- 一行一个属性值，用四个空格进行缩进
- `android:id` 永远放在第一个位置
- `android:layout_****` 这些layout属性放在最上面
- `style` 属性放在下面
- 方便 排序和添加 新的属性  标签结束符 `/>` 最好单起一行.
- 考虑使用Android Studio [属性预览功能](http://tools.android.com/tips/layout-designtime-attributes)支持的方式来写些东西 别写死 `android:text`.

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    >

    <TextView
        android:id="@+id/name"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_alignParentRight="true"
        android:text="@string/name"
        style="@style/FancyText"
        />

    <include layout="@layout/reusable_part" />

</LinearLayout>
```

一个简单的规则,  `android:layout_****` 这种属性需要定义在 layout XML, 而其他属性 `android:****`放在style XML. 这规则有例外，但是大多数情况下都行得通. 这个理念就是只把布局 (positioning, margin, sizing) 和内容放在布局文件里面, 而样式啊  (colors, padding, font) 这种控制外观的放在 styles 文件里.

哪些是例外呢:

- `android:id` 这个属性很明显必须放layout文件里面
- `LinearLayout` 的`android:orientation` 放在layout 里面更合理
- `android:text` 这个需要放在layout里面因为定义了内容
- 有时候需要统一宽度 高度 `android:layout_width` `android:layout_height` 把这两个属性抽出去，但是一般来说都是放在自己的layout文件里面

**用法.** 几乎每个项目都会用到样式，因为 用样式来统一定义 复用 很正常。最起码你有个统一的字体样式, 比如说:

```xml
<style name="ContentText">
    <item name="android:textSize">@dimen/font_normal</item>
    <item name="android:textColor">@color/basic_black</item>
</style>
```

这个用在 TextViews上面:

```xml
<TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@string/price"
    style="@style/ContentText"
    />
```

没准你也想放在按钮上, 但这么做还远远不够. 进可能的把能抽的 `android:****` 这些属性都抽出来放在一个公用的style文件里面.

**把一个大的style文件分割成零碎的.** 你不需要一个单独的 `styles.xml` 文件. Android SDK 支持其他文件名引用样式,重要的是里面的 `<style>` 标签. 也就是说你可以分割成 `styles.xml`, `styles_home.xml`, `styles_item_details.xml`, `styles_forms.xml`. 不像资源文件夹名字有特殊意义你不能改，编译会出问题, 但在 `res/values` 这里面的文件名是随意的.

**`colors.xml` 是颜色样式文件.** 在 `colors.xml` 文件里面就应该只放颜色名和 RGBA 值的映射关系. 别用它来定义特指的按钮.

*别这么干:*

```xml
<resources>
    <color name="button_foreground">#FFFFFF</color>
    <color name="button_background">#2A91BD</color>
    <color name="comment_background_inactive">#5F5F5F</color>
    <color name="comment_background_active">#939393</color>
    <color name="comment_foreground">#FFFFFF</color>
    <color name="comment_foreground_important">#FF9D2F</color>
    ...
    <color name="comment_shadow">#323232</color>
```

总而言之你这么写就不好抽出来，万一别的地方要用这个颜色呢？可重复性太差。这种东西 比如按钮 评价 等等应该放在style里面 而不是 `colors.xml` 文件

取而代之这么做:

```xml
<resources>

    <!-- grayscale -->
    <color name="white"     >#FFFFFF</color>
    <color name="gray_light">#DBDBDB</color>
    <color name="gray"      >#939393</color>
    <color name="gray_dark" >#5F5F5F</color>
    <color name="black"     >#323232</color>

    <!-- basic colors -->
    <color name="green">#27D34D</color>
    <color name="blue">#2A91BD</color>
    <color name="orange">#FF9D2F</color>
    <color name="red">#FF432F</color>

</resources>
```

问好你的设计师你需要的色调块. 名字不一定非要是颜色具体的名字 比如`蓝色``绿色`. 名字可以是这种 "brand_primary"（品牌主色调）, "brand_secondary"（品牌次色调）都行. 弄好颜色格式也很容易方便你以后批量修改重构, 也能特别明确的知道到底用了多少颜色. 对于一个富有美感的UI来说，减少色彩的多样性挺有必要的.

**处理 dimens.xml 要像 colors.xml一样.** 你也需要定义一个类似 "调色板"的选择器用来定义间距啊，长度啊,就像我们定义调色板处理颜色一样. 下面是个比较好的例子:

```xml
<resources>

    <!-- font sizes -->
    <dimen name="font_larger">22sp</dimen>
    <dimen name="font_large">18sp</dimen>
    <dimen name="font_normal">15sp</dimen>
    <dimen name="font_small">12sp</dimen>

    <!-- typical spacing between two views -->
    <dimen name="spacing_huge">40dp</dimen>
    <dimen name="spacing_large">24dp</dimen>
    <dimen name="spacing_normal">14dp</dimen>
    <dimen name="spacing_small">10dp</dimen>
    <dimen name="spacing_tiny">4dp</dimen>

    <!-- typical sizes of views -->
    <dimen name="button_height_tall">60dp</dimen>
    <dimen name="button_height_normal">40dp</dimen>
    <dimen name="button_height_short">32dp</dimen>

</resources>
```

你应该用 `spacing_****` 尺寸值 来定义间距 边界 而不是 写死, 就像我们平常定义string字符串通用时候一样. 用这种常量给人一种好看的感觉而且改起来比较容易（还是那点，能抽的就抽出来，不要重复写代码）.

**strings.xml**

用那种带命名空间的方式来命名string, 而且不要担心重复写好多key麻烦. 语言这种东西是复杂的，你不写具体点多加个命名很容易含糊不清.

**差的方式**
```xml
<string name="network_error">Network error</string>
<string name="call_failed">Call failed</string>
<string name="map_failed">Map loading failed</string>
```

**好的方式**
```xml
<string name="error.message.network">Network error</string>
<string name="error.message.call">Call failed</string>
<string name="error.message.map">Map loading failed</string>
```

别写全大写的字符串. 坚持正常的字符结构  (比如首字母大写). 如果你想让所有字母都大写可以这么做, 加[`textAllCaps`](http://developer.android.com/reference/android/widget/TextView.html#attr_android:textAllCaps) 这个属性到Textview上.

**差的方式**
```xml
<string name="error.message.call">CALL FAILED</string>
```

**好的方式**
```xml
<string name="error.message.call">Call failed</string>
```

**别建那么深的树形视图.** 有时候为了能整理码一大堆界面，你可能尝试就贪图省事加个没弄完的LinearLayout. 这种情况就发生了:

```xml
<LinearLayout
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    >

    <RelativeLayout
        ...
        >

        <LinearLayout
            ...
            >

            <LinearLayout
                ...
                >

                <LinearLayout
                    ...
                    >
                </LinearLayout>

            </LinearLayout>

        </LinearLayout>

    </RelativeLayout>

</LinearLayout>
```

即使你不是特别重视，有时候这种情况开始会发生.

很容易出现问题. 你可能遇到性能问题, 因为处理器需要处理一大堆复杂UI树. 最糟糕可能还会是这个 [StackOverflowError](http://stackoverflow.com/questions/2762924/java-lang-stackoverflow-error-suspected-too-many-views)问题.

所以说, 尽量保证你的view 别那么深: 学习怎么使用 [RelativeLayout](https://developer.android.com/guide/topics/ui/layout/relative.html), 怎么 [优化布局](http://developer.android.com/training/improving-layouts/optimizing-layout.html) 并且尝试使用 [`<merge>` 标签](http://stackoverflow.com/questions/8834898/what-is-the-purpose-of-androids-merge-tag-in-xml-layouts).

**当心WebViews会遇到的问题.** 当你需要展示一个网页, 比如新闻页面, 不要再客户端清理html元素, 宁可从后台用  "*纯的*" HTML . [WebViews也会导致内存泄露](http://stackoverflow.com/questions/3130654/memory-leak-in-webview) 当被activity 引用时候, 而不是受 约束ApplicationContext. 尽量TextViews 或者 Buttons代替webview 来干一些事情.


### 测试用的框架

Android SDK's 的测试框架还挺薄弱, 尤其是UI测试方面. Android Gradle 最近实现了测试任务叫 [`connectedAndroidTest`](http://tools.android.com/tech-docs/new-build-system/user-guide#TOC-Testing) 可以用来跑单元测试, 使用的是  [JUnit的 Android 扩展](http://developer.android.com/reference/android/test/package-summary.html). 也就是说测试时候要连设备 或者模拟器. 要测试可以查阅下这个参考手册 [[1]](http://developer.android.com/tools/testing/testing_android.html) [[2]](http://developer.android.com/tools/testing/activity_test.html) .

**用 [Robolectric](http://robolectric.org/) 这个就单单跑单元测试用, 没法跑界面测试.** 这个测试框架提供了 "不连接设备" 达到 models and view models 关于. 然而, 在 Robolectric 下面测试不精确也没法完整的做ＵＩ测试s. 测试 UI 动画元素 , 对话框, 等等, and this will be complicated by the fact that you are "walking in the dark" (testing without seeing the screen being controlled).

**[Robotium](https://code.google.com/p/robotium/) makes writing UI tests easy.** You do not need Robotium for running connected tests for UI cases, but it will probably be beneficial to you because of its many helpers to get and analyse views, and control the screen. Test cases will look as simple as:

```java
solo.sendKey(Solo.MENU);
solo.clickOnText("More"); // searches for the first occurence of "More" and clicks on it
solo.clickOnText("Preferences");
solo.clickOnText("Edit File Extensions");
Assert.assertTrue(solo.searchText("rtf"));
```

### 模拟器

要是你是个专业的android 开发者, 买个 [Genymotion 模拟器](http://www.genymotion.com/)证书不错. Genymotion emulators run at a faster frames/sec rate than typical AVD emulators. They have tools for demoing your app, emulating network connection quality, GPS positions, etc. They are also ideal for connected tests. You have access to many (not all) different devices, so the cost of a Genymotion license is actually much cheaper than buying multiple real devices.

说明: Genymotion 模拟器保留了完整的google服务框架 比如google商店 地图什么的 . 你可能需要测试 Samsung-specific APIs, 所以买个三星设备也许挺需要.

### 代码混淆器的配置

[ProGuard](http://proguard.sourceforge.net/) 在 Android项目中 压缩代码模糊代码方面 用的很普遍.

Whether you are using ProGuard or not depends on your project configuration. Usually you would configure gradle to use ProGuard when building a release apk.

```groovy
buildTypes {
    debug {
        minifyEnabled false
    }
    release {
        signingConfig signingConfigs.release
        minifyEnabled true
        proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
    }
}
```

In order to determine which code has to be preserved and which code can be discarded or obfuscated, you have to specify one or more entry points to your code. These entry points are typically classes with main methods, applets, midlets, activities, etc.
Android framework uses a default configuration which can be found from `SDK_HOME/tools/proguard/proguard-android.txt`. Using the above configuration, custom project-specific ProGuard rules, as defined in `my-project/app/proguard-rules.pro`, will be appended to the default configuration.

A common problem related to ProGuard is to see the application crashing on startup with `ClassNotFoundException` or `NoSuchFieldException` or similar, even though the build command (i.e. `assembleRelease`) succeeded without warnings.
This means one out of two things:

1. ProGuard has removed the class, enum, method, field or annotation, considering it's not required.
2. ProGuard has obfuscated (renamed) the class, enum or field name, but it's being used indirectly by its original name, i.e. through Java reflection.

Check `app/build/outputs/proguard/release/usage.txt` to see if the object in question has been removed.
Check `app/build/outputs/proguard/release/mapping.txt` to see if the object in question has been obfuscated.

In order to prevent ProGuard from *stripping away* needed classes or class members, add a `keep` options to your ProGuard config:
```
-keep class com.futurice.project.MyClass { *; }
```

To prevent ProGuard from *obfuscating* classes or class members, add a `keepnames`:
```
-keepnames class com.futurice.project.MyClass { *; }
```

Check [this template's ProGuard config](https://github.com/futurice/android-best-practices/blob/master/templates/rx-architecture/app/proguard-rules.pro) for some examples.
Read more at [Proguard](http://proguard.sourceforge.net/#manual/examples.html) for examples.

**Early on in your project, make a release build** to check whether ProGuard rules are correctly keeping whatever is important. Also whenever you include new libraries, make a release build and test the apk on a device. Don't wait until your app is finally version "1.0" to make a release build, you might get several unpleasant surprises and a short time to fix them.

**建议.** Save the `mapping.txt` file for every release that you publish to your users. By retaining a copy of the `mapping.txt` file for each release build, you ensure that you can debug a problem if a user encounters a bug and submits an obfuscated stack trace.

**DexGuard**. If you need hard-core tools for optimizing, and specially obfuscating release code, consider [DexGuard](http://www.saikoa.com/dexguard), a commercial software made by the same team that built ProGuard. It can also easily split Dex files to solve the 65k methods limitation.

### 感谢

Antti Lammi, Joni Karppinen, Peter Tackage, Timo Tuominen, Vera Izrailit, Vihtori Mäntylä, Mark Voit, Andre Medeiros, Paul Houghton and other Futurice developers for sharing their knowledge on Android development.

### License

[Futurice Oy](http://www.futurice.com)
Creative Commons Attribution 4.0 International (CC BY 4.0)