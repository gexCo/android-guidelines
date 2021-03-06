# Android Libraries
* [AutoValue](https://github.com/jun159/android-guidelines/blob/master/libraries.md#autovalue)
* [ButterKnife](https://github.com/jun159/android-guidelines/blob/master/libraries.md#butterknife)
* [Dagger 2](https://github.com/jun159/android-guidelines/blob/master/libraries.md#dagger-2)
* [Gson](https://github.com/jun159/android-guidelines/blob/master/libraries.md#gson)
* [Picasso](https://github.com/jun159/android-guidelines/blob/master/libraries.md#picasso)
* [RxAndroid](https://github.com/jun159/android-guidelines/blob/master/libraries.md#reactivex)
* [Retrofit](https://github.com/jun159/android-guidelines/blob/master/libraries.md#retrofit)
* [SQL Brite](https://github.com/jun159/android-guidelines/blob/master/libraries.md#sql-brite)

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
    // AutoValue
    provided 'com.google.auto.value:auto-value-parcel:1.2-rc1'
    apt 'com.google.auto.value:auto-value:1.2-rc1'
    apt 'com.ryanharter.auto.value:auto-value-parcel:0.2.3-rc2' // Parcelable
    compile 'com.ryanharter.auto.value:auto-value-parcel-adapter:0.2.3.rc2' // TypeAdapter
    annotationProcessor 'com.ryanharter.auto.value:auto-value-gson:0.4.5'   // Gson
    provided 'com.ryanharter.auto.value:auto-value-gson:0.4.5'  // Gson
    
    // Retrofit
    compile 'com.squareup.retrofit2:converter-gson:2.1.0'
    compile 'com.squareup.retrofit2:adapter-rxjava:2.1.0'
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

### AutoValue Gson Extension with Retrofit 2.0
#### TypeAdapter
```java
@GsonTypeAdapterFactory
public abstract class AutoValueGsonFactory implements TypeAdapterFactory {

    public static TypeAdapterFactory create() {
        return new AutoValueGson_AutoValueGsonFactory();
    }
}
```

#### Application
```java
public class MyApplication extends Application {

    private MyService service;
    
    @Override
    public void onCreate() {
        super.onCreate();
        
        GsonConverterFactory gsonConverterFactory = GsonConverterFactory.create(new GsonBuilder()
                .registerTypeAdapterFactory(AutoValueGsonFactory.create())
                .setDateFormat("yyyy-MM-dd'T'HH:mm:ss.SSS'Z'")
                .create());
                    
        Retrofit retrofitClient = new Retrofit.Builder()
            .baseUrl(BuildConfig.END_POINT)
            .addConverterFactory(gsonConverterFactory)
            .client(okHttpClient.build())
            .build();
            
        myService = retrofit.create(MyService.class);
    }
 
    public MyService getService() {
        return myService;
    }
}
```

#### User
```java
@AutoValue
public abstract class User {

    @SerializedName("firstName")
    public abstract String firstName();
    @SerializedName("lastName")
    public abstract String lastName();
    
    public static User create(String firstName, String lastName) {
        return new AutoValue_User(firstName, lastName);
    }
    
    public static TypeAdapter<User> typeAdapter(Gson gson) {
        return new AutoValue_User.GsonTypeAdapter(gson);
    }
}
```

### AutoValue with Builders
Instead of using constructors, we can use Builders to set values in a class using `AutoValue.Builder` annotation for inner Builder class.
```java
@AutoValue
public abstract class User {

    @SerializedName("firstName")
    public abstract String firstName();
    @SerializedName("lastName")
    public abstract String lastName();
    
    public static Builder builder() {
        return new AutoValue_User.Builder();
    }
    
    public static TypeAdapter<User> typeAdapter(Gson gson) {
        return new AutoValue_User.GsonTypeAdapter(gson);
    }
    
    @AutoValue.Builder
    public abstract static class Builder {
        public abstract Builder setFirstName(String firstName);
        public abstract Builder setLastName(String lastName);
        public abstract User build();
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

### Custom Qualifer
`@Qualifier` annotation create "tags" for dependencies which have the same interface, such as Application and Activity. Qualifier will help you to identify the appropriate one. Some examples are:

| Annotation | Description |
| --- | --- |
| `@ActivityContext` | Keeps references as long as Activity exists<br>(E.g. we can share single instance of any class between all fragments hosted in this Activity) |
| `@ApplicationContext` | Keeps references as long as Application exists |

Simply define a custom qualifier like this:
```java
@Qualifier
@Retention(RetentionPolicy.RUNTIME)
public @interface ActivityContext {
}
```

### Custom Scopes
Scopes mechanism cares about keeping single instance of class as long as its scope exists. Scopes give us "local singletons" which live as long as scope itself. It is similar to `@Singleton` annotation. Some examples are:

| Annotation | Description |
| --- | --- |
| `@UserScope` | Scope for classes instances associated with picked user, keeps references as long as User exists<br>(E.g. Logged-in user) | `@PerActivity` | Scope to permit objects who will live as long as Activity is alive |
| `@ConfigPersistent` | Scope to annotate dependencies that need to survive configuration changes (e.g. Presenters)  |

Simply define a custom scope like this:
```java
@Scope
@Retention(RetentionPolicy.RUNTIME)
public @interface PerActivity {
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

## [RxAndroid](https://github.com/ReactiveX/RxAndroid)
RxAndroid is an Android library that handles observer pattern. In software engineering, observer design pattern is used when observable (subject), maintains a list of its dependents, called observers, and notifies them automatically of any state changes, usually by calling one of their methods. The observer pattern is also a key part in the familiar model–view–controller (MVC) architectural pattern. Read more at [Observer Design Pattern](https://github.com/jun159/SoftwareEngineering/wiki/Object-Interaction-Design#observer)

### Step 1: Gradle
#### build.gradle(Module)
```gradle
dependencies {
    compile 'io.reactivex.rxjava2:rxandroid:2.0.1'
    compile 'io.reactivex.rxjava2:rxjava:2.1.8'
}
```
### Step 2: 
```
public class MainActivity extends AppCompatActivity {
    private TextView textView;
    private Observable<String> mObservable;
    private Observer<String> mObserver;
    
    @Override
    protected void onCreate(Bundle savedIntanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        textView = (TextView) findViewById(R.id.textView);
        mObservable = Observable.just("Hello);
        
        mObserver = new Observer<String>() {
            @Override 
            public void onSubsribe(Disposable d) {
            
            }
            
            @Override 
            public void onNext(Disposable d) {
                textView.setText(s);
            }
        }
    }
    
    public void subscribeNow(View view) {
        mObservable.subscribe(mObserver);
    }
}
```

## [Retrofit](https://github.com/square/retrofit)

## [SQL Brite](https://github.com/square/sqlbrite)

## References
* [AutoValue Gson Extension with Retrofit 2.0](https://medium.com/3xplore/autovalue-with-retrofit-2-0-61f9530787b1)
* [Android Working With ButterKnife ViewBinding Library](https://www.androidhive.info/2017/10/android-working-with-butterknife-viewbinding-library/)
* [Dependency Injection With Dagger 2 on Android](https://code.tutsplus.com/tutorials/dependency-injection-with-dagger-2-on-android--cms-23345)
* [Dependency Injection with Dagger 2 - Custom Scopes](http://frogermcs.github.io/dependency-injection-with-dagger-2-custom-scopes/)
