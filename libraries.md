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
Dagger 2 library handles dependency injection for Android applications. Dependency Injection is a software design pattern to make applications loosely coupled, extensible and maintainable. There is dependency when an object depends on another object to do work. In dependency injection context, dependencies are supplied to the class that needs dependency to avoid the need for a class itself to create them for software to be loosely coupled and highly maintainable.

| Annotation | Description |
| --- | --- |
| `@Module` | Classes with methods that provides dependencies |
| `@Provides` | Methods within `@Method` classes |
| `@Provides @Singleton` | Indicates there will be only one instance of object |
| `@Inject` | Request a dependency (a constructor, a field or a method) |
| `@Component` | Bridge interface between `@Module` and `@Inject` |

### Steps
1. Identify the dependent objects and its dependencies.
2. Create a class with the @Module annotation, using the @Provides annotation for every method that returns a dependency.
3. Request dependencies in your dependent objects using the @Inject annotation.
4. Create an interface using the @Component annotation and add the classes with the @Module annotation created in the second step.
5. Create an object of the @Component interface to instantiate the dependent object with its dependencies.

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

### Step 1: Identify Dependent Objects
`Vehicle`(dependent) requires `Motor`(independent) to work properly. 

#### Motor
```java
public class Motor {
 
    private int rate;
 
    public Motor(){
        this.rate = 0;
    }
 
    public int getRate(){
        return rate;
    }
 
    public void accelerate(int value){
        rate = rate + value;
    }
 
    public void brake(){
        rate = 0;
    }
}
```

#### Vehicle
```java
public class Vehicle {
 
    private Motor motor;
 
    public Vehicle(Motor motor){
        this.motor = motor;
    }
 
    public void increaseSpeed(int value){
        motor.accelerate(value);
    }
 
    public void stop(){
        motor.brake();
    }
 
    public int getSpeed(){
        return motor.getRpm();
    }
}
```

### Step 2: Create `@Module` Class
#### VehicleModule
`VehicleModule` provides the objects needed with its dependencies satisfied. Since `Vehicle` is dependent on `Motor`, both providers are created in this class, with `Vehicle` indicating its dependency to `Motor` as parameters. Every provider method must have `@Provides` annotation and the class must have `@Module` annotation.

```java
@Module
public class VehicleModule {
 
    @Provides @Singleton
    Motor provideMotor(){
        return new Motor();
    }
 
    @Provides @Singleton
    Vehicle provideVehicle(){
        return new Vehicle(new Motor());
    }
}
```

### Step 3: Request Dependencies in Dependent Objects
Add `@Inject` annotation in `Vehicle` constructor. Note that `@Inject` can be used to request dependencies in constructor, fields or even methods.

```java
@Inject
public Vehicle(Motor motor){
    this.motor = motor;
}
```

### Step 4: Connect `@Module` With `@Inject`
Connection between provider of dependencies `@Module` and class requesting them `@Inject` is made using `@Component` as an interface. Add methods for dependent providers.

```java
@Singleton
@Component(modules = {VehicleModule.class})
public interface VehicleComponent {

    Vehicle provideVehicle();
 
}
```

### Step 5: Use `@Component` Interface to Obtain Objects
```
public class MainActivity extends ActionBarActivity {
 
    Vehicle vehicle;
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
 
        VehicleComponent component = Dagger_VehicleComponent.builder().vehicleModule(new VehicleModule()).build();
 
        vehicle = component.provideVehicle();
 
        Toast.makeText(this, String.valueOf(vehicle.getSpeed()), Toast.LENGTH_SHORT).show();
    }
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
