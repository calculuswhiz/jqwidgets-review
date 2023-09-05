# JQWidgets is Cursed

The horrors of dealing with an unidiomatic API: A review of the JQWidgets widgets library

# Introduction

I started in Web and Full Stack development back in 2016. The team I joined had adopted this nice-looking Widgets library called JQWidgets by then. On the surface, it looked competently done, but as soon as I started development work with it, I quickly found that it was plagued with some major problems. No, I don't just mean problems like display glitches that I have to hack around. If that was all, I would be happy leaving it alone. I'm talking about **fundamental issues** that JQWidgets has at the programming language and development level.

# The JQuery API
For the sake of this review, I will only be looking primarily at the jQuery API since I have not worked with their other frameworks.

## Creating widgets
First, the basics. This is how you create a widget. In this case, I'm making a grid:

```js
// The pattern is: <JQuery>.jqx<widgetName>(<widgetOptions>), E.g.
$('#data-grid').jqxGrid(
{
  width: '100%',
  // Other properties
});
```
OK, so far so good. I understand that I'm attaching the Grid widget to the element designated by id `dataGrid`. No major sins here. But what happens if I want to do anything with it?

## Problem 1: Method calls must be done with strings

As stated in the [docs][1], you call methods like this:
```js
// Select row by index 1
$("#jqxgrid").jqxGrid('selectrow', 1);
```
This is awful, even apart from the non-camel-cased `selectrow`. One should not have to specify a method's name as a string argument to another method in order to invoke it.

A well designed API would have let you obtain a reference to the grid, then call the method like so:
```js
gridReference.selectrow(1);
```
Unfortunately there's no _documented_ way to do this.

## Problem 2: Reading properties may be ambiguous to method calls

You get properties of the grid like this:
```js
var filter = $('#jqxGrid').jqxGrid('filter');
```
...which is exactly how you would call a zero-argument method called `filter`. There's no way to distinguish between a zero-argument function call and a property retrieval.

A sensible API would have let you do this:
```js
const filter = gridReference.filter;
```
No ambiguity whatsoever. Is there anything like this in the docs? Again, no.

## Problem 3: Setting Properties is always ambiguous

OK, so if you get properties by calling the widget function with the string, how do you even set properties? There are two ways:
1. Pass an object with properties
```js
$('#jqxGrid').jqxGrid({ selectionmode: 'none'});
```
2. Pass the name as the first argument and value as second argument
```js
$('#jqxGrid').jqxGrid('selectionmode', 'none');
```
The first way, while less offensive, is still a problem because it's ambiguous as to whether you're creating a widget, or just setting a few options. 

The second way is ambiguous. Don't ever use it. Are you calling a 1-argument method or setting a property? 

You might think a sensible API would do this:

```js
gridRef.selectionmode = 'none';
```
But you would want the widget to refresh when the property is updated. So a special method would have to be used for setting options. DevExtreme does this, for example:
```js
gridRef.option({prop: 'none'});
```
The following is also valid. Unlike JQwidgets, it's not ambiguous because `option` is the method used for setting options.
```js
gridRef.option('prop', 'none');
```
Do the docs show anything like this? Yet again, no.

## Problem 4: Static analysis and TypeScript have laughably poor support

As wonderful as Typescript is, any benefit is nullified if you don't have good type definitions to begin with.

Because everything is done with strings in JQWidgets, it's extremely difficult to at least hack together a type declaration file that annotates the return type of all these methods. (Plus it's really not my job.) It doesn't look like there's a `@types` package for it, so it looks like there's no choice but to slap `any` everywhere...

...Or not? You see, they actually made a .d.ts file. It's [right here][2], tucked away in their Angular integration code. So that's it, right? All the bad times are over? Well, no. Even if you import it into your TS project, the documented JQuery API is _still_ not recognized. It also has no JSDoc annotations, and `any` littered everywhere. So back to square one, it would seem... Right?

## Problem 5: Documentation failure and workarounds

Remember all the times above when I said, "the docs don't show a way to do this?" Well, there's [this line][3] in the .d.ts file and a method called `createInstance`. How does it work?

### Instances
You can create instances two different ways:
```js
// Don't celebrate yet. This is still not great. Note that you have to specify 'jqxGrid' as a string and declare a bunch of types
// This is actually not too hard to mitigate, but requires your own .d.ts file to declare overrides. They didn't bother to do this.
const gridReference: jqwidgets.jqxGrid = jqwidgets.createInstance('#data-grid', 'jqxGrid', options as jqwidgets.GridOptions);
```
But what about existing instances? Use the `getInstance` "method call." It's not great, but at least this abuse of syntax is in fewer places now:
```js
// I recommend adding an overload for this method in your .d.ts file as the only valid way to call `jqx`-anything.
// Stupidly, this also returns an `any` type, but you can override this in your own .d.ts file if you want.
const gridReference: jqwidgets.jqxGrid = $('#data-grid').jqxGrid('getInstance');
```

### Setting Properties
Like I mentioned above, setting properties doesn't refresh the widget, so you have to call a proxy method. Luckily, they also have `setOptions()` built in (undocumented):
```js
// Their .d.ts was somewhat incompetently made, so if you want to make this work, you'll have to modify the .d.ts file to fix their inheritence/implementation mess for them
gridReference.setOptions({option: value});
```

With this, you are one step closer to a healthier Web Dev Experience. The next step would be to ditch it entirely, but at this point it may take months of clean up. If your project is still young, good. For most of these widgets, DevExtreme offers a lot of equivalents, but with far better TS/TSX integration. For everything else? Just do it yourself and save your dev team the headache.

# Appendix A: Even easier instantiation
You can actually create most widgets with this syntax too:
```js
const gridReference = new jqwidgets.jqxGrid('#data-grid', options as jqwidgets.GridOptions);
```
Just be aware that this is not guaranteed to work, and you still have to add this to your own .d.ts file if you want types. The modifications required are less messy than the ones required to type every `createInstance` overload though. In my testing only instantiating `jqxTabs` this way is broken. However, everything else seems to work, so I just accept the risk and make an exception for `jqxTabs`.

# Appendix B: The overriding the .d.ts file
I mentioned this several times in yhe workarounds above, but you can augment their .d.ts file with your own. 

I have added a skeleton in this repository: https://github.com/calculuswhiz/jqwidgets-review/blob/main/mitigations.d.ts

Good luck.

[1]: https://www.jqwidgets.com/jquery-widgets-documentation/documentation/jqxgrid/jquery-grid-api.htm
[2]: https://github.com/jqwidgets/jQWidgets/blob/9b1b78202c4182592fccfcf5dd0216728f1417bf/jqwidgets-ts/jqwidgets.d.ts
[3]: https://github.com/jqwidgets/jQWidgets/blob/9b1b78202c4182592fccfcf5dd0216728f1417bf/jqwidgets-ts/jqwidgets.d.ts#L16
