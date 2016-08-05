---
layout: post
title: Retrofit源码解析
date: 2016-8-5
categories: blog
tags: [源码解析]
description: 源码解析
---   

**retrofit如何为我们的接口实现实例**

我们使用retrofit需要去定义一个接口，然后可以通过调用retrofit.create(IUserBiz.class);方法，得到一个接口的实例，最后通过该实例执行我们的操作，那么retrofit如何实现我们指定接口的实例呢？

其实原理是：动态代理。但是不要被动态代理这几个词吓唬到，Java中已经提供了非常简单的API帮助我们来实现动态代理。


```
return (T) Proxy.newProxyInstance(service.getClassLoader(), new Class<?>[] { service },
        new InvocationHandler() {
          private final Platform platform = Platform.get();

          @Override public Object invoke(Object proxy, Method method, Object... args)
              throws Throwable {
            // If the method is a method from Object then defer to normal invocation.
            if (method.getDeclaringClass() == Object.class) {
              return method.invoke(this, args);
            }
            if (platform.isDefaultMethod(method)) {
              return platform.invokeDefaultMethod(method, service, proxy, args);
            }
            ServiceMethod serviceMethod = loadServiceMethod(method);
            OkHttpCall okHttpCall = new OkHttpCall<>(serviceMethod, args);
            return serviceMethod.callAdapter.adapt(okHttpCall);
          }
        });
```


#### retrofit整体实现流程

**Retrofit的构建**

这里依然是通过构造者模式进行构建retrofit对象，好在其内部的成员变量比较少，我们直接看build()方法。

```
 public Retrofit build() {
      if (baseUrl == null) {
        throw new IllegalStateException("Base URL required.");
      }

      okhttp3.Call.Factory callFactory = this.callFactory;
      if (callFactory == null) {
        callFactory = new OkHttpClient();
      }

      Executor callbackExecutor = this.callbackExecutor;
      if (callbackExecutor == null) {
        callbackExecutor = platform.defaultCallbackExecutor();
      }

      // Make a defensive copy of the adapters and add the default Call adapter.
      List<CallAdapter.Factory> adapterFactories = new ArrayList<>(this.adapterFactories);
      adapterFactories.add(platform.defaultCallAdapterFactory(callbackExecutor));

      // Make a defensive copy of the converters.
      List<Converter.Factory> converterFactories = new ArrayList<>(this.converterFactories);

      return new Retrofit(callFactory, baseUrl, converterFactories, adapterFactories,
          callbackExecutor, validateEagerly);
    }
```


- baseUrl必须指定，这个是理所当然的；
- 然后可以看到如果不着急设置callFactory，则默认直接new OkHttpClient()，可见如果你需要对okhttpclient进行详细的设置，需要构建OkHttpClient对象，然后传入；
- 接下来是callbackExecutor，这个想一想大概是用来将回调传递到UI线程了，当然这里设计的比较巧妙，利用platform对象，对平台进行判断，判断主要是利用Class.forName("")进行查找，具体代码已经被放到文末，如果是Android平台，会自定义一个Executor对象，并且利用Looper.getMainLooper()实例化一个handler对象，在Executor内部通过handler.post(runnable)，ok，整理凭大脑应该能构思出来，暂不贴代码了。
- 接下来是adapterFactories，这个对象主要用于对Call进行转化，基本上不需要我们自己去自定义。
- 最后是converterFactories，该对象用于转化数据，例如将返回的responseBody转化为对象等；当然不仅仅是针对返回的数据，还能用于一般备注解的参数的转化例如@Body标识的对象做一些操作，后面遇到源码详细再描述。
ok，总体就这几个对象去构造retrofit，还算比较少的~


**具体Call构建流程**

我们构造完成retrofit，就可以利用retrofit.create方法去构建接口的实例了，上面我们已经分析了这个环节利用了动态代理，而且我们也分析了具体的Call的构建流程在invoke方法中，下面看代码：


