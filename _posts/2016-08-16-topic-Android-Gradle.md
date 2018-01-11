---
title: Gradle多渠道打包
excerpt: 动态设定App名称，应用图标，替换常量，更改包名，变更渠道
categories:
  - topics
tag: Android  
---

> 如果本文帮助到你，本人不胜荣幸，如果浪费了你的时间，本人深感抱歉。
希望用最简单的大白话来帮助那些像我一样的人。如果有什么错误，请一定指出，以免误导大家、也误导我。
本文来自：http://www.jianshu.com/users/320f9e8f7fc9/latest_articles
感谢您的关注。

最近有个需求一次要打包9个类型的App，而且常量和String.xml都有变量。虽然之前也是一直存在变量，但是每次也仅仅只打包一个。这让我每次改变量，打包9个。要是以后每次都打包9次，我得疯了。

根据之前的了解，gradle 应该是可以解决这个问题的。所以就仔细研究了一番。

<hr />
先放一个完整的 多渠道/多环境 打包的配置，然后再来讲解。
> **实现了：**
> 1. 不同环境，不同包名；
> 2. 不同环境，修改不同的 string.xml 资源文件；
> 3. 不同环境，修改指定的常量；
> 4. 不同环境，修改 AndroidManifest.xml 里渠道变量；
> 5. 不同环境，引用不同的 module。

先放一个完整的配置，可以参考：

	apply plugin: 'com.android.application'

	android {
		compileSdkVersion 22
		buildToolsVersion '22.0.1'

		// 签名文件
		signingConfigs {
			config {
				keyAlias 'lyl'
				keyPassword '123456'
				storeFile file('../lyl.jks')
				storePassword '123456'
			}
		}

		// 默认配置
		defaultConfig {
			//applicationId "com.lyl.app"
			minSdkVersion 16
			targetSdkVersion 22
			versionCode 1
			versionName "1.0.0"

			signingConfig signingConfigs.config
			multiDexEnabled true
		}

		// 多渠道/多环境 的不同配置
		productFlavors {
			dev {
				// 每个环境包名不同
				applicationId "com.lyl.dev"
				// 动态添加 string.xml 字段；
				// 注意，这里是添加，在 string.xml 不能有这个字段，会重名！！！
				resValue "string", "app_name", "dev_myapp"
				resValue "bool", "isrRank", 'false'
				// 动态修改 常量 字段
				buildConfigField "String", "ENVIRONMENT", '"dev"'
				// 修改 AndroidManifest.xml 里渠道变量
                manifestPlaceholders = [UMENG_CHANNEL_VALUE: "dev"]
			}
			stage {
				applicationId "com.lyl.stage"

				resValue "string", "app_name", "stage_myapp"
				resValue "bool", "isrRank", 'true'

				buildConfigField "String", "ENVIRONMENT", '"stage"'

				manifestPlaceholders = [UMENG_CHANNEL_VALUE: "stage"]
			}
			prod {
				applicationId "com.lyl.prod"

				resValue "string", "app_name", "myapp"
				resValue "bool", "isrRank", 'true'

				buildConfigField "String", "ENVIRONMENT", '"prod"'

				manifestPlaceholders = [UMENG_CHANNEL_VALUE: "prod"]
			}
		}

        dexOptions {
            incremental true
            // javaMaxHeapSize "4g"
        }

		//移除lint检测的error
		lintOptions {
			abortOnError false
		}

        def releaseTime() {
            return new Date().format("yyyy-MM-dd", TimeZone.getTimeZone("UTC"))
        }

		buildTypes {
			debug {
				signingConfig signingConfigs.config
			}

			release {
				buildConfigField("boolean", "LOG_DEBUG", "false")
				minifyEnabled false
				zipAlignEnabled true
				//移除无用的resource文件
				shrinkResources true
				proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
				signingConfig signingConfigs.config

				// 批量打包
				applicationVariants.all { variant ->
					variant.outputs.each { output ->
						def outputFile = output.outputFile
						if (outputFile != null && outputFile.name.endsWith('.apk')) {
							//输出apk名称为：渠道名_版本名_时间.apk
							def fileName = "${variant.productFlavors[0].name}_v${defaultConfig.versionName}_${releaseTime()}.apk"
							output.outputFile = new File(outputFile.parent, fileName)
						}
					}
				}
			}
		}
	}

	repositories {
		mavenCentral()
	}

	dependencies {
		compile 'com.facebook.android:facebook-android-sdk:4.0.0'
		compile project(':qrscan')
		compile 'com.android.support:appcompat-v7:22.0.0'
		compile 'com.google.code.gson:gson:2.3'
		compile files('libs/android-async-http-1.4.6.jar')
		compile 'com.google.android.gms:play-services:7.5.0'
		compile 'com.android.support:support-annotations:22.1.1'
		compile 'com.github.bumptech.glide:glide:3.7.0'
		compile 'de.hdodenhof:circleimageview:2.1.0'
	}

