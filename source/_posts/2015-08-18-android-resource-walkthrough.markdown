---
layout: post
title: "Android Resource walkthrough"
date: 2015-08-18 23:13:17 +0800
comments: true
categories: Android
---
> 关于Android Resource这一块[老罗博客](http://blog.csdn.net/luoshengyang/article/details/8738877)有详细分析，并且罗老师的博客大家都懂的，质量很高。但是我还是决定把自己从
开发者的角度对这一块的理解记录下来，所以就有了这篇文章，本文章也来自于对罗老师博客的学习研究。

文章主要从 **代码编写**、**代码编译**、**代码运行** 三个阶段来分析。
## 代码编写阶段
首先我们来看Android应用资源分类,Android应用程序资源可以分为两大类，分别是assets和res。

* assets
  * assets类资源放在工程根目录的assets子目录下，它里面保存的是一些原始的文件，可以以任何方式来进行组织。这些文件最终会被原装不动地打包在apk文件中。如果我们要在程序中访问这些文件，那么
  就需要指定文件名来访问。

<!-- More-->

* res
  * animator 这类资源以XML文件保存在res/animator目录下，用来描述属性动画。
  * anim 这类资源以XML文件保存在res/anim目录下，用来描述补间动画。
  * color 这类资源以XML文件保存在res/color目录下，用来描述对象颜色状态。
  * drawable(mipmap) 这类资源以XML或者Bitmap文件保存在res/drawable目录下，用来描述可绘制对象。
  * layout 这类资源以XML文件保存在res/layout目录下，用来描述应用程序界面布局。
  * menu 这类资源以XML文件保存在res/menu目录下，用来描述应用程序菜单.
  * raw 这类资源以任意格式的文件保存在res/raw目录下，它们和assets类资源一样，都是原装不动地打包在apk文件中的，不过它们会被赋予资源ID，这样我们就可以在程序中通过ID来访问它们。
  * values 这类资源以XML文件保存在res/values目录下，用来描述一些简单值，例如，数组、颜色、尺寸、字符串和样式值等，一般来说，这六种不同的值分别保存在名称为arrays.xml、colors.xml、
  dimens.xml、strings.xml和styles.xml文件中。
  * xml 这类资源以XML文件保存在res/xml目录下，一般就是用来描述应用程序的配置信息。

我们在一般开发App过程中resource这一块主要就是操作这些文件夹，需要额外说的地方就是Android resource对于多分辨率、多语言、多版本、多屏幕等的支持。在这一块有一种说法就是Android默认应用程序资源的组织
方式有18个维度，当然如果从系统层次搞明白Android资源加载的话我们是可以自定义一些维度进去的。

| 配置 | 限定修饰词 |
| :---- | -------: |
| MCC and MNC |    mcc310等 |
| language and region | en、fr、en-rFR、zh等 |
| layout direction | ldrtl、ldltr等 |
| smallestWidth | sw320dp、sw600dp等 |
| available width | w720dp、w1024dp等 |
| available height | h720dp、h1024dp等 |
| screen size | small、normal、large、xlarge等 |
| screen aspect | long、notlong |
| screen orientation | port、land |
| UI mode | car、desk、television等 |
| night mode | night、notnight |
| screen pixel density | ldpi、mdpi、hdpi、xhdpi等 |
| touchscreen type | notouch、finger |
| keyboard availability | keysexposed、keyshidden、keysoft |
| primary text input method | nokeys、qwerty、12key |
| navigation key availability | navexposed、 navhidden |
| primary non-touch navigation method | nonav、dpad、trackball等 |
| platform version(API level) | v14、v16、v21等 |

## 代码编译阶段
Android resource资源是通过aapt(Android Asset Package Tool)打包到应用apk中的，在打包之前会做以下三件事情

* 将类型为res/animator、res/anim、res/color、res/drawable（非Bitmap文件，即非.png、.9.png、.jpg、.gif文件）、res/layout、res/menu、res/values和res/xml的资源文件会从文
本格式的XML文件编译成二进制格式的XML文件。
  * 二进制格式的XML文件占用空间更小。这是由于所有XML元素的标签、属性名称、属性值和内容所涉及到的字符串都会被统一收集到一个字符串资源池中去，并且会去重。有了这个字符串资源池，原来使用字符串的
  地方就会被替换成一个索引到字符串资源池的整数值，从而可以减少文件的大小。
  * 二进制格式的XML文件解析速度更快。这是由于二进制格式的XML元素里面不再包含有字符串值，因此就避免了进行字符串解析，从而提高速度。
* 赋予每一个非assets资源一个ID值，这些ID值以常量的形式定义在R.java文件中。
* 生成一个resources.arsc文件，用来描述具有ID值的资源的配置信息，它的内容就相当于是一个资源索引表。

接下来我们就按照Android资源打包工具的执行流程来分析

* 解析AndroidManifest.xml
  * 解析AndroidManifest.xml是为了得到应用程序的包名称，有了包名称就可以创建ResourceTable对象了。
     * ResourceTable用来总体描述一个资源表，具体ResourceTable可以在源码/frameworks/base/tools/aapt目录下找到。
* 添加被引用资源包
  * 在原生Android中，编译应用程序资源的时候，一般会涉及到两个包，其中一个是被引用的系统资源包（Android系统编译的时候会在out/target/common/obj/APPS/framework-res_intermediates
  目录下生成一个package-export.apk），另外一个就是当前正在编译的应用程序资源包。当然一些第三方定制ROM也会有自己的资源包，也会被添加进来。
* 收集资源文件
  * 在编译应用程序之前，Android资源打包工具aapt会创建一个AaptAssets对象，用来收集需要编译的资源文件。
     * AaptAssets类可以在/frameworks/base/tools/aapt目录下可以找到。
     * 收集到的资源是按照类型来保存的，在AaptAssets中有一个ResourceTypeSet的变量在记录。
* 将收集到的资源增加到资源表
  * 将上一步收集到的资源文件同时增加到资源表中，即第一步创建的ResourceTable对象。注意，这一步不会将values类型的资源收集到资源表。
* 编译values类资源
  * values类型的资源是在编译的过程中收集的，也会同样添加到ResourceTable对象中。
* 给Bag资源分配ID
  * values类型的资源，如string、bag、style、plurals和array类的资源。这些资源会给自己定义一些专用的值，这些带有专用值的资源就统称为Bag资源。
  在继续编译其它非values的资源之前，需要给之前收集到的Bag资源分配资源ID，因为它们可能会被其它非values类资源引用到。
* 编译xml资源文件
  * 这一步会编译除了values类型的所有资源。这一块比较繁杂，如果有兴趣可以单独研究，我就大概看了下。
     * 解析xml文件
     * 赋予属性名称资源ID
     * 解析属性值
     * 将xml文件转换为二进制格式
        * 收集有资源ID的属性的名称字符串
        * 收集其他字符串
        * 写入xml文件头
        * 写入字符串资源池
        * 写入资源ID
        * 将xml文件里边的字符串替换掉
* 生成资源符号
  * Android资源打包工具aapt只要遍历每一个ResourceTable对象里边的type，并根据entry的名称以及出现的次序来计算的到资源ID。
* 生成资源索引表
  * 这个过程就是生成resources.arsc的过程
     * 收集类型字符串
     * 收集资源项名称字符串
     * 收集资源项值字符串
     * 生成Package数据块（这一块同样略复杂）
         * 写入Package资源项元信息数据块头部,这一块有一个比较注意的点就是，类似于Android或者第三方定制rom会在resource里边定义public标签的属性之，这个地方是为了保证不论何时编译，这些
         属性值ID的唯一性。
         * 写入字符串资源池
         * 写入资源项名称字符串资源池
         * 写入类型规范数据块
         * 写入类型资源项数据块
     * 写入资源索引表头部
     * 写入资源项的字符串资源池
     * 写入Package数据块
* 编译AndroidManifest.xml
  * 当前边步骤都进行完之后就会把AndroidManifest.xml也编译成二进制格式的xml文件，之所以要在应用程序的所有资源都编译完成之后再编译AndroidManifest，是因为后者可能会引用前者。
* 生成R.java文件
  * 前边步骤已经将所有资源以及对应的ID都收集起来了，直接写入到R.java中即可。
* 打包APK文件
  * assets目录打包进apk
  * res目录打包进apk，除了res/values，因为values目录下的资源在编译的过程中都直接写入到资源索引表中去了，这也是为什么values类型资源会单独编译的原因。
  * 资源索引文件resources.arsc打包进apk
  * AndroidManifest.xml打包进apk
  * 其他文件打包进apk，比如class.dex、签名等。

## 代码运行阶段　
在代码运行阶段主要是通过AssetManager和Resources两个类来完成资源的加载的。Resources类根据ID（ID来源于之前生成并打包进apk的resources.arsc文件）来查找资源，AssetManager根据文件
名来查找资源。我们同样以一个Activity OnCreate的流程来分析

* Activity.setContentView
```java
/**
     * Set the activity content from a layout resource.  The resource will be
     * inflated, adding all top-level views to the activity.
     *
     * @param layoutResID Resource ID to be inflated.
     *
     * @see #setContentView(android.view.View)
     * @see #setContentView(android.view.View, android.view.ViewGroup.LayoutParams)
     */
    public void setContentView(int layoutResID) {
        getWindow().setContentView(layoutResID);
        initWindowDecorActionBar();
    }
```
* PhoneWindow.setContentView
```
@Override
    public void setContentView(int layoutResID) {
        // Note: FEATURE_CONTENT_TRANSITIONS may be set in the process of installing the window
        // decor, when theme attributes and the like are crystalized. Do not check the feature
        // before this happens.
        if (mContentParent == null) {
            installDecor();
        } else if (!hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
            mContentParent.removeAllViews();
        }

        if (hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
            final Scene newScene = Scene.getSceneForLayout(mContentParent, layoutResID,
                    getContext());
            transitionTo(newScene);
        } else {
            mLayoutInflater.inflate(layoutResID, mContentParent);
        }
        final Callback cb = getCallback();
        if (cb != null && !isDestroyed()) {
            cb.onContentChanged();
        }
    }
```
* LayoutInflater.inflate
```
/**
     * Inflate a new view hierarchy from the specified xml resource. Throws
     * {@link InflateException} if there is an error.
     *
     * @param resource ID for an XML layout resource to load (e.g.,
     *        <code>R.layout.main_page</code>)
     * @param root Optional view to be the parent of the generated hierarchy (if
     *        <em>attachToRoot</em> is true), or else simply an object that
     *        provides a set of LayoutParams values for root of the returned
     *        hierarchy (if <em>attachToRoot</em> is false.)
     * @param attachToRoot Whether the inflated hierarchy should be attached to
     *        the root parameter? If false, root is only used to create the
     *        correct subclass of LayoutParams for the root view in the XML.
     * @return The root View of the inflated hierarchy. If root was supplied and
     *         attachToRoot is true, this is root; otherwise it is the root of
     *         the inflated XML file.
     */
    public View inflate(int resource, ViewGroup root, boolean attachToRoot) {
        final Resources res = getContext().getResources();
        if (DEBUG) {
            Log.d(TAG, "INFLATING from resource: \"" + res.getResourceName(resource) + "\" ("
                    + Integer.toHexString(resource) + ")");
        }

        final XmlResourceParser parser = res.getLayout(resource);
        try {
            return inflate(parser, root, attachToRoot);
        } finally {
            parser.close();
        }
    }
```
   * 最终调用到了Resource.getLayout
* Resources.getLayout
```
/**
     * Return an XmlResourceParser through which you can read a view layout
     * description for the given resource ID.  This parser has limited
     * functionality -- in particular, you can't change its input, and only
     * the high-level events are available.
     *
     * <p>This function is really a simple wrapper for calling
     * {@link #getXml} with a layout resource.
     *
     * @param id The desired resource identifier, as generated by the aapt
     *           tool. This integer encodes the package, type, and resource
     *           entry. The value 0 is an invalid identifier.
     *
     * @throws NotFoundException Throws NotFoundException if the given ID does not exist.
     *
     * @return A new parser object through which you can read
     *         the XML data.
     *
     * @see #getXml
     */
    public XmlResourceParser getLayout(int id) throws NotFoundException {
        return loadXmlResourceParser(id, "layout");
    }
```
* Resources.loadXmlResourceParser
* Resources.getValue
* AssetManager.getResourceValue
* AssetManager.loadResourceValue
   * 这是一个native方法，对应到android_content_AssetManager_loadResourceValue函数，这个函数定义在文件frameworks/base/core/jni/android_util_AssetManager.cpp中。
   得到的资源项值及其配置信息拷贝到参数outValue所描述的一个Java层的TypedValue对象中。这个地方会有一个ResTable的对象用来记录一系列数据。
* AssetManager.getResources
   * AssetManager类的成员函数getResources的实现看起来比较复杂，但是它要做的事情就是解析当前应用程序所使用的资源包里面的resources.arsc文件。
* ResTable.getResource
   * 根据之前AssetManager.loadResourceValue得到的各种信息，在上一步解析resources.arsc得到的数据中get到相应的value值。这一块的逻辑处理非常复杂，我并没有深入研究下去。
* ResTable.resolveReference
   * ResTable类的成员函数resolveReference的实现其实很简单，它就是对参数value所描述的一个资源项值进行解析。
* Resources.loadXmlResourceParser
```
/*package*/ XmlResourceParser loadXmlResourceParser(String file, int id,
            int assetCookie, String type) throws NotFoundException {
        if (id != 0) {
            try {
                // These may be compiled...
                synchronized (mCachedXmlBlockIds) {
                    // First see if this block is in our cache.
                    final int num = mCachedXmlBlockIds.length;
                    for (int i=0; i<num; i++) {
                        if (mCachedXmlBlockIds[i] == id) {
                            //System.out.println("**** REUSING XML BLOCK!  id="
                            //                   + id + ", index=" + i);
                            return mCachedXmlBlocks[i].newParser();
                        }
                    }

                    // Not in the cache, create a new block and put it at
                    // the next slot in the cache.
                    XmlBlock block = mAssets.openXmlBlockAsset(
                            assetCookie, file);
                    if (block != null) {
                        int pos = mLastCachedXmlBlockIndex+1;
                        if (pos >= num) pos = 0;
                        mLastCachedXmlBlockIndex = pos;
                        XmlBlock oldBlock = mCachedXmlBlocks[pos];
                        if (oldBlock != null) {
                            oldBlock.close();
                        }
                        mCachedXmlBlockIds[pos] = id;
                        mCachedXmlBlocks[pos] = block;
                        //System.out.println("**** CACHING NEW XML BLOCK!  id="
                        //                   + id + ", index=" + pos);
                        return block.newParser();
                    }
                }
            } catch (Exception e) {
                NotFoundException rnf = new NotFoundException(
                        "File " + file + " from xml type " + type + " resource ID #0x"
                        + Integer.toHexString(id));
                rnf.initCause(e);
                throw rnf;
            }
        }

        throw new NotFoundException(
                "File " + file + " from xml type " + type + " resource ID #0x"
                + Integer.toHexString(id));
    }
```
* AssetManager.openXmlBlockAsset
```
/**
     * {@hide}
     * Retrieve a non-asset as a compiled XML file.  Not for use by
     * applications.
     *
     * @param cookie Identifier of the package to be opened.
     * @param fileName Name of the asset to retrieve.
     */
    /*package*/ final XmlBlock openXmlBlockAsset(int cookie, String fileName)
        throws IOException {
        synchronized (this) {
            if (!mOpen) {
                throw new RuntimeException("Assetmanager has been closed");
            }
            long xmlBlock = openXmlAssetNative(cookie, fileName);
            if (xmlBlock != 0) {
                XmlBlock res = new XmlBlock(this, xmlBlock);
                incRefsLocked(res.hashCode());
                return res;
            }
        }
        throw new FileNotFoundException("Asset XML file: " + fileName);
    }
```
* AssetManager.openXmlAssetNative

...

这中间有一系列的native方法调用

* LayoutInflater.inflate
  * 这一步根据之前得到的Parser对象来继续进行
* LayoutInflater.createViewFromTag
  * 根据应用的属性值得到一个view
* PhoneLayoutInflater.onCreateView
* LayoutInflater.createView

这一块会有一个疑问，应用是如何找到Android系统资源的？在之前提到的编译阶段，系统提供的apk资源包的资源也会被收集进来，那么运行阶段是如何找到的呢？

系统的资源打包在/system/framework/framework-res.apk文件中，它在应用程序进程中是通过一个单独的Resources对象和一个单独的AssetManager对象来管理的。这个单独的Resources
对象就保存在Resources类的静态成员变量mSystem中，我们可以通过Resources类的静态成员函数getSystem就可以获得这个Resources对象，而这个单独的AssetManager对象就保存在AssetManager
类的静态成员变量sSystem中，我们可以通过AssetManager类的静态成员函数getSystem同样可以获得这个AssetManager对象。