```
public <T> T create(final Class<T> service) {
    Utils.validateServiceInterface(service);
    //...
    return (T) Proxy.newProxyInstance(service.getClassLoader(), new Class<?>[] { service },
        new InvocationHandler() {
           @Override 
          public Object invoke(Object proxy, Method method, Object... args){
            //...
            ServiceMethod serviceMethod = loadServiceMethod(method);
            OkHttpCall okHttpCall = new OkHttpCall<>(serviceMethod, args);
            return serviceMethod.callAdapter.adapt(okHttpCall);
          }
        });
}
```

主要也就三行代码，第一行是根据我们的method将其包装成ServiceMethod，第二行是通过ServiceMethod和方法的参数构造retrofit2.OkHttpCall对象，第三行是通过serviceMethod.callAdapter.adapt()方法，将OkHttpCall进行代理包装；

下面一个一个介绍：

- ServiceMethod应该是最复杂的一个类了，包含了将一个method转化为Call的所有的信息         

```
#Retrofit class
ServiceMethod loadServiceMethod(Method method) {
    ServiceMethod result;
    synchronized (serviceMethodCache) {
      result = serviceMethodCache.get(method);
      if (result == null) {
        result = new ServiceMethod.Builder(this, method).build();
        serviceMethodCache.put(method, result);
      }
    }
    return result;
  }

#ServiceMethod
public ServiceMethod build() {
      callAdapter = createCallAdapter();
      responseType = callAdapter.responseType();
      if (responseType == Response.class || responseType == okhttp3.Response.class) {
        throw methodError("'"
            + Utils.getRawType(responseType).getName()
            + "' is not a valid response body type. Did you mean ResponseBody?");
      }
      responseConverter = createResponseConverter();

      for (Annotation annotation : methodAnnotations) {
        parseMethodAnnotation(annotation);
      }


      int parameterCount = parameterAnnotationsArray.length;
      parameterHandlers = new ParameterHandler<?>[parameterCount];
      for (int p = 0; p < parameterCount; p++) {
        Type parameterType = parameterTypes[p];
        if (Utils.hasUnresolvableType(parameterType)) {
          throw parameterError(p, "Parameter type must not include a type variable or wildcard: %s",
              parameterType);
        }

        Annotation[] parameterAnnotations = parameterAnnotationsArray[p];
        if (parameterAnnotations == null) {
          throw parameterError(p, "No Retrofit annotation found.");
        }

        parameterHandlers[p] = parseParameter(p, parameterType, parameterAnnotations);
      }

      return new ServiceMethod<>(this);
    }
```                 

直接看build方法，首先拿到这个callAdapter最终拿到的是我们在构建retrofit里面时adapterFactories时添加的，即为：new ExecutorCallbackCall<>(callbackExecutor, call)，该ExecutorCallbackCall唯一做的事情就是将原本call的回调转发至UI线程。

接下来通过callAdapter.responseType()返回的是我们方法的实际类型，例如:Call<User>,则返回User类型，然后对该类型进行判断。

接下来是createResponseConverter拿到responseConverter对象，其当然也是根据我们构建retrofit时,addConverterFactory添加的ConverterFactory对象来寻找一个合适的返回，寻找的依据主要看该converter能否处理你编写方法的返回值类型，默认实现为BuiltInConverters，仅仅支持返回值的实际类型为ResponseBody和Void，也就说明了默认情况下，是不支持Call<User>这类类型的。

接下来就是对注解进行解析了，主要是对方法上的注解进行解析，那么可以拿到httpMethod以及初步的url（包含占位符）。

后面是对方法中参数中的注解进行解析，这一步会拿到很多的ParameterHandler对象，该对象在toRequest()构造Request的时候调用其apply方法。

ok，这里我们并没有去一行一行查看代码，其实意义也不太大，只要知道ServiceMethod主要用于将我们接口中的方法转化为一个Request对象，于是根据我们的接口返回值确定了responseConverter,解析我们方法上的注解拿到初步的url,解析我们参数上的注解拿到构建RequestBody所需的各种信息，最终调用toRequest的方法完成Request的构建。

