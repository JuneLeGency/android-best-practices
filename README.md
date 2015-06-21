# Android 开发的最正确做法

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

**[Retrolambda](https://github.com/evant/gradle-retrolambda)** 是个在Android和JDK8预览版平台用lambda表达式的库 . It helps keep your code tight and readable especially if you use a functional style with for example with RxJava. To use it, install JDK8, set that as your SDK Location in the Android Studio Project Structure dialog, and set `JAVA8_HOME` and `JAVA7_HOME` environment variables, then in the project root build.gradle:

```groovy
dependencies {
    classpath 'me.tatarka:gradle-retrolambda:2.4.1'
}
```

and in each module's build.gradle, add

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

Android Studio offers code assist support for Java8 lambdas. If you are new to lambdas, just use the following to get started:

- Any interface with just one method is "lambda friendly" and can be folded into the more tight syntax
- If in doubt about parameters and such, write a normal anon inner class and then let Android Studio fold it into a lambda for you.

**Beware of the dex method limitation, and avoid using many libraries.** Android apps, when packaged as a dex file, have a hard limitation of 65536 referenced methods [[1]](https://medium.com/@rotxed/dex-skys-the-limit-no-65k-methods-is-28e6cb40cf71) [[2]](http://blog.persistent.info/2014/05/per-package-method-counts-for-androids.html) [[3]](http://jakewharton.com/play-services-is-a-monolith/). You will see a fatal error on compilation if you pass the limit. For that reason, use a minimal amount of libraries, and use the [dex-method-counts](https://github.com/mihaip/dex-method-counts) tool to determine which set of libraries can be used in order to stay under the limit. Especially avoid using the Guava library, since it contains over 13k methods.

### Activities 和 Fragments

There is no consensus among the community nor Futurice developers how to best organize Android architectures with Fragments and Activities. Square even has [a library for building architectures mostly with Views](https://github.com/square/mortar), bypassing the need for Fragments, but this still is not considered a widely recommendable practice in the community.

Because of Android API's history, you can loosely consider Fragments as UI pieces of a screen. In other words, Fragments are normally related to UI. Activities can be loosely considered to be controllers, they are specially important for their lifecycle and for managing state. However, you are likely to see variation in these roles: activities might be take UI roles ([delivering transitions between screens](https://developer.android.com/about/versions/lollipop.html)), and [fragments might be used solely as controllers](http://developer.android.com/guide/components/fragments.html#AddingWithoutUI). We suggest to sail carefully, taking informed decisions since there are drawbacks for choosing a fragments-only architecture, or activities-only, or views-only. Here are some advices on what to be careful with, but take them with a grain of salt:

- 避免大规模 [嵌套 fragments](https://developer.android.com/about/versions/android-4.2.html#NestedFragments) , 因为 会有这种[matryoshka bugs](http://delyan.me/android-s-matryoshka-problem/) . Use nested fragments only when it makes sense (for instance, fragments in a horizontally-sliding ViewPager inside a screen-like fragment) or if it's a well-informed decision.
- Avoid putting too much code in activities. Whenever possible, keep them as lightweight containers, existing in your application primarily for the lifecycle and other important Android-interfacing APIs. Prefer single-fragment activities instead of plain activities - put UI code into the activity's fragment. This makes it reusable in case you need to change it to reside in a tabbed layout, or in a multi-fragment tablet screen. Avoid having an activity without a corresponding fragment, unless you are making an informed decision.
- Don't abuse Android-level APIs such as heavily relying on Intent for your app's internal workings. You could affect the Android OS or other applications, creating bugs or lag. For instance, it is known that if your app uses Intents for internal communication between your packages, you might incur multi-second lag on user experience if the app was opened just after OS boot.

### Java 包结构

Java architectures for Android applications can be roughly approximated in [Model-View-Controller](http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller). In Android, [Fragment and Activity are actually controller classes](http://www.informit.com/articles/article.aspx?p=2126865). On the other hand, they are explicity part of the user interface, hence are also views.

For this reason, it is hard to classify fragments (or activities) as strictly controllers or views. It's better to let them stay in their own `fragments` package. Activities can stay on the top-level package as long as you follow the advice of the previous section. If you are planning to have more than 2 or 3 activities, then make also an `activities` package.

Otherwise, the architecture can look like a typical MVC, with a `models` package containing POJOs to be populated through the JSON parser with API responses, and a `views` package containing your custom Views, notifications, action bar views, widgets, etc. Adapters are a gray matter, living between data and views. However, they typically need to export some View via `getView()`, so you can include the `adapters` subpackage inside `views`.

Some controller classes are application-wide and close to the Android system. These can live in a `managers` package. Miscellaneous data processing classes, such as "DateUtils", stay in the `utils` package. Classes that are responsible for interacting with the backend stay in the `network` package.

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

**命名.** Follow the convention of prefixing the type, as in `type_foo_bar.xml`. Examples: `fragment_contact_details.xml`, `view_primary_button.xml`, `activity_main.xml`.

**组织 layout XMLs文件.** If you're unsure how to format a layout XML, the following convention may help.

- One attribute per line, indented by 4 spaces
- `android:id` as the first attribute always
- `android:layout_****` attributes at the top
- `style` attribute at the bottom
- Tag closer `/>` on its own line, to facilitate ordering and adding attributes.
- Rather than hard coding `android:text`, consider using [Designtime attributes](http://tools.android.com/tips/layout-designtime-attributes) available for Android Studio.

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

As a rule of thumb, attributes `android:layout_****` should be defined in the layout XML, while other attributes `android:****` should stay in a style XML. This rule has exceptions, but in general works fine. The idea is to keep only layout (positioning, margin, sizing) and content attributes in the layout files, while keeping all appearance details (colors, padding, font) in styles files.

The exceptions are:

- `android:id` should obviously be in the layout files
- `android:orientation` for a `LinearLayout` normally makes more sense in layout files
- `android:text` should be in layout files because it defines content
- Sometimes it will make sense to make a generic style defining `android:layout_width` and `android:layout_height` but by default these should appear in the layout files

**用法.** Almost every project needs to properly use styles, because it is very common to have a repeated appearance for a view. At least you should have a common style for most text content in the application, for example:

```xml
<style name="ContentText">
    <item name="android:textSize">@dimen/font_normal</item>
    <item name="android:textColor">@color/basic_black</item>
</style>
```

Applied to TextViews:

```xml
<TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@string/price"
    style="@style/ContentText"
    />
```

You probably will need to do the same for buttons, but don't stop there yet. Go beyond and move a group of related and repeated `android:****` attributes to a common style.

**Split a large style file into other files.** You don't need to have a single `styles.xml` file. Android SDK supports other files out of the box, there is nothing magical about the name `styles`, what matters are the XML tags `<style>` inside the file. Hence you can have files `styles.xml`, `styles_home.xml`, `styles_item_details.xml`, `styles_forms.xml`. Unlike resource directory names which carry some meaning for the build system, filenames in `res/values` can be arbitrary.

**`colors.xml` is a color palette.** There should be nothing else in your `colors.xml` than just a mapping from a color name to an RGBA value. Do not use it to define RGBA values for different types of buttons.

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

You can easily start repeating RGBA values in this format, and that makes it complicated to change a basic color if needed. Also, those definitions are related to some context, like "button" or "comment", and should live in a button style, not in `colors.xml`.

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

Ask for this palette from the designer of the application. The names do not need to be color names as "green", "blue", etc. Names such as "brand_primary", "brand_secondary", "brand_negative" are totally acceptable as well. Formatting colors as such will make it easy to change or refactor colors, and also will make it explicit how many different colors are being used. Normally for a aesthetic UI, it is important to reduce the variety of colors being used.

**Treat dimens.xml like colors.xml.** You should also define a "palette" of typical spacing and font sizes, for basically the same purposes as for colors. A good example of a dimens file:

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

You should use the `spacing_****` dimensions for layouting, in margins and paddings, instead of hard-coded values, much like strings are normally treated. This will give a consistent look-and-feel, while making it easier to organize and change styles and layouts.

**strings.xml**

Name your strings with keys that resemble namespaces, and don't be afraid of repeating a value for two or more keys. Languages are complex, so namespaces are necessary to bring context and break ambiguity.

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

**别建那么深的树形视图.** Sometimes you might be tempted to just add yet another LinearLayout, to be able to accomplish an arrangement of views. This kind of situation may occur:

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

Even if you don't witness this explicitly in a layout file, it might end up happening if you are inflating (in Java) views into other views.

A couple of problems may occur. You might experience performance problems, because there are is a complex UI tree that the processor needs to handle. Another more serious issue is a possibility of [StackOverflowError](http://stackoverflow.com/questions/2762924/java-lang-stackoverflow-error-suspected-too-many-views).

Therefore, try to keep your views hierarchy as flat as possible: learn how to use [RelativeLayout](https://developer.android.com/guide/topics/ui/layout/relative.html), how to [optimize your layouts](http://developer.android.com/training/improving-layouts/optimizing-layout.html) and to use the [`<merge>` tag](http://stackoverflow.com/questions/8834898/what-is-the-purpose-of-androids-merge-tag-in-xml-layouts).

**Beware of problems related to WebViews.** When you must display a web page, for instance for a news article, avoid doing client-side processing to clean the HTML, rather ask for a "*pure*" HTML from the backend programmers. [WebViews can also leak memory](http://stackoverflow.com/questions/3130654/memory-leak-in-webview) when they keep a reference to their Activity, instead of being bound to the ApplicationContext. Avoid using a WebView for simple texts or buttons, prefer TextViews or Buttons.


### 测试用的框架

Android SDK's testing framework is still infant, specially regarding UI tests. Android Gradle currently implements a test task called [`connectedAndroidTest`](http://tools.android.com/tech-docs/new-build-system/user-guide#TOC-Testing) which runs JUnit tests that you created, using an [extension of JUnit with helpers for Android](http://developer.android.com/reference/android/test/package-summary.html). This means you will need to run tests connected to a device, or an emulator. Follow the official guide [[1]](http://developer.android.com/tools/testing/testing_android.html) [[2]](http://developer.android.com/tools/testing/activity_test.html) for testing.

**Use [Robolectric](http://robolectric.org/) only for unit tests, not for views.** It is a test framework seeking to provide tests "disconnected from device" for the sake of development speed, suitable specially for unit tests on models and view models. However, testing under Robolectric is inaccurate and incomplete regarding UI tests. You will have problems testing UI elements related to animations, dialogs, etc, and this will be complicated by the fact that you are "walking in the dark" (testing without seeing the screen being controlled).

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
