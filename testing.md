# Android Testing
* [Crashlytics](https://github.com/jun159/android-guidelines/blob/master/testing.md#crashlytics)
* [Timber](https://github.com/jun159/android-guidelines/blob/master/testing.md#timber)

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

## [Timber](https://github.com/JakeWharton/timber)
Timber is a logging library for Android. Some advantage to use this over the default logging in Android is:
1. No tagging is required. The tag is filename where you are logging from.
2. Add additional fields to be shown in the log, such as line number.
3. Custom the logs to be shown during debug or release.
4. Use it with crash reports library such as Crashlytics.

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