<hr />
接下来我们来详细看看修改特定的字段。

**不同环境的设置基本都是在 productFlavors 里设置的，
而且在里面你想添加多少个环境都可以。**

#### 1. 不同环境，不同包名；

	productFlavors {
		dev {
			applicationId "com.lyl.dev"
		}

		stage {
			applicationId "com.lyl.stage"
		}

		prod {
			applicationId "com.lyl.prod"
		}
	}

这里注意，在 defaultConfig 中，大家应该都是写了个默认的 applicationId 的。 <br />
经测试，productFlavors 设置的不同环境包名会覆盖 defaultConfig 里面的设置， <br />
所以我们可以推测，它执行的顺序应该是先执行默认的，然后在执行分渠道的，如果冲突，会覆盖处理，这也很符合逻辑。

<br />
#### 2. 不同环境，添加 string.xml 资源文件；
利用 resValue 来定义资源的值，顾名思义 res 底下的内容应该都可以创建，最后用 R.xxx.xxx 来引用。 <br />
如下就根据不同的类型，添加了不同的 app_name 字段，以及定义了 布尔值，可以通过 R.string.app_name 来引用。

##### 注意，这里是**添加**，是在 string.xml 里面添加了一个字段app_name，所以在现有的 string.xml 中不能有这个字段，否则会报错！！！

    productFlavors {
		dev {
			resValue "string", "app_name", "dev_myapp"
			resValue "bool", "isrRank", 'false'
		}
		stage {
			resValue "string", "app_name", "stage_myapp"
			resValue "bool", "isrRank", 'true'
		}
		prod {
			resValue "string", "app_name", "myapp"
			resValue "bool", "isrRank", 'true'
		}
	}

通过以上我们大概可以推测出 color、dimen 也可以通过类似的方法添加。

<br />
#### 3. 不同环境，动态修改指定的常量；
使用 BuildConfig 的变量。
##### ①定义字段
当我们定义如下字段之后，编译后自动生成文件，在 app/build/source/BuildConfig/dev/com.lyl.dev/BuildConfig 目录，<br />
打开这个文件，我们就能看到我们所定义的字段了。

    productFlavors {
		dev {
			buildConfigField "String", "ENVIRONMENT", '"dev"'
		}
		stage {
			buildConfigField "String", "ENVIRONMENT", '"stage"'
		}
		prod {
			buildConfigField "String", "ENVIRONMENT", '"prod"'
		}
	}

##### ②引用字段
在我们自己的任意的类中，来直接通过 BuildConfig 就可以调用我们定义的字段。

	public class Constants {
		public static final String ENVIRONMENT = BuildConfig.ENVIRONMENT;

	}


**注意：**这里有个小细节，看其中第三个参数，是先用了“'”，然后在用了“"”，这种语法在 Java 里可能比较陌生，但是在很多其他语言中，这种用法是很常见的。<br />
它的意思是 "dev" 这个整体是属于一个字符串，至于为什么要这么写，你把单引号去掉，然后去 app/build/source/BuildConfig/dev/com.lyl.dev/BuildConfig 这个文件看一看就知道了。


<br />
#### 4. 不同环境，修改 AndroidManifest.xml 里渠道变量

##### ①在 AndroidManifest.xml 里添加渠道变量

	<application
		android:icon="${app_icon}"
		android:label="@string/app_name"
		android:theme="@style/AppTheme">
		...
		<meta-data
			android:name="UMENG_CHANNEL"
			android:value="${ENVIRONMENT}" />
		...
	</application>



