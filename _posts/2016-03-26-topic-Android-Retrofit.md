---
title: Retrofit 2.0非常简单的入门
excerpt: 翻译官方文档
categories:
  - topics
tag: Android  
---

> 如果本文帮助到你，本人不胜荣幸，如果浪费了你的时间，本人深感抱歉。
希望用最简单的大白话来帮助那些像我一样的人。如果有什么错误，请一定指出，以免误导大家、也误导我。
本文来自：http://www.jianshu.com/users/320f9e8f7fc9/latest_articles
感谢您的关注。

**声明：本文是转载的**
转自：http://blog.csdn.net/leilba/article/details/50685205 <br />
很好奇这么好的东西，为什么没人看。

Retrofit：Square提供的开源产品，为Android平台的应用提供一个类型安全的REST客户端。据说网络请求非常的快。 <br />
这个是官方文档（英文）：http://square.github.io/retrofit/

开始正文。
***


### 1. 介绍
Retrofit可以将你的HTTP API转化为JAVA的接口的形式。例如：
  <br />
	public interface GitHubService {
	  @GET("users/{user}/repos")
	  Call<List<Repo>> listRepos(@Path("user") String user);
	}


而Retrofit类能够生成对应接口的实现。例如：

	Retrofit retrofit = new Retrofit.Builder()
	    .baseUrl("https://api.github.com/")
	    .build();

	GitHubService service = retrofit.create(GitHubService.class);

每一个由接口返回的Call对象都可以与远程web服务端进行同步或者异步的HTTP请求通信。例如：

    Call<List<Repo>> repos = service.listRepos("octocat");

Retrofit使用注解来描述HTTP请求：  <br />
1.URL参数的替换和query参数的支持  <br />
2.对象转化为请求体（如：JSON，protocol buffers等）  <br />
3.多重请求体和文件上传  <br />
注意：本站仍在为2.0扩展新的API

### 2. API说明

Retrofit需要注解接口的请求方法和方法的参数来表明该请求需要怎么样的处理。

**2.1请求方法**
每一个方法必须要有一个HTTP注解来标明请求的方式和相对URL。有五种内置的注解方式：GET、POST、PUT、DELETE以及HEAD。资源的相对URL需要在注解里面明确给出：

      @GET("users/list")

当然你也可以将query参数直接写在URL里：

      @GET("users/list?sort=desc")

**2.2 URL操作**
一个请求的URL可以通过替换块和请求方法的参数来进行动态的更新。替换块是由被{}包裹起来的数字或字母组成的字符串构成的，相应的方法参数需要由@Path来注解同样的字符串。例如：

    @GET("group/{id}/users")
    Call<List<User>> groupList(@Path("id") int groupId);

Query参数也能同时添加。

	@GET("group/{id}/users")
	Call<List<User>> groupList(@Path("id") int groupId, @Query("sort") String sort);

复杂的query参数可以用Map来构建

	@GET("group/{id}/users")
	Call<List<User>> groupList(@Path("id") int groupId, @QueryMap Map<String, String> options);

****
**2.3请求主体**
能够通过@Body注解来指定一个方法作为HTTP请求主体

	@POST("users/new")
	Call<User> createUser(@Body User user);

这个参数对象会被Retrofit实例中的converter进行转化。如果没有给Retrofit实例添加任何converter的话则只有RequestBody可以作为参数使用。

**2.4 form encode 和 multipart**
方法也可以通过声明来发送form-encoded和multipart类型的数据。 <br />
可以通过@FormUrlEncoded注解方法来发送form-encoded的数据。每个键值对需要用@Filed来注解键名，随后的对象需要提供值。

	@FormUrlEncoded
	@POST("user/edit")
	Call<User> updateUser(@Field("first_name") String first, @Field("last_name") String last);


也可以通过@Multipart注解方法来发送Mutipart请求。每个部分需要使用@Part来注解。

	@Multipart@PUT("user/photo")
	Call<User> updateUser(@Part("photo") RequestBody photo, @Part("description") RequestBody description);

多个请求部分需要使用Retrofit的converter或者是自己实现 RequestBody来处理自己内部的数据序列化。

