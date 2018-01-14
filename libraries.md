# Android Libraries
* [AutoValue](https://github.com/jun159/android-guidelines/blob/master/libraries.md#autovalue)
* [ButterKnife](https://github.com/jun159/android-guidelines/blob/master/libraries.md#butterknife)
* [Crashlytics](https://github.com/jun159/android-guidelines/blob/master/libraries.md#crashlytics)
* [Dagger 2](https://github.com/jun159/android-guidelines/blob/master/libraries.md#dagger-2)
* [Timber](https://github.com/jun159/android-guidelines/blob/master/libraries.md#timber)

## [AutoValue](https://github.com/google/auto/tree/master/value)
AutoValue is a code generating annotation processor that reduces boilerplate code. For example in OOP, classes often comes with getter, setter, equals(Object), hashCode() and toString() methods. These methods are considered as boilerplate as they are written in the standard way in every class. To avoid writing by hand, we use `AutoValue` to auto-generate these methods at compile time. 

### User (Manual)
```java
public class User {

    private String firstName;
    private String lastName;
    
    public User(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }
    
    public String getFirstName() {
        return this.firstName;
    }
    
    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }
    
    public String getLastName() {
        return this.lastName;
    }
    
    public void setLastName(String lastName) {
        this.lastName = lastName;
    }
    
    @Override
    public boolean equals(Object o) {
        if (o instanceof User) {
            return this.firstName.equals(o.firstName) 
                && this.lastName.equals(o.lastName);
        }

        return false;
    }
    
    @Override 
    public int hashCode() {
        int result = firstName != null ? firstName.hashCode : 0;
        result = 31 * result + (lastName != null ? lastName.hashCode() : 0);
        return result;
    }
    
    @Override 
    public String toString() {
        return "User(firstName='" + firstName + "\'" +
            lastName='" + lastName + "\')";
    }
}
```

### Step 1: Gradle
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
    provided 'com.google.auto.value:auto-value-parcel:1.2-rc1'
    apt 'com.google.auto.value:auto-value:1.2-rc1'
    apt 'com.ryanharter.auto.value:auto-value-parcel:0.2.3-rc2' // Parcelable
    compile 'com.ryanharter.auto.value:auto-value-parcel-adapter:0.2.3.rc2' // TypeAdapter
}
```

### Step 2: Using AutoValue in User object
```java
@AutoValue
public abstract class User {
    public abstract String firstName();
    public abstract String lastName();
    
    public static User with(String firstName, String lastName) {
        return new AutoValue_User(firstName, lastName);
    }
    
    public String fullName() {
        return firstName() + lastName();
    }
}
```

### Parcelable
When passing complex objects between Activities in Android (e.g. `intent.putExtra("user", user)`), the object must be parcelable. Parcelable is an efficient serialization mechanism but contains many boilerplate code. We can use `AutoValue` by simply implementing `Parcelable` in our class to avoid writing boilerplate code by hand. 
```java
@AutoValue
public abstract class User implements Parcelable {
    public abstract String firstName();
    public abstract String lastName();
    
    public static User with(String firstName, String lastName) {
        return new AutoValue_User(firstName, lastName);
    }
    
    public String fullName() {
        return firstName() + lastName();
    }
}
```

### TypeAdapters
If class contains properties that are not `Parcelable`, we can use `TypeAdapters` offered in AutoValue library. It allows you to define custom parcelling behaviour to ensure AutoValue can be used with any type.
```java
@AutoValue
public abstract class User implements Parcelable {
    public abstract String firstName();
    public abstract String lastName();
    @ParcelAdapter(AccountTypeAdapter.class) public abstract Account account();  // Not parcelable, e.g. from external library
    
    public static User with(String firstName, String lastName) {
        return new AutoValue_User(firstName, lastName);
    }
    
    public String fullName() {
        return firstName() + lastName();
    }
}
```

Write a type adapter class that implements `TypeAdapter`.
```java
public class AccountTypeAdapter implements TypeAdapter<Account> {
    @Override
    public Account fromParcel(Parcel in) {
        return new Account(in.readString(), new Date(in.readString()));
    }
    
    @Override
    public void toParcel(Account value, Parcel dest) {
        dest.writeString(value.tokenId);
        dest.writeLong(value.expiration.getTime());
    }
}
```

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

### Step 1: Gradle
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

### Step 2: Using in Components
#### Activity
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

#### Fragment
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

#### ViewHolder
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
## [Crashlytics](https://fabric.io/kits/ios/crashlytics?utm_campaign=crashlytics-marketing&utm_medium=natural)
Crashlytics is a tool that provides crash reports. This tool can be used with logging libraries such as Timber.

#### Application
```java
public class MyApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();
        
        if(BuildConfig.DEBUG) {
            // Debug mode - show all logs
            Timber.plant(new Timber.DebugTree());
        } else {
            // Release mode - only show logs that are important such as warnings and errors to see them in the release version of the app
            Fabric.with(this, new Crashlytics());
            Timber.plant(new ReleaseTree());
        }
    }
}
```

#### Custom Tree
```java
public class ReleaseTree extends Timber.Tree {

    private static final int MAX_LOG_LENGTH = 4000;
    
    @Override
    protected boolean isLoggable(int priority) {
        if(priority == Log.VERBOSE || priority == Log.DEBUG || priority == LOG.INFO) {
            return false;
        }
        
        // Only log WARN, ERROR, WTF
        return true;
    }
    
    @Override
    protected void log(int priority, String tag, String message, Throwable t) {
        if(isLoggable(priority) {
            
            // Report caught exceptions to Crashlytics
            if(priority == Log.ERROR && t != null) {
                Crashlytics.log(e);
            }
            
            if(message.length() < MAX_LOG_LENGTH) {
                if(priority == Log.ASSERT) {
                    Log.wtf(tag, message);
                } else {
                    Log.println(priority, tag, message);
                }
                
                return;
            }
        }
    }
}
```

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

### Step 1: Gradle
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

### Step 2: Create @Module Class
`ApplicationModule` provides the objects needed with its dependencies satisfied. Since `SharedPreferences` is dependent on `Context`, both providers are created in this class to get both `SharedPreferences` and `Context`, with `SharedPreferences` indicating its dependency to `Context` as parameters. Every provider method must have `@Provides` annotation and the class must have `@Module` annotation.

To use `SharedPreferences`, instead of getting `Context` directly, we request `Context` from Dagger and Dagger will look for `Context` on our behalf via `provideContext()` method. 

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
    Resources provideResources() {
        return application.getResources();
    }

    @Provides
    @Singleton
    SharedPreferences provideSharedPreferences(Context app) {
        return app.getSharedPreferences("pref_title", Context.MODE_PRIVATE);
    }
}
```
### Step 3: Request Dependencies in Dependent Objects via Injection
An `Application` class is where injection happens. Another way is to add `@Inject` annotation in constructor, fields or even methods.
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
}
```

### Step 4: Use `@Component` to Connect `@Module` with `@Inject`
Connection between provider of dependencies `@Module` and class requesting them `@Inject` is made using `@Component` as an interface. Specify places (Activity / Application) to inject.
```java
@Singleton
@Component(modules = ApplicationModule.class)
public interface ApplicationComponent {

    void inject(MyApplication application);
    
    void inject(BaseActivity baseActivity);

}

```

### Step 5: Use `@Component` Interface to Obtain Objects
Create `BaseActivity` class that every Activity in the application must implement. 
```java
public class BaseActivity extends AppCompatActivity {
    
    @Inject public SharedPreferences prefs;
    @Inject public Context context;
    @Inject public Resources res;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
         MyApplication.get(this).getComponent().inject(this);
    }
}
```

### Step 6: Create an `Activity` that implements `BaseActivity`
```java
public class MainActivity extends BaseActivity {

    private TextView textView;
  
    @Override
    protected void onCreate(Bundle savedInstanceState) {
         super.onCreate(savedInstanceState);
         setContentView(R.layout.activity_main);
         
         textView = (TextView) findViewById(R.id.textView);
         textView.setBackgroundColor(res.getColor(android.R.color.holo_red_dark));
         
         prefs.edit().putInt("number", 11).apply();
    }
}
```

## [Gson](https://github.com/google/gson)
## [Picasso](https://github.com/square/picasso)
## [ReactiveX](https://github.com/ReactiveX/RxAndroid)
## [Retrofit](https://github.com/square/retrofit)
## [SQL Brite](https://github.com/square/sqlbrite)

## [Timber](https://github.com/JakeWharton/timber)
Timber is a logging library for Android. Some advantage to use this over the default logging in Android is:
1. No tagging is required. The tag is filename where you are logging from.

### Step 1: Gradle
#### build.gradle(Module)
```gradle

dependencies {
    compile 'com.jakewharton.timber:timber:4.0.1'
}
```

### Step 2: Timber Initialization
```java
public class MyApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();
        
        Timber.plant(new Timber.DebugTree() {
            // Add line number to the tag
            @Override
            protected String createStackElementTag(StackTraceElement element) {
                return super.createStackElementTag(element) + ": " + element.getLineNumber();
            }
        }
    }
}
```

### Step 3: Log in Activity
```java
public class MainActivity extends BaseActivity {

