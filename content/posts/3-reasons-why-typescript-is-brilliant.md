---
author: alc
category:
  - typescript
cover:
  alt: typescript
  image: /wp-content/uploads/2014/03/typescript.png
date: "2014-03-14T15:10:23+00:00"
guid: http://laubplusco.net/?p=847
title: 3 reasons why TypeScript is brilliant
url: /3-reasons-typescript-brilliant/

---
I had the pleasure of attending a talk with Anders Hejlsberg from Microsoft today at the Copenhagen University department of computer science.

The talk was an introduction master class to TypeScript and even though I played a bit around with TypeScript before it was so cool and beneficial to hear about it from the mind who created it.

I am still somewhat of a TypeScript newbie but since I am so excited about what I saw today at the talk I will share some of the things which I think is so extremely brilliant.

## TypeScript 1-0-1

TypeScript is a superset of JavaScript that compiles to JavaScript which is compliant with the ECMA script standards used by all common browsers.

There is no propriety code involved, no need to run it in Internet Explorer only, no need for plugins or anything. It is just plain JavaScript which comes out of the compiler which will work with any JavaScript VM.

It is completely "cross-platform" what in today's world means completely cross-browser compatible.

The TypeScript compiler is open-source and build in TypeScript. The whole thing can be downloaded [here](http://typescript.codeplex.com/sourcecontrol/latest#README.txt).

It is built into the new versions of Visual Studio and can be installed in 2012. But it is not limited to Visual Studio in any way, there exist TypeScript integration for almost all IDE's out there including [eclipse](https://marketplace.eclipse.org/content/typescript) and even for [Sublime text](https://github.com/raph-amiard/sublime-typescript).

Since TypeScript is a superset of JavaScript it can treat normal JavaScript as TypeScript, simply change the file postfix from js to ts and you are effectively running TypeScript.

Then what does it offer at all? Well type safety foremost which is something that JavaScript does not offer at all.

This makes it scale for enterprise size solutions without creating a maintenance nightmare.

The compiler can be set to either explicit or implicit type safety, so you can have it either way you want. This is done by using the flag _"no implicit any_" disallowing variables which are implicit of _any_ type. You can still define variables to be of type any but then it is type safe _any_.

## 1) Lambda expressions in JavaScript

This is the first and foremost reason why I think TypeScript is brilliant, I almost couldn't get my hands down when Anders Hejlsberg showed this.

Everyone who have played around with JavaScript have experienced losing the _this_ object when running functions async. Suddenly the _this_ is not what you expect since the function is run in a different scope so _this_ is something unexpected with no clear reason for other than a JavaScript VM or maybe someone extremely gifted.

The typical work-around which works just fine can look like this:

```js
var that = this;

var myFunction = function() {
	that.x = "x";
}
```

Simply declaring a variable in the outer scope and use _that_ instead of _this_. This works just fine..

TypeScript introduces Lambda expressions which in itself is so cool but to make this work it also automates the _that-equals-this_ pattern.

The TypeScript code:

```js
var myFunction = f => { this.x = "x"; }
```

Is compiled into this piece of JavaScript, automatically creating the _that-equals-this_ pattern:

```js
var _this = this;
var myFunction = function (f) {
    _this.x = "x";
};
```

There is a whole lot more to say about TypeScript lambda expressions, they are so extremely powerful. That will be in later posts though.

## 2) Generics

We all know the concept of generics from C# where a generic type can be passed as arguments to a method with type safety guaranteed through constraints.

Well it works the same way in TypeScript: Here is a really simple example:

```js
interface NameInterface {
    name:string;
}

class NameList<T extends NameInterface> {
	private internalNameList:Array<T>;
	constructor() {
		this.internalNameList = new Array<T>();
	}

	add(value:T):void {
		this.internalNameList.push(value);
	}
}

var list = new NameList();
list.add({ name: "some name"});
```

The above TypeScript is compiled to the following JavaScript:

```js
var NameList = (function () {
    function NameList() {
        this.internalNameList = new Array();
    }
    NameList.prototype.add = function (value) {
        this.internalNameList.push(value);
    };
    return NameList;
})();

var list = new NameList();
list.add({ name: "some name" });
```

Notice that the outputted JavaScript is the exact same code just without the type safety. Generics are only there to help out us developers.

_Also notice how the private modifier is ignored in the outputted JavaScript. TypeScript will show an error if you will try to access it directly from outside the class even though it is valid in the compiled JavaScript._  [![typescripcompilerwarning_1](/wp-content/uploads/2014/03/typescripcompilerwarning_1.png)](/wp-content/uploads/2014/03/typescripcompilerwarning_1.png)

## 3) Type safety

The primary purpose of TypeScript, provide type safety in JavaScript.

TypeScript introduces both interface and class definitions.

Actually a class is just an interface with a constructor but still there is so much more behind the scenes.

Compared with all the plumbing code one usually had to write to "namespace" JavaScript then take a look at this really simple example:

```js
module LaubPlusCo.TypeScript.Example {
	class Test {
		constructor(name:string) {

		}
	}
}
```

This gets compiled to the following chunk of JavaScript:

```js
var LaubPlusCo;
(function (LaubPlusCo) {
    (function (TypeScript) {
        (function (Example) {
            var Test = (function () {
                function Test(name) {
                }
                return Test;
            })();
        })(TypeScript.Example || (TypeScript.Example = {}));
        var Example = TypeScript.Example;
    })(LaubPlusCo.TypeScript || (LaubPlusCo.TypeScript = {}));
    var TypeScript = LaubPlusCo.TypeScript;
})(LaubPlusCo || (LaubPlusCo = {}));
```

Just amazing, no more faking namespaces by hand, now it is automated :) yet alone that is amazing, I can become a JavaScript wizard coming from my C# / Java / Backend influenced world.

I have so much more I want to write about TypeScript and so many small experiments I want to perform but I have run out of time for now.

[Play around with TypeScript here](http://www.typescriptlang.org/Playground/), I promise that if you have worked with JavaScript before you will be amazed.

Also find the header files for making TypeScript work with your favorite JavaScript API here at the GitHub [Definetely Typed repository](https://github.com/borisyankov/DefinitelyTyped).
