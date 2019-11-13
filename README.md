# Dynamic Proto JavaScript

Generates dynamic prototype methods for JavaScript objects (classes) by supporting method definition within their "class" constructor (like an instance version), this removes the need to expose internal properties on the instance (this) and the usage of ```ClassName.prototype.funcName()``` both of which result in better code minfication (smaller output) and therefore improved load times for your users.

The dynamically generated prototype methods support class inheritance of any type, which means you can extend from base classes that use instance or prototype defined methods, you also don't need to add the normal boiler plate code to handle detecting, saving and calling any previous instance methods that you are overriding as support for this is provided automatically. 

So whether creating a new class or extending some other class/code, your resulting code, can be successfully extended via TypeScript or JavaScript.

## Removing / Hiding internal properties from instance
By defining the properties / methods within the constructors closure, each instance can contain or define internal state in the form of properties which it does not have to expose publically as each defined "public" instance method has direct access to this define state within the context/scope of the closure method. 

While this does require some additional CPU and memory at the point of creating each instance object this is designed to be as minimal as possible and should be outwayed by the following advantages :-

* Avoids polluting the instance (this) namespace with internal values that can cause issues with inheritence for base/super classes or even derived classes that extend your class.
* Smaller code as the internal properties and methods when defined within the instance can be minified.
* As the resulting generated code can be better minified this *should* result in a smaller minified result and therefore better load times for your users. 

## Basic Usage
```typescript
import dynamicProto from "@microsoft/dynamicproto-js";
class ExampleClass extends BaseClass {
    constructor() {
        dynamicProto(ExampleClass, this, (_self, base) => {
            // This will define a function that will be converted to a prototype function
            _self.newFunc = () => {
                // Access any "this" instance property  
                if (_self.someProperty) {
                    ...
                }
            }
            // This will define a function that will be converted to a prototype function
            _self.myFunction = () => {
                // Access any "this" instance property
                if (_self.someProperty) {
                    // Call the base version of the function that we are overriding
                    base.myFunction();
                }
                ...
            }
            _self.initialize = () => {
                ...
            }
            // Warnings: While the following will work as _self is simply a ference to
            // this, if anyone overrides myFunction() the overridden will be called first
            // as the normal JavaScript method resolution will occur and the defined
            // _self.initialize() function is actually gets removed from the instance and
            // a proxy prototype version is created to reference the created method.
            _self.initialize();
        });
    }
}
```

## Build & Test this repo

1. Install all dependencies

    ```sh
    npm install
    npm install -g @microsoft/rush
    ```

2. Navigate to the root folder and update rush dependencies

    ```sh
    rush update
    ```

3. Build, lint, create docs and run tests

    ```sh
    rush build
    npm run test
    ```

If you are changing package versions or adding/removing any package dependencies, run<br>**```rush update --purge --recheck --full```**<br>before building. Please check-in any files that change under common\ folder.

## Performance

The minified version of this adds a negligible amount of code and loadtime to your source code and by using this library, your generated code can be better minified as it removes most references of Classname.prototype.XXX methods from the generated code.

> Summary:
>
> - **~2 KB** minified (uncompressed)

## Example usage and resulting minified code

In this first example of code that is typically emitted by TypeScript it contains several references to the Classname.prototype and "this" references, both of which cannot be minfied.

```Javascript
var NormalClass = /** @class */ (function () {
    function NormalClass() {
        this.property1 = [];
        this.property1.push("Hello");
    }
    InheritTest1.prototype.function1 = function () {
        //...
        doSomething();
    };
    InheritTest1.prototype.function2 = function () {
        //...
        doSomething();
    };
    InheritTest1.prototype.function3 = function () {
        //...
        doSomething();
    };
    return NormalClass;
}());
```

So the result would look something like this which represents a ~45% compression, note that the Classname.prototype appears several times.

```JavaScript
var NormalClass=(InheritTest1.prototype.function1=function(){doSomething()},InheritTest1.prototype.function2=function(){doSomething()},InheritTest1.prototype.function3=function(){doSomething()},function(){this.property1=[],this.property1.push("Hello")});
```

While in this example when using the dynamicProto helper to create the same resulting class and objects there are no references to Classname.prototype and only 1 reference to this.

```JavaScript
var DynamicClass = /** @class */ (function () {
    function DynamicClass() {
        dynamicProto(DynamicClass, this, function (_self, base) {
            _self.property1 = [];
            _self.property1.push("Hello()");
            _self.function1 = function () {
                //...
                doSomething();
            };
            _self.function1 = function () {
                //...
                doSomething();
            };
            _self.function1 = function () {
                //...
                doSomething();
            };
        });
    }
    return DynamicClass;
}());
```

Which results in the following minified code which is much smaller and represents ~63% compression.