    private TextView textView;
  
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        Timber.d("Hello");
        Timber.v("Goodbye");
    }
}
```

#### Step 4: Logcat
```
10-08 14:31:39.617 1933-19833/? D/MainActivity:14: Hello      // Line 14 in MainActivity
10-08 14:31:39.617 1933-19833/? D/MainActivity:15: Goodbye    // Line 15 in MainActivity
```

### Custom Log 
Define the type of logs to be logged. This is useful as VERBOSE and DEBUG logs must be disabled on release builds. We can leave INFORMATION, WARNING and ERROR logs enabled only if they are not leaking private information such as email addresses.

#### Application
```java
public class MyApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();
        
        // OR setup debug and release environment as shown below
        if(BuildConfig.DEBUG) {
            // Debug mode - show all logs
            Timber.plant(new Timber.DebugTree() {
                // Add line number to the tag
                @Override
                protected String createStackElementTag(StackTraceElement element) {
                    return super.createStackElementTag(element) + ": " + element.getLineNumber();
                }
            }
        } else {
            // Release mode - only show logs that are important such as warnings and errors to see them in the release version of the app
            Timber.plant(new ReleaseTree());
        }
    }
}
```

#### ReleaseTree
```java
public class ReleaseTree extends Timber.Tree {
    
    @Override
    protected boolean isLoggable(int priority) {
        if(priority == Log.VERBOSE || priority == Log.DEBUG || priority == LOG.INFO) {
            return false;
        }
        
        // Only log WARN, ERROR, WTF
        return true;
    }
}
```

### Debug Build Variant
#### Step 1: Create `debug` folder 
Create `debug` folder under `app->src` and `java` folder under `debug` folder. Create package that is of the same name as your application package.

#### Step 2: Application
```java
public class MyApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();
        
        Timber.plant(new Timber.DebugTree() {
            // Add line number to the tag
            @Override
            protected String createStackElementTag(StackTraceElement element) {
                return super.createStackElementTag(element) + ": " + element.getLineNumber();
            }
        }
    }
}
```

### Release Build Variant
#### Step 1: Create `release` folder 
Create `release` folder under `app->src` and `java` folder under `release` folder. Create package that is of the same name as your application package, for example `com.company`. Release version shall only log warnings, error and wtfs.

#### Step 2: Application
```java
public class MyApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();
        
        Timber.plant(new ReleaseTree());
        Fabric.with(this, new Crashlytics());
    }
}
```

#### Step 3: ReleaseTree
#### ReleaseTree
```java
public class ReleaseTree extends Timber.Tree {
    
    @Override
    protected boolean isLoggable(int priority) {
        if(priority == Log.VERBOSE || priority == Log.DEBUG || priority == LOG.INFO) {
            return false;
        }
        
        // Only log WARN, ERROR, WTF
        return true;
    }
}
```

## References
* [Android Working With ButterKnife ViewBinding Library](https://www.androidhive.info/2017/10/android-working-with-butterknife-viewbinding-library/)
* [Dependency Injection With Dagger 2 on Android](https://code.tutsplus.com/tutorials/dependency-injection-with-dagger-2-on-android--cms-23345)
