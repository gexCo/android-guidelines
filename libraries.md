# Android Libraries

## [AutoValue](https://github.com/google/auto/tree/master/value)


## [Butterknife](https://github.com/JakeWharton/butterknife)
Butterknife library is used for view injection in Android using annotations.

| Annotation | Description | Example |
| --- | --- | --- |
| `@BindView` | Binds view object such as TextView. | `@BindView(R.id.text_name)`<br>`TextView textName` |
| `@onClick` | Attach click event on an element.<br>Once this is used, we do not have to declare the button `@BindView(R.id.btn1) Button btn1` anymore. |   `@onClick(R.id.btn1, R.id.btn2)`<br>`void onButtonClick(View view)` 
| `@BindDrawable` | Binds drawable element. | `@BindDrawable(R.mipmap.ic_launcher)`<br>`Drawable logo` |
| `@BindString` | Binds string resource. | `@BindString(R.string.name)`<br>`String name` |
| `@BindColor` | Binds color resource | `@BindColor(R.color.colorPrimary)`<br>`int colorBar`|
| `@BindAnim` | Binds anim resource | `@BindAnim(R.anim.move_left)`<br>`Animation animMoveLeft` |

### Gradle
#### build.gradle(Project)
```gradle
buildscript {
    // ...
    dependencies {
        classpath 'com.neenbedankt.gradle.plugins:android-apt:1.8'
    }
}
```

#### build.gradle(Module)
```gradle
apply plugin: 'com.neenbedankt.android-apt'

//...

dependencies {
    compile 'com.jakewharton:butterknife:8.8.1'
    apt 'com.jakewharton:butterknife-compiler:8.8.1'
}
```

### Activity
Call `ButterKnife.bind(this)` in `onCreate()` method after `setContentView()` is called.
```java
public class MainActivity extends AppCompatActivity {
  
    @BindView(R.id.input_name)
    EditText inputName;
    
    @BindColor(R.color.colorPrimaryDark)
    int colorTitle;
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        ButterKnife.bind(this);
        
        inputName.setTextColor(colorTitle);
    }
 
    @OnClick(R.id.btn_1, R.id.btn_2)
    public void onButtonClick(View view) {
       switch(view.getId()) {
          // ...
       }
    }
}
```

### Fragment
The usage is similar to Activity but we have to unbind to free up some memory space.
```java
public class MyFragment extends Fragment {
 
    @BindView(R.id.input_name)
    EditText inputName;
    
    Unbinder unbinder;
 
    public MyFragment() {
        // Required empty public constructor
    }
 
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        // Inflate the layout for this fragment
        View view = inflater.inflate(R.layout.fragment_my, container, false);
        
        unbinder = ButterKnife.bind(this, view);
 
        return view;
    }
 
    @Override
    public void onDestroyView() {
        super.onDestroyView();
        unbinder.unbind();
    }
}
```

### ViewHolder
```java
public class MyViewHolder extends RecyclerView.ViewHolder {
    @BindView(R.id.name)
    TextView name;
 
    @BindView(R.id.mobile)
    TextView mobile;
 
    public MyViewHolder(View view) {
        super(view);

        ButterKnife.bind(this, view);
    }
}
```

## [Calligraphy](https://github.com/chrisjenx/Calligraphy)

## [Dagger 2](https://github.com/google/dagger)
Dagger 2 library handles dependency injection for Android applications. Dependency Injection is a software design pattern to make applications loosely coupled, extensible and maintainable. There is dependency when an object depends on another object to do work. 

A dependency is an object that can be used (as a service). An injection is the passing of a dependency to a dependent object. In dependency injection context, dependencies are supplied to the class that needs dependency to avoid the need for a class itself to create them for software to be loosely coupled and highly maintainable. 

For example, `SharedPreferences` is dependent on `Context`. To get a reference to SharedPreferences, we have to use `this.getSharedPreferences` where `this` represents `Context` in Android. 
```java
public class MyApplication extends Application {

    private SharedPreferences myPrefs;

    @Override
    public void onCreate() {
        super.onCreate();
        
        myPrefs = this.getSharedPreferences("com.company", Context.MODE_PRIVATE);
    }
}
```