**2.5头部操作**
你可以通过使用@Headers注解来设置请求静态头。

	@Headers("Cache-Control: max-age=640000")
	@GET("widget/list")
	Call<List<Widget>> widgetList();

<br />
 
	@Headers({
	    "Accept: application/vnd.github.v3.full+json",
	    "User-Agent: Retrofit-Sample-App"
	})
	@GET("users/{username}")
	Call<User> getUser(@Path("username") String username);

注意的是头部参数并不会相互覆盖，同一个名称的所有头参数都会被包含进请求里面。 <br />
当然你可以通过 @Header 注解来动态更新请求头。一个相应的参数必须提供给 @Header 注解。如果这个值是空（null）的话，那么这个头部参数就会被忽略。否则的话， 值的 toString 方法将会被调用，并且使用调用结果。

	@GET("user")
	Call<User> getUser(@Header("Authorization") String authorization)

当然你可以通过[OkHttp interceptor](https://github.com/square/okhttp/wiki/Interceptors)来指定每一个需要的头部参数。

**2.6同步 VS 异步**
你可以同步或者异步地来执行实例。每个示例只能使用一次，但是可以使用 clone() 来创建一个可以使用的新的实例。 <br />
在Android环境中，callback将会在主线程中执行；而在JVM环境中，callback将会在和Http请求的同一个线程中执行。

### 3. Retrofit配置
Retrofit类会通过你定义的API接口转化为可调用的对象。默认情况下，Retrofit会返还给你合理的默认值，但也允许你进行指定。

**3.1转化器（Converters）**
默认情况下，Retrofit只能将HTTP体反序列化为OKHttp的 ResonseBody 类型，而且只能接收 RequestBody类型作为 @Body。 <br /> 
转化器的加入可以用于支持其他的类型。以下六个同级模块采用了常用的序列化库来为你提供方便。
> 
[Gson](https://github.com/google/gson): com.squareup.retrofit2:converter-gson  <br />
[Jackson](http://wiki.fasterxml.com/JacksonHome): com.squareup.retrofit2:converter-jackson  <br />
[Moshi](https://github.com/square/moshi/): com.squareup.retrofit2:converter-moshi  <br />
[Protobuf](https://developers.google.com/protocol-buffers/): com.squareup.retrofit2:converter-protobuf  <br />
[Wire](https://github.com/square/wire): com.squareup.retrofit2:converter-wire  <br />
[Simple XML](http://simple.sourceforge.net/): com.squareup.retrofit2:converter-simplexml  <br />
Scalars (primitives, boxed, and String): com.squareup.retrofit2:converter-scalars

下面提供一个使用GsonConverterFactory类生成 GitHubService的接口实现gson反序列化的例子。

	Retrofit retrofit = new Retrofit.Builder()
	    .baseUrl("https://api.github.com")
	    .addConverterFactory(GsonConverterFactory.create())
	    .build();

	GitHubService service = retrofit.create(GitHubService.class);

**3.2自定义转化器**
如果你需要与没有使用Retrofit提供的内容格式的API进行交互的话或者是你希望使用一个不同的库来实现现有的格式，你也可以轻松创建使用自己的转化器。你需要创建一个继承自Converter.Factory的类并且在构建适配器的时候加入到实例里面。

### 4. 下载
[↓ Latest JAR](https://search.maven.org/remote_content?g=com.squareup.retrofit2&a=retrofit&v=LATEST)

源代码、例子和网站在 [available on GitHub](http://github.com/square/retrofit)

**4.1 maven**

	<dependency>
	  <groupId>com.squareup.retrofit2</groupId>
	  <artifactId>retrofit</artifactId>
	  <version>2.0.0</version>
	</dependency>

**4.2 gradle**

      compile 'com.squareup.retrofit2:retrofit:2.0.0'

Retrofit支持最低 Java7 和 Android 2.3

**4.3 混淆**
如果你的工程中使用了代码混淆，那么你的配置中需要添加一下的几行

	-dontwarn retrofit2.**
	-keep class retrofit2.** { *; }
	-keepattributes Signature
	-keepattributes Exceptions