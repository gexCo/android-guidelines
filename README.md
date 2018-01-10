# Android Guidelines 
Code guidelines used when developing Android applications

## Architecture

## Naming Convention
### Class Files
Class names are in **UpperCamelCase**. The name should end with component, for example: LoginActivity, ProfileFragment, BroadcastService, ContactAdapter, ContactHolder.

### Resource Files
Resource names are in **lowercase_underscore**.

#### Layouts
Layout names should specify the component type followed by the class name defined.

| Component | Class Name | Layout Name |
| --- | ---  | ---  |
| Activity | LoginActivity | activity_login |
| Fragment | ProfileFragment | fragment_profile |
| Adapter | -- | item_contact |
| Dialog | EditTextDialog | dialog_edit_text |

#### Strings
String names should specify the content type followed by the type of element (title, ).

| Component | Class Name | Layout Name |
| --- | ---  | ---  |
| Activity | LoginActivity | activity_login |

#### Drawables
Drawable names should specify the type of drawable followed by the use of drawable.

|Drawable Type | Prefix | Drawable Name |
| --- | --- | --- |
| Background image (png, jpeg) | bg_ | bg_article |
| Button with background | btn_bg | btn_bg_purple |
| Button with border | btn_bd | btn_bd_purple |
| Button with radius 3dp | btn_rd | btn_rd_3 |
| Button with background, border and radius | btn_bg_bd_rd | btn_bg_white_bd_purple_rd_3  |
| Icon | ic_ | ic_launcher |

#### IDs
ID names should specify 

#### Dimensions


## Java Styles
### Fields Naming
* Private, non-static field names start with m.
* Private, static field names start with s.
* Other fields start with a lower case letter.
* Static final fields (constants) are ALL_CAPS_WITH_UNDERSCORES.

```java
public class MyClass {
    public static final int SOME_CONSTANT = 42;
    public int publicField;
    private static MyClass sSingleton;
    int mPackagePrivate;
    private int mPrivate;
    protected int mProtected;
}
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
```
if (condition) body();
```

## Libraries
* [Butterknife](https://github.com/JakeWharton/butterknife)
* [Calligraphy](https://github.com/chrisjenx/Calligraphy)
* [Gson](https://github.com/google/gson)
* [Picasso](https://github.com/square/picasso)
* [Retrofit](https://github.com/square/retrofit)

# References
* [Project Guidelines](https://github.com/ribot/android-guidelines/blob/master/project_and_code_guidelines.md)
* [Architecture Guidelines](https://github.com/ribot/android-boilerplate)
* [XML Naming Convention](https://jeroenmols.com/blog/2016/03/07/resourcenaming/)
* [Google Java Style](https://google.github.io/styleguide/javaguide.html)
* [Android Development Best Practices](https://blog.mindorks.com/android-development-best-practices-83c94b027fd3)
