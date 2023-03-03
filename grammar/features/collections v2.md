# Collections V2

TODO: elaborate on differences between V1 and V2

This is the original problem that this feature aims to solve:
> When defining a collection, how should the syntax be defined?
> This is my current idea:
> 
> ```swift
> // collection V1
> collection InterfaceOptions {
>   req key<Str>;
>   req icon<Icon>;
>   var constraints<Arr<ConstraintFn>> = 
>   func keyDoesNotContainSpaces()<Bool> {
>       return pairs.where((key<Str>, icon<Icon>) => key.contains(' ')).isEmpty;
>    };
>
>    schema {
>       var pairs = *[key: icon];
>    }
> }
> ```
> For many reasons, this syntax is very ugly and cluttered:
> - The constructor syntax `req key<Str>;` is being mixed carelessly with the schema syntax.
> - The constraining functions are being declared as a field, when they could be declared within their own block instead (since they do not need to be accessed outside the collection, and are required for all collections).
> - Since `keys` is being declared as a variable, what would its exact type be? `Arr<Pair<Str, Icon>>`?

# Description


## Collection Schema and Constraints Syntax
```swift
// collection V1
collection Status {
    req named status:
        req named code<Int>,
        req named msg<Str>;

    var constraints<Arr<Fn<Bool, None>>> = [
        func codeIsNotOk()<Void> {
            if (status.code != 200) throw "Code is not OK 200";
        };
    ];
}
```

```swift
// collection V2
collection Status {
   schema {
      req named status: schema {
        req named code<Int>,
        req named msg<Str>;
      }
   }
}

func takeStatusInline(
    req named status: schema {
        req named code<Int>,
        req named msg<Str>;
    },
)<Void> {
    Console.log(params.schema == Status);        // false
    Console.log(params.schema == Status.schema); // true
}

func takeStatusCollection(req status<Status>)<Void> {
    Console.log(params.schema == Status.schema);          // true
    Console.log(params.schema == takeStatusInline.schema) // true
}
```
## Rules and the `Name` Type
```swift
collection InterfaceOptions1 {
    schema {
        *[req key<Name>, req icon<Icon>]
    }
}

collection InterfaceOptions2 {
    schema {
        *[req key<Str>, req icon<Icon>]
    }
}
```
`InterfaceOptions1` would not be valid code, since plans for the `Name` type have been scrapped. Instead, in situations where a `Str` type is expected and the user types an identifier, the IDE will automatically insert single quotes around the name **IF AND ONLY IF** the identifier does not point to an exisitng variable.