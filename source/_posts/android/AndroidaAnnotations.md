title: AndroidAnnotations
date: 2017/10/11 10:16:21
categories: android
tags:
- AndroidAnnotations
- 注解
- 反射
---

>AndroidStudio中使用AndroidAnnotations

![mark](http://p2fqmxqeh.bkt.clouddn.com/blog/180112/85mgmca86D.jpg?imageslim)

<!--more-->
Building Project Gradle
-----------------------
Gradle2.3.0及以后的版本构建支持annotation processors，使用方法如下
project build.gradle

    buildscript {
        repositories {
          mavenCentral()
        }
        dependencies {
            // replace with the current version of the Android plugin
            classpath 'com.android.tools.build:gradle:2.3.0'
        }
    }

Module build.gradle

    apply plugin: 'com.android.application'
    def AAVersion = 'XXX'

    dependencies {
        annotationProcessor "org.androidannotations:androidannotations:$AAVersion"
        compile "org.androidannotations:androidannotations-api:$AAVersion"
    }

**Gradle2.3.0之前可以使用gradle插件android-apt,来加载annotation processors，使用方法如下**
project build.gradle

    buildscript {
        repositories {
          mavenCentral()
        }
        dependencies {
            // replace with the current version of the Android plugin
            classpath 'com.android.tools.build:gradle:1.5.0'
            // replace with the current version of the android-apt plugin
            classpath 'com.neenbedankt.gradle.plugins:android-apt:1.8'
        }
    }

Module build.gradle

    apply plugin: 'com.android.application'
    apply plugin: 'android-apt'
    def AAVersion = 'XXX'

    dependencies {
        apt "org.androidannotations:androidannotations:$AAVersion"
        compile "org.androidannotations:androidannotations-api:$AAVersion"
    }


**AndroidAnnotations中的注解**
==========================
**强化Application**
-------------
**@EApplication**

    @EApplication
    public class MyApplication extends Application{
    }

@App
可以用在@EActivity,@EBean中,注入Application;

    @EActivity
    public class MyActivity extends Activity{
        @App
        MyApplication application;
    }

**强化Activity**
--------------

**@EActivity**

    @EActivity(R.layout.activity_my)
    public class MyActivity extends Activity {
      // ...
    }

生成下面这个子类，在同一个包下面，但在不同的源文件中：

    public final class MyActivity_ extends MyActivity {
      // ...
    }

在AndroidManifest.xml

    <activity android:name=".MyActivity_"/>

启动MyActicvity

    MyActivity_.intent(context).withOptions(bundle).start().withAnimation(enterAnimRes, exitAnimRes);

启动MyActivity带返回值的

    MyActivity_.intent(context).startForResult(REQUEST_CODE);

**@onActivityResult**

    @OnActivityResult(REQUEST_CODE)
    void onResult(int resultCode, Intent data) {
    }
    @OnActivityResult(REQUEST_CODE)
    void onResult(@OnActivityResult.Extra String strVal, @OnActivityResult.Extra int intVal) {
    }

**onBackPressed()**
使用@EActivity后可以使用onBackPressed()方法，处理回退键响应

    public final class MyActivity_ extends MyActivity
    {
        @Override
        public boolean onKeyDown(int keyCode, KeyEvent event) {
            if (((SdkVersionHelper.getSdkInt()< 5)&&(keyCode == KeyEvent.KEYCODE_BACK))&&(event.getRepeatCount() == 0)) {
                onBackPressed();
            }
            return super.onKeyDown(keyCode, event);
        }
    }

**@WindowFeature**

    @WindowFeature({ Window.FEATURE_NO_TITLE, Window.FEATURE_INDETERMINATE_PROGRESS })
    @EActivity
    public class MyActivity extends Activity {
    }

**@Fullscreen**

    @Fullscreen
    public class MyActivity extends Activity {
    }

**强化Fragment**
--------------

**@Efragment**

    @EFragment(R.layout.my_fragment_layout)
    public class MyFragment extends Fragment{
    }

如果重写onCreateView，且返回的该方法返回的view不为空，采用暴力注入布局使得布局生效：

    @Efragment(value = R.layout.my_custom_layout, forceLayoutInjection = true)
    public class MyFragment extends ListFragment{
    }

**@FragmentArg**

    @EFragment
    public class MyFragment extends Fragment {
      @FragmentArg("myStringArgument")
      String myMessage;

      @FragmentArg
      String anotherStringArgument;

      @FragmentArg("myDateExtra")
      Date myDateArgumentWithDefaultValue = new Date();
    }

使用下面这种方式就可以传值了:

    MyFragment myFragment = MyFragment_.builder()
        .myMessage("hello")
        .anotherStringArgument("world")
        .build();

**强化Service**
-------------

**@EService**

    @EService
    public class MyService extends Service {
    }

强化后可以使用一下方法启动和停止Service

    MyService_.intent(getApplication()).start();
    MyService_.intent(getApplication()).stop();

**@EIntentService**

    @EIntentService
    public class MyIntentService extends IntentService {
    }

**@ServiceAction**

    @EIntentService
    public class MyIntentService extends IntentService {
        @ServiceAction
        void myAction(String param) {
            // ...
        }
    }

通过如下方法使用

    MyIntentService_.intent(getApplication())
        .myAction("test")
        .start();

**@SystemService**
可以作为方法注解，方法参数注解

    @SystemService
    NotificationManager notificationManager;
    @SystemService
    AudioManager audioManager;

**强化BroadcastReceivers**
------------------------

**@EReceiver**

@EReceiver
    public class MyReceiver extends BroadcastReceiver {
    }

**@ReceiverAction**
默认使用方法名作为Action name,也可以重新使用注解的值作为Action;

    @EReceiver
    public class MyReceiver extends BroadcastReceiver {
        @ReceiverAction("BROADCAST_ACTION_NAME")
        void mySimpleAction(Intent intent) {
            // ...
        }

        @ReceiverAction
        void myAction(@ReceiverAction.Extra String valueString, Context context) {
            // ...
        }

        @ReceiverAction
        void anotherAction(@ReceiverAction.Extra("specialExtraName") String valueString, @ReceiverAction.Extra long valueLong) {
            // ...
        }

        @ReceiverAction(actions = android.content.Intent.VIEW, dataSchemes = "http")
        protected void onHttp() {
          // Will be called when an App wants to open a http website but not for https.
        }

        @ReceiverAction(actions = android.content.Intent.VIEW, dataSchemes = {"http", "https"})
        protected void onHttps() {
          // Will be called when an App wants to open a http or https website.
      }
    }

**@Receiver**

    @EActivity
    public class MyActivity extends Activity {
      @Receiver(actions = "org.androidannotations.ACTION_1")
      protected void onAction1() {
      }
    }

**强化contentproviders**
----------------------

**@EProvider**

    @EProvider
    public class MyContentProvider extends ContentProvider {
    }

**强化Custom View、Custom Classes**
--------------------------------

**@EView**
在layout中使用的时候别忘记“_”;

    @EView
    public class CustomButton extends Button {
    }

在代码中创建可以使用下面方法:

    CustomButton button = CustomButton_.build(context);

**@EViewGroup**
在layout中使用的时候别忘记“_”;

    @EViewGroup(R.layout.title_with_subtitle)
    public class TitleWithSubtitle extends RelativeLayout {
    }

**@EBean**

    @EBean
    public class MyClass {
    }

默认为每次注入创建新的实例，单例模式使用下面的方法

    @EBean(scope = Scope.Singleton)
    public class MySingleton {
    }

**@Bean**
注入

    @Bean
    MyClass mClass;

**强化SharedPreferences**
-----------------------

**@SharePref**

    @SharedPref
    public interface MyPrefs {
        // The field name will have default value "John"
        @DefaultString("John")
        String name();

        // The field age will have default value 42
        @DefaultInt(42)
        int age();

        // The field lastUpdated will have default value 0
        long lastUpdated();
    }

创建整个application共享的sharedPref,使用：

    @SharedPref(value=SharedPref.Scope.UNIQUE)
    public interface MyPrefs {
    }

**@Pref**
注入

    @Pref
    MyPrefs_ myPrefs;

可以通过下面的方式使用SharedPreferences
简单编辑：

    myPrefs.name().put("John");

批量编辑：

    myPrefs.edit()
      .name()
      .put("John")
      .age()
      .put(42)
      .apply();

清除preferences:

    myPrefs.clear();

检查一个值是否存在:

    boolean nameExists = myPrefs.name().exists();

获取一个值:

    long lastUpdated = myPrefs.lastUpdated().get();

获取值并提供默认值:

    long now = System.currentTimeMillis();
    long lastUpdated = myPrefs.lastUpdated().getOr(now);

可以设置sharePreference的默认值通过以下注解:
**@DefaultString**
**@DefaultInt**
**@DefaultLong**
**@DefaultRes**
通过注解的值设置默认值，也可以直接使用方法名作为值

    @SharedPref
    public interface MyPrefs {
        @DefaultRes(R.string.defaultPrefName)
        String resourceName();

        @DefaultRes // uses 'R.string.defaultPrefAge' to set default value
        String defaultPrefAge();
    }


**DataBinding Support**
-----------------------

**@DataBound**
**@BindingObject**

    @DataBound
    @EActivity(R.layout.main_activity)
    public class MainActivity extends Activity {

        @BindingObject
        MainActivityBinding binding;

        @AfterViews
        void bindData() {
            binding.setTitle("Your title"); // example binding
        }

        @BindingObject
        void injectBinding(MainActivityBinding binding) {
            // assign binding
        }

        void injectBinding2(@BindingObject MainActivityBinding binding) {
            // assign binding
        }
    }

**使用示例**
----
**Before**

    public class BookmarksToClipboardActivity extends Activity {

      BookmarkAdapter adapter;

      ListView bookmarkList;

      EditText search;

      BookmarkApplication application;

      Animation fadeIn;

      ClipboardManager clipboardManager;

      @Override
      protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        requestWindowFeature(Window.FEATURE_NO_TITLE);
        getWindow().setFlags(FLAG_FULLSCREEN, FLAG_FULLSCREEN);

        setContentView(R.layout.bookmarks);

        bookmarkList = (ListView) findViewById(R.id.bookmarkList);
        search = (EditText) findViewById(R.id.search);
        application = (BookmarkApplication) getApplication();
        fadeIn = AnimationUtils.loadAnimation(this, anim.fade_in);
        clipboardManager = (ClipboardManager) getSystemService(CLIPBOARD_SERVICE);

        View updateBookmarksButton1 = findViewById(R.id.updateBookmarksButton1);
        updateBookmarksButton1.setOnClickListener(new OnClickListener() {

          @Override
          public void onClick(View v) {
            updateBookmarksClicked();
          }
        });

        View updateBookmarksButton2 = findViewById(R.id.updateBookmarksButton2);
        updateBookmarksButton2.setOnClickListener(new OnClickListener() {

          @Override
          public void onClick(View v) {
            updateBookmarksClicked();
          }
        });

        bookmarkList.setOnItemClickListener(new OnItemClickListener() {

          @Override
          public void onItemClick(AdapterView<?> p, View v, int pos, long id) {
            Bookmark selectedBookmark = (Bookmark) p.getAdapter().getItem(pos);
            bookmarkListItemClicked(selectedBookmark);
          }
        });

        initBookmarkList();
      }

      void initBookmarkList() {
        adapter = new BookmarkAdapter(this);
        bookmarkList.setAdapter(adapter);
      }

      void updateBookmarksClicked() {
        UpdateBookmarksTask task = new UpdateBookmarksTask();

        task.execute(search.getText().toString(), application.getUserId());
      }

      private static final String BOOKMARK_URL = //
      "http://www.bookmarks.com/bookmarks/{userId}?search={search}";


      class UpdateBookmarksTask extends AsyncTask<String, Void, Bookmarks> {

        @Override
        protected Bookmarks doInBackground(String... params) {
          String searchString = params[0];
          String userId = params[1];

          RestTemplate client = new RestTemplate();
          HashMap<String, Object> args = new HashMap<String, Object>();
          args.put("search", searchString);
          args.put("userId", userId);
          HttpHeaders httpHeaders = new HttpHeaders();
          HttpEntity<Bookmarks> request = new HttpEntity<Bookmarks>(httpHeaders);
          ResponseEntity<Bookmarks> response = client.exchange( //
              BOOKMARK_URL, HttpMethod.GET, request, Bookmarks.class, args);
          Bookmarks bookmarks = response.getBody();

          return bookmarks;
        }

        @Override
        protected void onPostExecute(Bookmarks result) {
          adapter.updateBookmarks(result);
          bookmarkList.startAnimation(fadeIn);
        }

      }

      void bookmarkListItemClicked(Bookmark selectedBookmark) {
        clipboardManager.setText(selectedBookmark.getUrl());
      }

    }

