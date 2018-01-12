# Android Code Style
Code guidelines used when developing Android applications.

## Naming Convention
### Class Files
Class names are in **UpperCamelCase**. 

| Component | Class Name | 
| --- | ---  | 
| Activity | LoginActivity | 
| Fragment | ProfileFragment | 
| Service | BroadcastService |
| Adapter | ContactAdapter | 
| Holder | ContactHolder | 
| Dialog | EditTextDialog |

### Resource Files
Resource names are in **lowercase_underscore**.

#### Layouts
Layout names should specify the component type follow by the class name defined.

| Component | Class Name | Layout Name |
| --- | ---  | ---  |
| Activity | LoginActivity | activity_login |
| Fragment | ProfileFragment | fragment_profile |
| Adapter | -- | item_contact |
| Dialog | EditTextDialog | dialog_edit_text |

#### Strings
String names should specify the type of string follow by the use of string.

| Prefix | Description | 
| --- | ---  | 
| error_ | Error message | 
| msg_ | Regular information message | 
| title_ | Title e.g. Activity or dialog title | 
| action_ | An action such as save, cancel, ok etc. | 

#### Drawables
Drawable names should specify the type of drawable follow by the use of drawable.

|Drawable Type | Prefix | Drawable Name |
| --- | --- | --- |
| Background image (png, jpeg) | bg_ | bg_article |
| Button with background | btn_bg | btn_bg_purple |
| Button with border | btn_bd | btn_bd_purple |
| Button with radius | btn_rd | btn_radius |
| Button with background, border and radius | btn_bg_bd_rd | btn_bg_white_bd_purple_rd  |
| Icon | ic_ | ic_launcher |

#### Resources ID
ID names should specify type of element follow by use of the content.

|Element | Prefix |
| --- | --- |
| Layouts (e.g. LinearLayout) | layout_ | 
| TextView | text_ | 
| EditText | edit_ | 
| ImageView | image_ | 
| Button | button_ |
| FloatingActionButton | fab_ |
| ProgressBar | progress_ |

## Android Java Styles
### Fields Naming
* Constants (static final fields) are ALL_CAPS_WITH_UNDERSCORES.
* Private/protected, non-static field names start with m.
* Private/protected, static field names start with s.
* Other fields start with a lower case letter.

```java
public class MainActivity {
    public static final String TITLE_DIALOG = "title";
    
    public int id;
    
    private static int sVariable;
    private int mVariable;
}
```

### Constants Naming
|Element | Prefix | 
| --- | --- |
| SharedPreferences | PREF_ |
| Bundle | BUNDLE_ |
| Intent Extra | EXTRA_ |

```java
private static final String PREF_EMAIL = "PREF_EMAIL";
private static final String BUNDLE_EMAIL = "BUNDLE_EMAIL";
private static final String EXTRA_EMAIL = "com.myapp.extras.EXTRA_EMAIL";
```

### Braces
Braces shall be on the same line as the code before them.

```java
if(isTrue) {
    // ...
} else {
    // ...
}
```
Use braces even if the if condition and body fit on one line!

```java
if(isTrue) {
    // ...
}
```
### Variables Ordering
1. Constants
2. UI variables (TextView etc)
3. Other variables (String, int)

```java
public class MainActivity extends Activity {

    private static final String TAG = MainActivity.class.getSimpleName();
    
    private EditText mEditTextEmail;
    
    private String mEmail;
}
```

### Methods Ordering
1. Constructors
2. Override methods and callbacks (public or private)
3. Public methods
4. Private methods
5. Inner classes or interfaces

### Line Length Limit
Each code line should not exceed 100 characters. 

#### Long Method Chain
```java
Picasso.with(context)
        .load("http://google.com/hello.jpg")
        .into(imageView);
```

#### Long Method Paramters
```java
loadPicture(context,
        "http://google.com/hello.jpg",
        mImageViewProfilePicture,
        clickListener,
        "Title of the picture");
```

## Logging
Use Log to print out error messages or other information that may be useful for developers to identify issues. VERBOSE and DEBUG logs must be disabled on release builds. Leave INFORMATION, WARNING and ERROR logs enabled only if they are not leaking private information such as email addresses.

To only show logs on debug builds:
```java
if (BuildConfig.DEBUG) {
     Log.d(TAG, "Email: " + email);
}
```

### Verbose
Show all log messages (default).
```java
Log.v(String tag, String msg)
```

### Debug
Show debug log messages that are useful during development, such as logging the exact flow of your program.
```java
Log.d(String tag, String msg)
```

### Information
Show expected log messages for regular usage, as well as the message levels lower in this list.
```java
Log.i(String tag, String msg)
```

### Warning
Show possible issues that are not yet errors.
```java
Log.i(String tag, String msg)
```

### Error
Show issues that have caused errors, such as in a catch statement.
```java
Log.e(String tag, String msg)
```

Use class name as tag and define it as a static final field at the top of the file. For example:
```java
public class MainActivity {
    private static final String TAG = MainActivity.class.getSimpleName();

    public method() {
        Log.e(TAG, "Error message");
    }
}
```

# References
* [Project Guidelines](https://github.com/ribot/android-guidelines/blob/master/project_and_code_guidelines.md)
* [Architecture Guidelines](https://github.com/ribot/android-boilerplate)
* [XML Naming Convention](https://jeroenmols.com/blog/2016/03/07/resourcenaming/)
* [Google Java Style](https://google.github.io/styleguide/javaguide.html)
* [Android Development Best Practices](https://blog.mindorks.com/android-development-best-practices-83c94b027fd3)
