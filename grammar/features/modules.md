# Language Design: Modules

This is the original problem that this feature aims to solve:
> But then that got me thinking — wouldn't it be much better if type constraints could be statically checked as well? Rust uses the `test` method of the `Predicate` trait to constrain types, but functions can't be evaluated at compile-time, or can they? Turns out [they can](https://en.wikipedia.org/wiki/Compile-time_function_execution#D). For a function to be a compile-time constant, it must:
> - be [pure](https://en.wikipedia.org/wiki/Pure_function)
> - only work with constant values, including other constant functions
> Well, that's a bummer because we usually do:
> ```swift
> var foo<Str> = foobar();
> ```
> instead of
> ```swift
> var foo<Str> = "foobar";
> ```
> So, if I can implement a static type refinement system in Mochaccino, I believe the language would be greatly enhanced. Now all I need to do is figure out how.

# Description
Modules can simulate mini "environments" for the functions that they contain.
This is, in my opinion, the optimal solution because it really is impossible to have static type refinement as it hinges upon the single requisite of knowing most values at compile time. As such, type refinments can only be checked at runtime. But let's assume you *do* know the value of some variables at compile time, as is typically the case during debugging.

Say we have the following code, which is producing an error when trying to read the `payload` key from the `result` map:
```dart
void main() {
    ResponseMap result = Http.get("someUrl") as ResponseMap;
    Map payload = result['payload'];
}
```
We would debug it by doing this:
```dart
void getRequest() {
    ResponseMap result = Http.get("someUrl") as ResponseMap;
    print(result.containsKey('payload'));
    print(result);
    Map payload = result['payload'];
}
```
Then we would have to add error handlers so that when the `payload` key does not exist in future responses, the program does not panic.

However, the same functions can be done in Mochaccino like so:
```swift
module Foo of Bar {
    func getRequest()<Void> {
        var result<ResponseMap> = Http.get("someUrl");
        var payload<Map> = result['payload'];
    }
}

collection ResponseMap extends Map {
    var refinements<Arr<ConstraintFn>> = [
        func payloadIsNotNull() {
            return (get("payload") != null);
        }
    ];
}
```
By default, the refinements defined in the collection are only checked at runtime, so the collection will only report an error at runtime, much like the example before. However, adding an environment to the module causes variables declared within the module to be checked statically for refinements.
```swift
module Foo of Bar {
    define env: {
        struct Http {}
        module Get of Http {
            func get(_<Str>)<Map> {
                return {
                    "payload": null
                };
            }
        }
    }

    func getRequest()<Void> {
        var result<ResponseMap> = Http.get("someUrl");
        var payload<Map> = result['payload'];
    }
}

collection ResponseMap extends Map<Str, Obj> {
    var refinements<Arr<ConstraintFn>> = [
        func payloadIsNotNull() {
            return (get("payload") != null);
        }
    ];
}
```
What we're doing here is defining an environment for the `getRequest` function to use. For static analysis to occur, we need all values in the `getRequest` scope to be constant. Since that is not actually possible, we define an alternative environment named `getRequestEnv` for debugging, using the `define env` keywords. In this environment, we can create a debugging version of our structs and modules, where methods and properties are constant. The compiler will insert this environment into the `getRequest` function during execution. Since Mochaccino supports variable shadowing, the const version of the `Http` struct is used by the code instead of the actual struct. Finally, we also need to tell the compiler which function is going to be using the environment:
```swift
module Foo of Bar {
    define env getRequestEnv: {
        struct Http {}
        module Get of Http {
            func get(_<Str>)<Map> {
                return {
                    "payload": null
                };
            }
        }
    }

    @env:getRequestEnv
    func getRequest()<Void> {
        var result<ResponseMap> = Http.get("someUrl");
        var payload<Map> = result['payload'];
    }
}

collection ResponseMap extends Map<Str, Obj> {
    var refinements<Arr<ConstraintFn>> = [
        func payloadIsNotNull() {
            return (get("payload") != null);
        }
    ];
}
```
The best part is, to avoid cluttering your file, you can place all const environments in one place, and reference them from this file. For example, in an `environments` directory, we can have the following `getRequestEnv.mocc` file:
```swift
package env.getRequestEnv;

struct Http {}
module Get of Http {
    func get(_<Str>)<Map> {
        return {
            "payload": null
        };
    }
}
```
And directly reference the name of the environment instead:
```swift
module Foo of Bar {
    @env:getRequestEnv
    func getRequest()<Void> {
        var result<ResponseMap> = Http.get("someUrl");
        var payload<Map> = result['payload'];
    }
}

collection ResponseMap extends Map<Str, Obj> {
    var refinements<Arr<ConstraintFn>> = [
        func payloadIsNotNull() {
            return (get("payload") != null);
        }
    ];
}
```

Mochaccino's inspection and testing features work in a similar manner.