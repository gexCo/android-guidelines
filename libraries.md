# Android Libraries

## [AutoValue](https://github.com/google/auto/tree/master/value)
## [Butterknife](https://github.com/JakeWharton/butterknife)
Butterknife library is used for view injection in Android using annotations.

| Annotations | Description | Example |
| --- | --- | --- |
| @BindView | Binds view object such as TextView. | `@BindView(R.id.text_name)`<br>`TextView textName` |
| @onClick | Attach click event on an element.<br>Once this is used, we do not have to declare the button `@BindView(R.id.btn1) Button btn1` anymore. |   `@onClick(R.id.btn1, R.id.btn2)`<br>`void onButtonClick(View view)` 
| @BindDrawable | Binds drawable element. | `@BindDrawable(R.mipmap.ic_launcher)`<br>`Drawable logo` |
| @BindString | Binds string resource. | `@BindString(R.string.name)`<br>`String name` |
| @BindColor | Binds color resource | `@BindColor(R.color.colorPrimary)`<br>`int colorBar`|
| @BindAnim | Binds anim resource | `@BindAnim(R.anim.move_left)`<br>`Animation animMoveLeft` |

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
## [Dagger](https://github.com/google/dagger)
## [Facebook Stetho](https://github.com/facebook/stetho)
## [Gson](https://github.com/google/gson)
## [Picasso](https://github.com/square/picasso)
## [ReactiveX](https://github.com/ReactiveX/RxAndroid)
## [Retrofit](https://github.com/square/retrofit)
## [SQL Brite](https://github.com/square/sqlbrite)
## [Timber](https://github.com/JakeWharton/timber)

# References
* [Android Working With ButterKnife ViewBinding Library](https://www.androidhive.info/2017/10/android-working-with-butterknife-viewbinding-library/)
