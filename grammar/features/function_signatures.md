# Language Design: Function Signatures and the Schema Type

Mochaccino does have an `Fn` type to annotate variables holding functions, and the `Fn` type accepts two type arguments: the return type and parameter schema. The first is pretty straightforward — when you see `Fn<Void, X>`, you know that `Void` is the return type of the function. But what is `X`? What is parameter schema? We will understand everything step-by-step by recursively exploring related concepts, starting with collections.

If you don't know what collections are, you should check out the `Collections V2` document, which delves into the design of and usage of collections. There, you will learn how schemas came about, and the syntax for schemas. You will realise then, that function parameters are defined in much a similar fashion. Then it will occur to you that schemas for collections were designed with this usage in mind! How clever! Now that you know that schemas can be used to define function parameters, you would be wondering how the parameter schema of a function is represented as a type argument.

Let's bear in mind that types and type arguments have to be compile-time constants (that is, their values should be ascertainable statically). Since function parameters have their types and optionality defined statically, it would be possible to use them as type arguments. The only question that remains is how.

For that, Mochaccino exposes a `.schema` property on every function (and collection, for that matter). This property returns the `Schema` object of the function or collection in question. As such, we can do the following:

```swift
func doSumting(req named action<Str>)<Int> {}
var doSumtingPointer<Fn<Int, doSumting.schema>> = doSumting;
```

The only catch here is that we are going to have a hard time writing type annotations for inline functions:
```swift
var fooBarPtr<Fn<Int, ???>> = (req named baz<Str>) => baz.length;
```
Thus, Mochaccino recommends that functions be defined using the `func` keyword in most cases, and using inline functions only for single-use functions.