**接下来看OkHttpCall的构建，构造函数仅仅是简单的赋值**           

```
OkHttpCall(ServiceMethod<T> serviceMethod, Object[] args) {
    this.serviceMethod = serviceMethod;
    this.args = args;
  }
```

**最后一步是serviceMethod.callAdapter.adapt(okHttpCall)**      

我们已经确定这个callAdapter是ExecutorCallAdapterFactory.get()对应代码为：

```
final class ExecutorCallAdapterFactory extends CallAdapter.Factory {
  final Executor callbackExecutor;

  ExecutorCallAdapterFactory(Executor callbackExecutor) {
    this.callbackExecutor = callbackExecutor;
  }

  @Override
  public CallAdapter<Call<?>> get(Type returnType, Annotation[] annotations, Retrofit retrofit) {
    if (getRawType(returnType) != Call.class) {
      return null;
    }
    final Type responseType = Utils.getCallResponseType(returnType);
    return new CallAdapter<Call<?>>() {
      @Override public Type responseType() {
        return responseType;
      }

      @Override public <R> Call<R> adapt(Call<R> call) {
        return new ExecutorCallbackCall<>(callbackExecutor, call);
      }
    };
  }
```

可以看到adapt返回的是ExecutorCallbackCall对象，继续往下看：         

```
static final class ExecutorCallbackCall<T> implements Call<T> {
    final Executor callbackExecutor;
    final Call<T> delegate;

    ExecutorCallbackCall(Executor callbackExecutor, Call<T> delegate) {
      this.callbackExecutor = callbackExecutor;
      this.delegate = delegate;
    }

    @Override public void enqueue(final Callback<T> callback) {
      if (callback == null) throw new NullPointerException("callback == null");

      delegate.enqueue(new Callback<T>() {
        @Override public void onResponse(Call<T> call, final Response<T> response) {
          callbackExecutor.execute(new Runnable() {
            @Override public void run() {
              if (delegate.isCanceled()) {
                // Emulate OkHttp's behavior of throwing/delivering an IOException on cancellation.
                callback.onFailure(ExecutorCallbackCall.this, new IOException("Canceled"));
              } else {
                callback.onResponse(ExecutorCallbackCall.this, response);
              }
            }
          });
        }

        @Override public void onFailure(Call<T> call, final Throwable t) {
          callbackExecutor.execute(new Runnable() {
            @Override public void run() {
              callback.onFailure(ExecutorCallbackCall.this, t);
            }
          });
        }
      });
    }
    @Override public Response<T> execute() throws IOException {
      return delegate.execute();
    }
  }
```


可以看出ExecutorCallbackCall仅仅是对Call对象进行封装，类似装饰者模式，只不过将其执行时的回调通过callbackExecutor进行回调到UI线程中去了。

**执行Call**

我们已经拿到了经过封装的ExecutorCallbackCall类型的call对象，实际上就是我们实际在写代码时拿到的call对象，那么我们一般会执行enqueue方法，看看源码是怎么做的

首先是ExecutorCallbackCall.enqueue方法，可以看到除了将onResponse和onFailure回调到UI线程，主要的操作还是delegate完成的，这个delegate实际上就是OkHttpCall对象，我们看它的enqueue方法


