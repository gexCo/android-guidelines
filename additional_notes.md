# Android

## Annotation
Annotations simplify code in applications and reduces the boilerplate(code with common patterns that is better to be auto-generated, e.g. getter and setter method in a Class). `@Override` is an example of Android annotations. 

### Annotation Processing
1. Build starts in java compiler. (Java compiler knows all processors, so if we want to create new one, we need to tell to compiler.)
2. Starts all Annotation Processors which is not executed. (Every annotation processor has its own implementation)
3. Loop over annotated elements inside the processor and find annotated classes, methods, fields.
4. Generate a new class with metadata of founded classes, methods, fields. (This is the place where you generate code.)
5. Create new file and write your generated string as a class.
6. Compiler checks if all annotation processors are executed. If not, start to next round.

### Dependency Injection With Annotations
Dependency Injection is a software design pattern in which one or more dependencies are injected into dependent object. Developers do not have to create objects themselves; let the dependency offer the object.

#### [Butterknife](https://github.com/JakeWharton/butterknife)
1. Take all fields with annotation
2. Look into `ViewGroup` and find the view by id
3. Cast to expected type
4. Set field

#### [Dagger](https://github.com/google/dagger)
1. Take all fields with annotation
2. Look into `Component` and find the object by type
3. Set field

## References
* [Annotation Processing](https://medium.com/@iammert/annotation-processing-dont-repeat-yourself-generate-your-code-8425e60c6657)
