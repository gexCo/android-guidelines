# Android

## Annotation
Annotations simplify code in applications and reduces the boilerplate(code with common patterns that is better to be auto-generated, e.g. getter and setter method in a Class). `@Override` is an example of Android annotations. 

### Annotation Processing
1. Build starts in java compiler. (Java compiler knows all processors, so if we want to create new one, we need to tell to compiler.)
2. Starts all Annotation Processors which is not executed. (Every annotation processor has its own implementation)
3. Loop over annotated elements inside the processor and find annotated classes, methods, fields
4. Generate a new class with metadata of founded classes, methods, fields. 
5. Create new file and write your generated string as a class
6. Compiler checks if all annotation processors are executed. If not, start to next round.

### Dependency Injection With Annotations
Dependency Injection is a software design pattern to make applications loosely coupled, extensible and maintainable. Developers do not have to create objects themselves; they let the dependency offer the object.

#### [Butterknife](https://github.com/JakeWharton/butterknife)
1. Take all fields with annotation
2. Look into `ViewGroup` and find the view by id
3. Cast to expected type
4. Set field

#### [Dagger](https://github.com/google/dagger)
1. Take all fields with annotation
2. Look into `Component` and find the object by type
3. Set field

#### Step 1: Identify Dependent Objects
`Vehicle`(dependent) requires `Motor`(independent) to work properly. 

##### Motor
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

##### Vehicle
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

#### Step 2: Create `@Module` Class
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

#### Step 3: Request Dependencies in Dependent Objects
Add `@Inject` annotation in `Vehicle` constructor. Note that `@Inject` can be used to request dependencies in constructor, fields or even methods.

```java
@Inject
public Vehicle(Motor motor){
    this.motor = motor;
}
```

#### Step 4: Connect `@Module` With `@Inject`
Connection between provider of dependencies `@Module` and class requesting them `@Inject` is made using `@Component` as an interface. Add methods for dependent providers.

```java
@Singleton
@Component(modules = {VehicleModule.class})
public interface VehicleComponent {

    Vehicle provideVehicle();
 
}
```

#### Step 5: Use `@Component` Interface to Obtain Objects
```java
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

## References
* [Annotation Processing](https://medium.com/@iammert/annotation-processing-dont-repeat-yourself-generate-your-code-8425e60c6657)