```
@Override
public void enqueue(final Callback<T> callback)
{
    okhttp3.Call call;
    Throwable failure;

    synchronized (this)
    {
        if (executed) throw new IllegalStateException("Already executed.");
        executed = true;

        try
        {
            call = rawCall = createRawCall();
        } catch (Throwable t)
        {
            failure = creationFailure = t;
        }
    }

    if (failure != null)
    {
        callback.onFailure(this, failure);
        return;
    }

    if (canceled)
    {
        call.cancel();
    }

    call.enqueue(new okhttp3.Callback()
    {
        @Override
        public void onResponse(okhttp3.Call call, okhttp3.Response rawResponse)
                throws IOException
        {
            Response<T> response;
            try
            {
                response = parseResponse(rawResponse);
            } catch (Throwable e)
            {
                callFailure(e);
                return;
            }
            callSuccess(response);
        }

        @Override
        public void onFailure(okhttp3.Call call, IOException e)
        {
            try
            {
                callback.onFailure(OkHttpCall.this, e);
            } catch (Throwable t)
            {
                t.printStackTrace();
            }
        }

        private void callFailure(Throwable e)
        {
            try
            {
                callback.onFailure(OkHttpCall.this, e);
            } catch (Throwable t)
            {
                t.printStackTrace();
            }
        }

        private void callSuccess(Response<T> response)
        {
            try
            {
                callback.onResponse(OkHttpCall.this, response);
            } catch (Throwable t)
            {
                t.printStackTrace();
            }
        }
    });
}
```

没有任何神奇的地方，内部实际上就是okhttp的Call对象，也是调用okhttp3.Call.enqueue方法。

中间对于okhttp3.Call的创建代码为：

```
private okhttp3.Call createRawCall() throws IOException
{
    Request request = serviceMethod.toRequest(args);
    okhttp3.Call call = serviceMethod.callFactory.newCall(request);
    if (call == null)
    {
        throw new NullPointerException("Call.Factory returned null.");
    }
    return call;
}
```


可以看到，通过serviceMethod.toRequest完成对request的构建，通过request去构造call对象，然后返回.

中间还涉及一个parseResponse方法，如果顺利的话，执行的代码如下：

```
Response<T> parseResponse(okhttp3.Response rawResponse) throws IOException
{
    ResponseBody rawBody = rawResponse.body();
    ExceptionCatchingRequestBody catchingBody = new ExceptionCatchingRequestBody(rawBody);

    T body = serviceMethod.toResponse(catchingBody);
    return Response.success(body, rawResponse);
```


通过serviceMethod对ResponseBody进行转化，然后返回，转化实际上就是通过responseConverter的convert方法。

```
#ServiceMethod
 T toResponse(ResponseBody body) throws IOException {
    return responseConverter.convert(body);
  }
```

到这里，我们整个源码的流程分析就差不多了，目的就掌握一个大体的原理和执行流程，了解下几个核心的类。

那么总结一下：

- 首先构造retrofit，几个核心的参数呢，主要就是baseurl,callFactory(默认okhttpclient),converterFactories,adapterFactories,excallbackExecutor。
- 然后通过create方法拿到接口的实现类，这里利用Java的Proxy类完成动态代理的相关代理
- 在invoke方法内部，拿到我们所声明的注解以及实参等，构造ServiceMethod，ServiceMethod中解析了大量的信息，最终可以通过toRequest构造出okhttp3.Request对象。有了okhttp3.Request对象就可以很自然的构建出okhttp3.call，最后calladapter对Call进行装饰返回。
- 拿到Call就可以执行enqueue或者execute方法了        


#### 自定义Converter.Factory              

**responseBodyConverter**

关于Converter.Factory，肯定是通过addConverterFactory设置的

```
Retrofit retrofit = new Retrofit.Builder()
    .addConverterFactory(GsonConverterFactory.create())
        .build();
```

该方法接受的是一个Converter.Factory factory对象

该对象呢，是一个抽象类，内部包含3个方法：

```
abstract class Factory {

    public Converter<ResponseBody, ?> responseBodyConverter(Type type, Annotation[] annotations,
        Retrofit retrofit) {
      return null;
    }


    public Converter<?, RequestBody> requestBodyConverter(Type type,
        Annotation[] parameterAnnotations, Annotation[] methodAnnotations, Retrofit retrofit) {
      return null;
    }


    public Converter<?, String> stringConverter(Type type, Annotation[] annotations,
        Retrofit retrofit) {
      return null;
    }
  }
```

