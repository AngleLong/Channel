[不怕跌倒，所以飞翔](http://www.jianshu.com/u/4a99c9554afc)

首先先说明下，这个是基于有盟统计的多渠道打包，因为有盟统计还是很强大的，一般我在公司的时候用的都是这个，所以应用懂的胡还是很广泛的，如果不想用的请绕行！嘻嘻。。。

关于多渠道打包的问题基本上是每个Android开发人员都会遇到的问题，不会的时候觉得这个东西好高大上，但是看看有关的内容其实也就是写一个脚本的问题，废话不多说，开始今天的内容，快上车！！！

###首先集成有盟统计的SDK
####1. 生成appKey
 首先要去有盟官网上去申请一个账号，写一个项目，生成一个**appkey**，这个东西应该是这个样子的！

![Appkey的说明](http://upload-images.jianshu.io/upload_images/2546238-651f58ad4baef9fe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

####2. 关联相关的类库
关于这里，一般现在都是用的studio所以直接导入到项目的关联文件就可以了！
```
compile 'com.umeng.analytics:analytics:latest.integration'
```

####3. 关联一些基本配置
因为不是要说明统计的问题，关于统计的问题可以按照官方文档进行操作，因为这里只是讲解多渠道打包，所以很大一部分就没有去写。
- 权限
```
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.READ_PHONE_STATE"/>
```
- 设置渠道和定义ID
```
<meta-data android:value="YOUR_APP_KEY" android:name="UMENG_APPKEY"/>
<meta-data android:value="Channel ID" android:name="UMENG_CHANNEL"/>
```
这里的YOUR_APP_KEY换成你自己申请的AppKey就可以了

基本上有盟就配置这些内容了，下面开始讲今天的重点了

------------------------------------华丽的分割线------------------------------------

###多渠道打包
####1.修改渠道号
```
<meta-data
        android:name="UMENG_CHANNEL"
        android:value="${UMENG_CHANNEL_VALUE}"/>
```
首先修改下渠道号，这里用的是占位符，这里你不用占位符的话，生成的包都是一个相同的包！

####2. 添加默认的渠道名称
```
multiDexEnabled true /*突破应用方法65535的方法*/
manifestPlaceholders = [UMENG_CHANNEL_VALUE: "youmeng"]/*默认的打包渠道*/
```
这个是添加在app的**build.gradle**的**defaultConfig**中设置的！整体代码如下：
```
  defaultConfig {
        applicationId "com.jinlong.channel"
        minSdkVersion 15
        targetSdkVersion 26
        versionCode 1
        versionName "1.0"
        multiDexEnabled true /*突破应用方法65535的方法*/
        manifestPlaceholders = [UMENG_CHANNEL_VALUE: "youmeng"]/*默认的打包渠道*/
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }
```
####3. 关联签名
关联签名的问题基本上就是和以前项目添加关联签名一样了没有什么可说的
```
  /*这个是自己配置的，由于debug打正式包没有意义，所以这里只要release的包就行了，所以这针对这个进行编写*/
    signingConfigs {
        debug {}

        release {
            /*配置签名文件*/
            storeFile file("D:\\Develop\\WorkSpace\\Channel\\Channel.jks")/*签名路径*/
            storePassword "123456"/*密码*/
            keyAlias "jinlong"/*关键字*/
            keyPassword "123456"/*密码*/
        }
    }

    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'

            signingConfig signingConfigs.release/*见上面的配置*/
        }
    }
```
不过这里注意一点，就是这个signingConfigs必须要写在上面，否则编译的时候不会通过，会报下面的异常。
```
Error:(24, 0) Could not get unknown property 'release' for SigningConfig container.
```

####4. 实现多渠道打包
在**signingConfigs**的同级节点处添加如下代码：
```
    productFlavors {
        xiaomi {
            manifestPlaceholders = [UMENG_CHANNEL_VALUE: "xiaomi"]
        }
        wandoujia {
            manifestPlaceholders = [UMENG_CHANNEL_VALUE: "wandoujia"]
        }
    }
```
代码说名：
就是用每一个相应的渠道名称去替换UMENG_CHANNEL_VALUE这个属性。
还有一种简单的写法：
```
    productFlavors {
        xiaomi {
//            manifestPlaceholders = [UMENG_CHANNEL_VALUE: "xiaomi"]
        }
        wandoujia {
//            manifestPlaceholders = [UMENG_CHANNEL_VALUE: "wandoujia"]
        }

        /*简单的写法*/
        productFlavors.all {
            flavor -> flavor.manifestPlaceholders = [UMENG_CHANNEL_VALUE: name]
        }
    }
```
####5. 使用命令行进行多渠道打包
最后这一步坑了我一段时间呢还！哈哈。。。
首先命令行是这样的，打所有release的包
```
./gradlew assembleRelease
```
**但是这个是苹果mac系统这么输入，如果你是Window系统的话你要把前面的"./"去掉就可以了**

还有一些其他的命令：
```
1 ./gradlew assembleDebug —— 打包debug版本
2 ./gradlew assembleXiaomiRelease —— 单独打包小米应用市场渠道的release版本
3 ./gradlew assembleXiaomi —— 单独打包小米应用市场渠道的debug和release版本
``` 
以上步骤就是集成的所有多渠道打包的步骤，如果你想要更改打包之后的名称的话继续往下看：

####6. 指定我们的release包的输出文件名就是我们的去到名字
```
     applicationVariants.all { variant ->
                variant.outputs.each { output ->
                    def outputFile = output.outputFile
                    if (outputFile != null && outputFile.name.endsWith('.apk')) {
                        def fileName = "${variant.productFlavors[0].name}" + ".apk"
                        output.outputFile = new File(outputFile.parent, fileName)
                    }
                }
            }
```
在release中直接添加上面的代码就可以了

####7. 实际开发中的一些设置
- 变更打包过程中的一些字符串更改
比如说有这样一个场景，在打包过程中我想要在小米应用市场叫小米，在华为应用市场叫华为这个就可以在相应懂的去到名称下面进行设置
```
    productFlavors {
        xiaomi {
//            manifestPlaceholders = [UMENG_CHANNEL_VALUE: "xiaomi"]
            resValue "string", "app_name", "xiaomi_app"
        }
        wandoujia {
//            manifestPlaceholders = [UMENG_CHANNEL_VALUE: "wandoujia"]
            resValue "string", "app_name", "wandoujia_app"
        }

        /*简单的写法，类似于一个for循环*/
        productFlavors.all {
            flavor -> flavor.manifestPlaceholders = [UMENG_CHANNEL_VALUE: name]
        }
    }
```
这里注意一下逻辑，当你写resValue的时候是在相应的资源文件中添加这个字符串的，所以要把项目中的字符串注解掉，否则会报错的！切记！！！

- 根据测试功能不同生成不同的包同事安装到手机上
需求是什么样的呢？就是当你给测试提交包的时候，你想让他针对Okhttp进行测试，或者针对jPush进行测试的话，你可以给他打出两个包，针对于专项测试；
```
 /*下面这两个说明的是针对不同的是功能打出两个包，同时安装到手机中去*/
        okhttp {
            applicationIdSuffix "okhttp"
            resValue "string", "app_name", "okhttp"
        }

        jpush {
            applicationIdSuffix "jpush"
            resValue "string", "app_name", "jpush"
        }
```
其实applicationIdSuffix 就相当于在包名后面添加它后面的字段打出一个包，然后运行就可以了，专项测试！


[Demo地址](https://github.com/AngleLong/Channel)
