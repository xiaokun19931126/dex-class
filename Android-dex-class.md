## Android的热更新小记

#### **关键类：DexClassLoader** 

> **关于DexClassLoader的官方解释：(双语注释)**  

![](http://ww2.sinaimg.cn/large/82c8e86egw1fbgqttq8t2j20r60dpgnj.jpg)

上述翻译大概如下：

一个类加载器，从.jar和.apk文件加载包含classes.dex条目的类。 这可以用于执行未作为应用程序的一部分安装的代码。

这个类加载器需要一个应用程序专用的可写目录来缓存优化的类。 使用Context.getCodeCacheDir（）创建这样的目录：

```
File dexOutputDir = context.getCodeCacheDir();//官方推荐此方法，但是这个方法需要api21
File dexOutputDir = mContext.getDir("dex", 0);//用这个方法可兼容
```

不要在外部存储上缓存优化的类。外部存储不提供必要的访问控制来保护您的应用程序免受代码注入攻击。



> **关于构造方法Public Constructors中的官方解释**DexClassLoader ([String](../../../reference/java/lang/String.html) dexPath, [String](../../../reference/java/lang/String.html) optimizedDirectory, [String](../../../reference/java/lang/String.html) libraryPath, [ClassLoader](../../../reference/java/lang/ClassLoader.html) parent)

![](http://ww4.sinaimg.cn/large/82c8e86egw1fbgqualfmuj20r10emabq.jpg)

创建一个DexClassLoader，用于查找解释和本机代码。 解释类存在于Jar或APK文件中包含的一组DEX文件中。

路径列表使用path.separator系统属性指定的字符分隔，默认为：.

**参数解释：**

**dex路径**    包含类和资源的jar / apk文件列表，由File.pathSeparator分隔，默认为Android上的“：”

**优化目录**    应该写入优化的dex文件的目录; 不能为null

**库路径**    包含本地库的目录列表，由File.pathSeparator分隔; 可以为null

**父母**    父类加载器



> **代码示例**

```
File dexOutputDir = mContext.getDir("dex", 0);
KLog.e("dexOutputDir:" + dexOutputDir.getAbsolutePath());
DexClassLoader cl = new DexClassLoader(file.getAbsolutePath(),
        dexOutputDir.getAbsolutePath(), null, getClassLoader());
Class mLoadClass = null;

try {
    mLoadClass = cl.loadClass("com.example.xiusou_regist.RegisterTool");
    Method[] methods = mLoadClass.getDeclaredMethods();

    for (int i = 0; i < methods.length; i++) {
        KLog.d(methods[i].toString());
    }

    Method init = mLoadClass.getDeclaredMethod("init", Context.class,
            String.class);
    init.setAccessible(true);
    init.invoke(mContext, new Object[]{mContext, "1234567"});
    // 调用方法
} catch (Exception exception) {
    exception.printStackTrace();
}
```