可以看到呢，3个方法都是空方法而不是抽象的方法，也就表明了我们可以选择去实现其中的1个或多个方法，一般只需要关注requestBodyConverter和responseBodyConverter就可以了。

ok，我们先看如何自定义，最后再看GsonConverterFactory.create的源码。

先来个简单的，实现responseBodyConverter方法，看这个名字很好理解，就是将responseBody进行转化就可以了。

ok，这里呢，我们先看一下上述中我们使用的接口：

```
package com.zhy.retrofittest.userBiz;

public interface IUserBiz
{
    @GET("users")
    Call<List<User>> getUsers();

    @POST("users")
    Call<List<User>> getUsersBySort(@Query("sort") String sort);

    @GET("{username}")
    Call<User> getUser(@Path("username") String username);

    @POST("add")
    Call<List<User>> addUser(@Body User user);

    @POST("login")
    @FormUrlEncoded
    Call<User> login(@Field("username") String username, @Field("password") String password);

    @Multipart
    @POST("register")
    Call<User> registerUser(@Part("photos") RequestBody photos, @Part("username") RequestBody username, @Part("password") RequestBody password);

    @Multipart
    @POST("register")
    Call<User> registerUser(@PartMap Map<String, RequestBody> params,  @Part("password") RequestBody password);

    @GET("download")
    Call<ResponseBody> downloadTest();

}
```


假设哈，我们这里去掉retrofit构造时的GsonConverterFactory.create，自己实现一个Converter.Factory来做数据的转化工作。

首先我们解决responseBodyConverter，那么代码很简单，我们可以这么写：


```
public class UserConverterFactory extends Converter.Factory
{
    @Override
    public Converter<ResponseBody, ?> responseBodyConverter(Type type, Annotation[] annotations, Retrofit retrofit)
    {
        //根据type判断是否是自己能处理的类型，不能的话，return null ,交给后面的Converter.Factory
        return new UserConverter(type);
    }

}

public class UserResponseConverter<T> implements Converter<ResponseBody, T>
{
    private Type type;
    Gson gson = new Gson();

    public UserResponseConverter(Type type)
    {
        this.type = type;
    }

    @Override
    public T convert(ResponseBody responseBody) throws IOException
    {
        String result = responseBody.string();
        T users = gson.fromJson(result, type);
        return users;
    }
}
```

使用的时候呢，可以

```
Retrofit retrofit = new Retrofit.Builder()
.callFactory(new OkHttpClient()).baseUrl("http://example/springmvc_users/user/")
//.addConverterFactory(GsonConverterFactory.create())
.addConverterFactory(new UserConverterFactory())
            .build();
```

ok，这样的话，就可以完成我们的ReponseBody到List<User>或者User的转化了。

可以看出，我们这里用的依然是Gson，那么有些同学肯定不希望使用Gson就能实现，如果不使用Gson的话，一般需要针对具体的返回类型，比如我们针对返回List<User>或者User

你可以这么写：


```
package com.zhy.retrofittest.converter;
/**
 * Created by zhy on 16/4/30.
 */
public class UserResponseConverter<T> implements Converter<ResponseBody, T>
{
    private Type type;
    Gson gson = new Gson();

    public UserResponseConverter(Type type)
    {
        this.type = type;
    }

    @Override
    public T convert(ResponseBody responseBody) throws IOException
    {
        String result = responseBody.string();

        if (result.startsWith("["))
        {
            return (T) parseUsers(result);
        } else
        {
            return (T) parseUser(result);
        }
    }

    private User parseUser(String result)
    {
        JSONObject jsonObject = null;
        try
        {
            jsonObject = new JSONObject(result);
            User u = new User();
            u.setUsername(jsonObject.getString("username"));
            return u;
        } catch (JSONException e)
        {
            e.printStackTrace();
        }
        return null;
    }

    private List<User> parseUsers(String result)
    {
        List<User> users = new ArrayList<>();
        try
        {
            JSONArray jsonArray = new JSONArray(result);
            User u = null;
            for (int i = 0; i < jsonArray.length(); i++)
            {
                JSONObject jsonObject = jsonArray.getJSONObject(i);
                u = new User();
                u.setUsername(jsonObject.getString("username"));
                users.add(u);
            }
        } catch (JSONException e)
        {
            e.printStackTrace();
        }
        return users;
    }
}
```