##### ②在 build.gradle 设置 productFlavors

    productFlavors {
	    dev {
            manifestPlaceholders = [ENVIRONMENT: "dev",
									app_icon   : "@drawable/icon_dev"]
		}
		stage {
			manifestPlaceholders = [ENVIRONMENT: "stage",
									app_icon   : "@drawable/icon_stage"]
		}
		prod {
			manifestPlaceholders = [ENVIRONMENT: "prod",
									app_icon   : "@drawable/icon_prod"]
		}
	}

这样我们可以在不同环境使用不同的 key 值。


<br />
#### 5. 不同环境，引用不同的 module
这个就很强大了，根据不同的环境，引用对应的 module。<br />
你可以替换大量的图片，string，color，vaule等等。<br />

首先，要建立跟渠道对应的 module，然后再引用。<br />
引用方式如下：

    dependencies {
        compile fileTree(dir: 'libs', include: ['*.jar'])

        // 引用本的项目
        devCompile project(':devModule')
        stageCompile project(':stageModule')
        prodCompile project(':prodModule')

        // 也可以分渠道引用网络的。因为这里都相同，所以地址也就都一样了
        devCompile 'com.roughike:bottom-bar:2.0.2'
        stageCompile 'com.roughike:bottom-bar:2.0.2'
        prodCompile 'com.roughike:bottom-bar:2.0.2'
    }

**xxxCompile 代表 各个渠道的名称。**
然后把需要分渠道的文件，放到不同的 module 里面，把主项目的文件删掉。
**千万注意：**如果这样做了，每次需要引用的时候，在各个渠道的 module 里面都必须要放置文件哦，不然会找不到资源。
通过这种方式可以替换整套素材资源，具体如何使用还得看项目需求。


<br />
<br />
**通过以上方式，我们基本可以 通过 gradle 动态设定应用标题，应用图标，替换常量，设置不同包名，更改渠道等等。**
<hr />

#### 打包编译
最后，做完所有的配置之后，然后就是打包操作了。

###### 打包某一个（日常编译）
因为 buildTypes 里面有两种，所以每个渠道都会有两种模式。

![](http://upload-images.jianshu.io/upload_images/1689895-77780c468c4d8159.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


###### 打包所有的，就是正常打包流程
如图所示：

![图片来源网络](http://upload-images.jianshu.io/upload_images/1689895-54d7be7aee3a17db.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![图片来源网络](http://upload-images.jianshu.io/upload_images/1689895-84f295265b8a6245.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<br />
打包完成之后，然后就可以在我们指定的目录下，看到我们所生成的apk包。

***
#### 使用 local.properties 存放私密配置
以上就可以基本实现 gradle 的设置，但是如果我们要将我们的项目上传到 Github ，或者要将项目发送给别人。上面有些私密的东西就会被别人看到。比如：.jks 文件的密码。<br />
在项目跟目录下，有个 local.properties 文件，我们可以使用它来存放一些私密的属性，然后在 gradle 中读取，而 local.properties 文件不需要上传。<br />
local.properties 文件里设置如下：

	sdk.dir=D\:\\Android\\android-sdk

	gaodeKey=e348025dd034d1c347dd0345e34802

	keyPassword=123456


#### 在 build.gradle 读取 local.properties 字段信息

	// 加载 local.properties 资源
	Properties properties = new Properties()
	InputStream inputStream = project.rootProject.file('local.properties').newDataInputStream() ;
	properties.load( inputStream )

	android {
		...

		// 签名文件
		signingConfigs {
			config {
				keyAlias 'lyl'
				// 获取 local.properties 字段信息
				keyPassword properties.getProperty( 'keyPassword' )
				storeFile file('../lyl.jks')
				storePassword properties.getProperty( 'keyPassword' )
			}
		}

		...
	}

这样就可以将自己想要隐藏的一些数据隐藏起来。
<hr />
#### 可能加快 Android Studio 编译的办法

###### 1. 在根目录的 build.gradle 里加上如下代码：

    allprojects {
        // 加上这个
        tasks.withType(JavaCompile) {
            //使在一个单独的守护进程编译
            options.fork = true
            //增量编译
            options.incremental = true
        }

        repositories {
            jcenter()
        }
    }

###### 2. 在 app 级别下 build.gradle 里 加上

    android {
        dexOptions {
            incremental true
        }
    }

***

最后放上一个多渠道的项目地址，可以参考：<br />
https://github.com/Wing-Li/boon


OK。<br />
如果本文有什么问题，请一定指出。<br />
O(∩_∩)O