**After**

    @Fullscreen
    @EActivity(R.layout.bookmarks)
    @WindowFeature(Window.FEATURE_NO_TITLE)
    public class BookmarksToClipboardActivity extends Activity {

      BookmarkAdapter adapter;

      @ViewById
      ListView bookmarkList;

      @ViewById
      EditText search;

      @App
      BookmarkApplication application;

      @RestService
      BookmarkClient restClient;

      @AnimationRes
      Animation fadeIn;

      @SystemService
      ClipboardManager clipboardManager;

      @AfterViews
      void initBookmarkList() {
        adapter = new BookmarkAdapter(this);
        bookmarkList.setAdapter(adapter);
      }

      @Click({R.id.updateBookmarksButton1, R.id.updateBookmarksButton2})
      void updateBookmarksClicked() {
        searchAsync(search.getText().toString(), application.getUserId());
      }

      @Background
      void searchAsync(String searchString, String userId) {
        Bookmarks bookmarks = restClient.getBookmarks(searchString, userId);
        updateBookmarks(bookmarks);
      }

      @UiThread
      void updateBookmarks(Bookmarks bookmarks) {
        adapter.updateBookmarks(bookmarks);
        bookmarkList.startAnimation(fadeIn);
      }

      @ItemClick
      void bookmarkListItemClicked(Bookmark selectedBookmark) {
        clipboardManager.setText(selectedBookmark.getUrl());
      }

    }
    @Rest("http://www.bookmarks.com")
    public interface BookmarkClient {

      @Get("/bookmarks/{userId}?search={search}")
      Bookmarks getBookmarks(@Path String search, @Path String userId);

    }