| Annotation | Description |
| --- | --- |
| `@Inject` | Used in a constructor, a field or a method in dependent object |
| `@Module` | Classes with methods that provides method to retrieve dependent object |
| `@Component` | Bridge interface between `@Module` and `@Inject`, containing all places where object is used |
| `@Provides` | Methods within `@Module` classes that returns object |
| `@Singleton` | Object is created only once and same object is used everywhere |
| `@PerActivity` | Custom annotation where object is created only once throughout an activity, but another instance is created in a different activity |

### Gradle
#### build.gradle(Project)
```gradle
buildscript {
    // ...
    dependencies {
        classpath 'com.neenbedankt.gradle.plugins:android-apt:1.8'
    }
}
```

#### build.gradle(Module)
```gradle
apply plugin: 'com.neenbedankt.android-apt'

//...

dependencies {
    compile 'com.google.dagger:dagger:2.0-SNAPSHOT' // Dagger library
    apt 'com.google.dagger:dagger-compiler:2.0-SNAPSHOT' // Code generation
    provided 'org.glassfish:javax.annotation:10.0-b28'  // Additional annotations required outside Dagger
}
```

### Module
`ApplicationModule` provides the objects needed with its dependencies satisfied. Since `SharedPreferences` is dependent on `Context`, both providers are created in this class to get both `SharedPreferences` and `Context`, with `SharedPreferences` indicating its dependency to `Context` as parameters. Every provider method must have `@Provides` annotation and the class must have `@Module` annotation.

Instead of getting `Context` directly, we request `Context` from Dagger and Dagger will look for `Context` on your behalf via `provideContext()` method. 
```java
@Module
public class ApplicationModule {
    protected final Application mApplication;

    public ApplicationModule(Application application) {
        mApplication = application;
    }

    @Provides
    Context provideContext() {
        return mApplication;
    }

    @Provides
    @Singleton
    SharedPreferences provideSharedPreferences(Context app) {
        return app.getSharedPreferences("pref_title", Context.MODE_PRIVATE);
    }

}
```
### Injection
An `Application` class is where injection happens. Another way is to add `@Inject` annotation in Vehicle constructor. Note that `@Inject` can be used to request dependencies in constructor, fields or even methods.
```java
public class MyApplication extends Application  {

    ApplicationComponent mApplicationComponent;

    @Override
    public void onCreate() {
        super.onCreate();
    }

    public static MyApplication get(Context context) {
        return (MyApplication) context.getApplicationContext();
    }

    public ApplicationComponent getComponent() {
        if (mApplicationComponent == null) {
            mApplicationComponent = DaggerApplicationComponent.builder()
                    .applicationModule(new ApplicationModule(this))
                    .build();
        }
        return mApplicationComponent;
    }

    // Needed to replace the component with a test specific one
    public void setComponent(ApplicationComponent applicationComponent) {
        mApplicationComponent = applicationComponent;
    }
}
```

### Component
Connection between provider of dependencies `@Module` and class requesting them `@Inject` is made using `@Component` as an interface. Add methods for dependent providers.
```java
@Singleton
@Component(modules = ApplicationModule.class)
public interface ApplicationComponent {

    void inject(MyApplication application);

}

```

## [Facebook Stetho](https://github.com/facebook/stetho)
## [Gson](https://github.com/google/gson)
## [Picasso](https://github.com/square/picasso)
## [ReactiveX](https://github.com/ReactiveX/RxAndroid)
## [Retrofit](https://github.com/square/retrofit)
## [SQL Brite](https://github.com/square/sqlbrite)
## [Timber](https://github.com/JakeWharton/timber)

## References
* [Android Working With ButterKnife ViewBinding Library](https://www.androidhive.info/2017/10/android-working-with-butterknife-viewbinding-library/)
* [Dependency Injection With Dagger 2 on Android](https://code.tutsplus.com/tutorials/dependency-injection-with-dagger-2-on-android--cms-23345)