这里简单读取了一个属性，大家肯定能看懂，这样就能满足返回值是Call<List<User>>或者Call<User>.

这里郑重提醒：如果你针对特定的类型去写Converter，一定要在UserConverterFactory#responseBodyConverter中对类型进行检查，发现不能处理的类型return null，这样的话，可以交给后面的Converter.Factory处理，比如本例我们可以按照下列方式检查:


```
public class UserConverterFactory extends Converter.Factory
{
    @Override
    public Converter<ResponseBody, ?> responseBodyConverter(Type type, Annotation[] annotations, Retrofit retrofit)
    {
        //根据type判断是否是自己能处理的类型，不能的话，return null ,交给后面的Converter.Factory
        if (type == User.class)//支持返回值是User
        {
            return new UserResponseConverter(type);
        }

        if (type instanceof ParameterizedType)//支持返回值是List<User>
        {
            Type rawType = ((ParameterizedType) type).getRawType();
            Type actualType = ((ParameterizedType) type).getActualTypeArguments()[0];
            if (rawType == List.class && actualType == User.class)
            {
                return new UserResponseConverter(type);
            }
        }
        return null;
    }

}
```

好了，到这呢responseBodyConverter方法告一段落了，谨记就是将reponseBody->返回值返回中的实际类型，例如Call<User>中的User;还有对于该converter不能处理的类型一定要返回null。


**requestBodyConverter**

ok，上面接口一大串方法呢，使用了我们的Converter之后，有个方法我们现在还是不支持的。

```
@POST("add")
Call<List<User>> addUser(@Body User user);
```

ok，这个@Body需要用到这个方法，叫做requestBodyConverter，根据参数转化为RequestBody，下面看下我们如何提供支持。

```
public class UserRequestBodyConverter<T> implements Converter<T, RequestBody>
{
    private Gson mGson = new Gson();
    @Override
    public RequestBody convert(T value) throws IOException
    {
        String string = mGson.toJson(value);
        return RequestBody.create(MediaType.parse("application/json; charset=UTF-8"),string);
    }
}
```

然后在UserConverterFactory中复写requestBodyConverter方法，返回即可：


```
public class UserConverterFactory extends Converter.Factory
{

    @Override
    public Converter<?, RequestBody> requestBodyConverter(Type type, Annotation[] parameterAnnotations, Annotation[] methodAnnotations, Retrofit retrofit)
    {
        return new UserRequestBodyConverter<>();
    }
}
```


这里偷了个懒，使用Gson将对象转化为json字符串了，如果你不喜欢使用框架，你可以选择拼接字符串，或者反射写一个支持任何对象的，反正就是对象->json字符串的转化。最后构造一个RequestBody返回即可。

ok,到这里，我相信如果你看的细致，自定义Converter.Factory是干嘛的，但是我还是要总结下：

- responseBodyConverter 主要是对应@Body注解，完成ResponseBody到实际的返回类型的转化，这个类型对应Call<XXX>里面的泛型XXX，其实@Part等注解也会需要responseBodyConverter，只不过我们的参数类型都是RequestBody，由默认的converter处理了。
- requestBodyConverter 完成对象到RequestBody的构造。
- 一定要注意，检查type如果不是自己能处理的类型，记得return null （因为可以添加多个，你不能处理return null ,还会去遍历后面的converter）.



**参考链接**   

[Retrofit2 完全解析 探索与okhttp之间的关系 - Hongyang - 博客频道 - CSDN.NET](http://blog.csdn.net/lmj623565791/article/details/51304204)