```Javascript
var DynamicClass=function n(){dynamicProto(n,this,function(n,o){n.property1=[],n.property1.push("Hello()"),n.function1=function(){doSomething()},n.function1=function(){doSomething()},n.function1=function(){doSomething()}})};
```

 So when looking at the code for NormalClass and DynamicClass, both end up with 1 instance property called ```property1``` and the 3 functions ```function1```, ```function2``` and ```function3```, in both cases the functions are defined ONLY on the "class" prototype and ```property1``` is defined on the instance. So anyone, whether using JavaScript or TypeScript will be able to "extend" either of class without any concerns about overloading instance functions and needing to save any previous method. And you are extending a 3rd party library you no longer have to worry about them changing the implementation as ```dynamicProto()``` handles converting overriden instance functions into prototype level ones. Yes, this means that if you don't override instance function it will continue to be an instance function.

## When to use

While this helper was created to support better minification for generated code via TypeScript code, it is not limited to only being used from within TypeScript, you can use the helper function directly in the same way as the examples above.

As with including any additional code into your project there are trade offs that you need to make, including if you are looking at this helper, one of the primary items is the overall size of the additional code that you will be including vs the minification gains that you *may* obtained. This project endeavours to keep it's impact (bytes) as small as possible while supporting you to create readable and maintainable code that will create a smaller minified output.

In most cases when creating JavaScript to support better minfication, when your code doesn't expose or provide a lot of public methods or only uses un-minifiable "names" less than 2 times, then you may not see enough potential gains to counteract the additional bytes required from the helper code. However, for any significant project you should.

So at the end of the day, if you are creating JS classes directly you *should* be able to create a simplier one-off solution that would result in smaller output (total bytes). This is how this project started, but, once we had several of these one-off solutions it made more sense to build it once.

## Browser Support

![Chrome](https://raw.githubusercontent.com/alrra/browser-logos/master/src/chrome/chrome_48x48.png) | ![Firefox](https://raw.githubusercontent.com/alrra/browser-logos/master/src/firefox/firefox_48x48.png) | ![IE8](https://raw.githubusercontent.com/hotoo/browser-logos/master/ie9-10/ie9-10_48x48.png) | ![Edge](https://raw.githubusercontent.com/alrra/browser-logos/master/src/edge/edge_48x48.png) | ![Opera](https://raw.githubusercontent.com/alrra/browser-logos/master/src/opera/opera_48x48.png) | ![Safari](https://raw.githubusercontent.com/alrra/browser-logos/master/src/safari/safari_48x48.png)
--- | --- | --- | --- | --- | --- |
Latest ✔ | Latest ✔ | 8+ Full ✔ | Latest ✔ | Latest ✔ | Latest ✔ |

## ES3/IE8 Compatibility

As an library there are numerous users which cannot control the browsers that their customers use. As such we need to ensure that this library continues to "work" and does not break the JS execution when loaded by an older browser. While it would be ideal to just not support IE8 and older generation (ES3) browsers there are numerous large customers/users that continue to require pages to "work" and as noted they may or cannot control which browser that their end users choose to use.

As part of enabling ES3/IE8 support we have set the ```tsconfig.json``` to ES3 and ```uglify``` settings in ```rollup.config.js``` transformations to support ie8. This provides a first level of support which blocks anyone from adding unsupported ES3 features to the code and enables the generated javascript to be validily parsed in an ES3+ environment.

Ensuring that the generated code is compatible with ES3 is only the first step, JS parsers will still parse the code when an unsupport core function is used, it will just fail or throw an exception at runtime. Therefore, we also need to require/use polyfil implementations or helper functions to handle those scenarios.

### ES3/IE8 Features, Solutions, Workarounds and Polyfil style helper functions

This table does not attempt to include ALL of the ES3 unsuported features, just the currently known functions that where being used at the time or writing. You are welcome to contribute to provide additional helpers, workarounds or documentation of values that should not be used.

|  Feature  |  Description  |  Usage |
|-----------|-----------------|------|
| ```Object.keys()``` | Not provided by ES3 and not used | N/A |
| ES5+ getters/setters<br>```Object.defineProperty(...)``` | Not provided by ES3 and not used | N/A |
| ```Object.create(protoObj, [descriptorSet]?)``` | Not provided by ES3 and not used | N/A |
| ```Object.defineProperties()``` | Not provided by ES3 and not used | N/A |
| ```Object.getOwnPropertyNames(obj)``` | Not provided by ES3 and not used | N/A |
| ```Object.getPrototypeOf(obj)``` | Not provided by ES3 and not used | ```_getObjProto(target:any)``` |
| ```Object.getOwnPropertyDescriptor(obj)``` | Not provided by ES3 and not used | N/A |
| ```Object.preventExtensions(obj)``` | Not provided by ES3 and not used | N/A |
| ```Object.isExtensible(obj)``` | Not provided by ES3 and not used | N/A |
| ```Object.seal(obj)``` | Not provided by ES3 and not used | N/A |
| ```Object.isSealed(obj)``` | Not provided by ES3 and not used | N/A |
| ```Object.freeze(obj)``` | Not provided by ES3 and not used | N/A |
| ```Object.isFrozen(obj)``` | Not provided by ES3 and not used | N/A |

## Contributing

Read our [contributing guide](./CONTRIBUTING.md) to learn about our development process, how to propose bugfixes and improvements, and how to build and test your changes to Application Insights.
