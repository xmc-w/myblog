title: Dagger2-Android依赖注入框架的学习
date: 2017/12/12 17:56:43
categories: android
tags:
- Dagger2
- 依赖注入
- 解耦
---

>官方的使用文档
<https://google.github.io/dagger//>
转载并修改自，天天博客
<http://www.cnblogs.com/tiantianbyconan/p/5092525.html>
原文翻译自：
<http://frogermcs.github.io/dependency-injection-with-dagger-2-the-api>
关于Dagger2的作用，以及依赖注入就不做介绍了，下面这边文章介绍的很好：
<http://frogermcs.github.io/dependency-injection-with-dagger-2-introdution-to-di>
我们还是通过学习如何使用它，来体验它给android编程带来的优点。

![mark](http://p2fqmxqeh.bkt.clouddn.com/blog/180112/B899D5jcd2.jpg?imageslim)

<!--more-->
***
这是Dagger2的API:

    public @interface Component {
        Class<?>[] modules() default {};
        Class<?>[] dependencies() default {};
    }
    
    public @interface Subcomponent {
        Class<?>[] modules() default {};
    }
    
    public @interface Module {
        Class<?>[] includes() default {};
    }
    
    public @interface Provides {
    }
    
    public @interface MapKey {
        boolean unwrapValue() default true;
    }
    
    public interface Lazy<T> {
        T get();
    }
    
还有在Dagger2中使用到的在JSR-330（java依赖注入的标准）中定义的其他元素：

    public @interface Inject {
    }
    
    public @interface Scope {
    }
    
    public @interface Qualifier {
    }

这API看上去十分的简单，但这个简单的API使用起来并不简单。

下面我们来学习这些API的使用

@Inject注解
---------

这个注解是Dagger2依赖注入框架(以下简称DI框架)中最重要的注解。是JSR-330标准的一部分，用来标记依赖注入框架应该提供的依赖关系，DI框架中有三种方式来提供依赖关系：
**Constructor injection构造方法注入**
@inject和类的构造方法一起使用:

    public class LoginActivityPresenter {
        
        private LoginActivity loginActivity;
        private UserDataStore userDataStore;
        private UserManager userManager;
        
        @Inject
        public LoginActivityPresenter(LoginActivity loginActivity,
                                      UserDataStore userDataStore,
                                      UserManager userManager) {
            this.loginActivity = loginActivity;
            this.userDataStore = userDataStore;
            this.userManager = userManager;
        }
    }

构造方法中的所有参数都通过依赖关系图获取。
在构造方法中使用@Inject注解也使得这个类成为依赖关系图的一部分。
这也表示它可以在需要的时候被注入，如（Fields injection）：

    public class LoginActivity extends BaseActivity {
    
        @Inject
        LoginActivityPresenter presenter;
       
        //...
    }

（这里需要注意的是我们不能给同一个类的多个构造函数添加@Inject注解）

**Fields injection字段注入**
@Inject来注解特定的字段：

    public class SplashActivity extends AppCompatActivity {
        
        @Inject
        LoginActivityPresenter presenter;
        @Inject
        AnalyticsManager analyticsManager;
        
        @Override
        protected void onCreate(Bundle bundle) {
            super.onCreate(bundle);
            getAppComponent().inject(this);
        }
    }

这种方式，注入过程中必须通过在我们的类中主动调用：

    public class SplashActivity extends AppCompatActivity {
        
        //...
        
        @Override 
        protected void onCreate(Bundle bundle) {
            super.onCreate(bundle);
            getAppComponent().inject(this);//Requested depenencies are injected in this moment
        }
    }

在这个调用之前，我们的依赖是null。
这里有一个限制，注入的依赖不能被private修饰。为什么呢？
简单解释下就是，在生成的代码中通过直接调用给他们赋值，如下：

    //This class is generated automatically by Dagger 2
    public final class SplashActivity_MembersInjector implements MembersInjector<SplashActivity> {
        //...
        @Override
        public void injectMembers(SplashActivity splashActivity) {
            if (splashActivity == null) {
                throw new NullPointerException("Cannot inject members into a null reference");
            }
            supertypeInjector.injectMembers(splashActivity);
            splashActivity.presenter = presenterProvider.get();
            splashActivity.analyticsManager = analyticsManagerProvider.get();
        }
    }

**Methods injection方法注入**
最后一种提供依赖的方法是通过对类中的public方法使用@Inject注解：

    public class LoginActivityPresenter {
        
        private LoginActivity loginActivity;
        
        @Inject 
        public LoginActivityPresenter(LoginActivity loginActivity) {
            this.loginActivity = loginActivity;
        }
    
        @Inject
        public void enableWatches(Watches watches) {
            watches.register(this); //Watches instance required fully constructed LoginActivityPresenter
        }
    }

所有方法的参数都通过依赖关系图获取。
有构造方法注入了，我们为什么还需要方法注入呢？
在这样一种情况下我们可能会用到，当我们需要把当前类的实例（this）作为依赖传递出去的时候，方法注入会在构造方法调用完成之后被调用，这也就保证了我们传递的this是一个构造完成的对象。

@Module注解
---------
@Module是Dagger2API的一部分，这个注解使用来表示提供依赖的类的-从而让Dagger知道需要的类是在哪里被构建的。

    @Module
    public class GithubApiModule {
        
        @Provides
        @Singleton
        OkHttpClient provideOkHttpClient() {
            OkHttpClient okHttpClient = new OkHttpClient();
            okHttpClient.setConnectTimeout(60 * 1000, TimeUnit.MILLISECONDS);
            okHttpClient.setReadTimeout(60 * 1000, TimeUnit.MILLISECONDS);
            return okHttpClient;
        }
    
        @Provides
        @Singleton
        RestAdapter provideRestAdapter(Application application, OkHttpClient okHttpClient) {
            RestAdapter.Builder builder = new RestAdapter.Builder();
            builder.setClient(new OkClient(okHttpClient))
                   .setEndpoint(application.getString(R.string.endpoint));
            return builder.build();
        }
    }

@Provides注解
--------------------
这个注解使用在@Module类中。在@Module类中@Provides用来标记哪些返回依赖的方法

    @Module
    public class GithubApiModule {
        
        //...
        
        @Provides   //This annotation means that method below provides dependency
        @Singleton
        RestAdapter provideRestAdapter(Application application, OkHttpClient okHttpClient) {
            RestAdapter.Builder builder = new RestAdapter.Builder();
            builder.setClient(new OkClient(okHttpClient))
                   .setEndpoint(application.getString(R.string.endpoint));
            return builder.build();
        }
    }

@Component注解
------------
这个注解用来构建一个把这些东西联系到一起的接口。在这个接口中，我们定义好要获取的依赖从哪些modules或者其它Components来。同时，在这个接口中我们还需要定义可见的依赖关系图（可见的依赖即可以被注入的）以及我们可以在哪些地方注入对象。可以认为@Component就像@Module和@Inject之间的桥梁一样。
在下面的实例代码中使用@Component构建的接口，使用了两个Modules，并且可以注入依赖到GithubClientApplication，并且标记了三个可见的依赖:

    @Singleton
    @Component(
        modules = {
            AppModule.class,
            GithubApiModule.class
        }
    )
    public interface AppComponent {
    
        void inject(GithubClientApplication githubClientApplication);
    
        Application getApplication();
    
        AnalyticsManager getAnalyticsManager();
    
        UserManager getUserManager();
    }

同样@Component可以依赖其他Component，并且必须得定义生命周期（这个会在下面scoping 说到）:

    @ActivityScope
    @Component(      
        modules = SplashActivityModule.class,
        dependencies = AppComponent.class
    )
    public interface SplashActivityComponent {
        SplashActivity inject(SplashActivity splashActivity);
    
        SplashActivityPresenter presenter();
    }

@Scope注解

    @Scope
    public @interface ActivityScope {
    }

JSR-330标准的另外一个部分。在Dagger2中，@scope是用来定义自定义scope的注解。自定义的scope可以使得依赖区别于单例，被single-instance注解的依赖只关联到Component的生命周期（不是整个application）。所有的自定义scope做同一件事情-他们持有对象的唯一实例。自定义的scope可以尽可能快的发现图表结构的问题。


先学习到这里，持续更新中... ...