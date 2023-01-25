# JQWidgets is Terrible

The horrors of dealing with an unidiomatic API: A review of the JQWidgets widgets library

# Introduction

I started in Web and Full Stack development back in 2016. My experience with JS and other programming was limited to some college projects. The team had adopted this nice-looking Widgets library called JQWidgets by then. On the surface, it looks competently done, but as soon as I started development work with it, I quickly found that it was plagued with a few problems. No, I don't just mean problems like display glitches that I have to hack around. If that was all, I would be happy leaving it alone.

# The JQuery API

## Creating widgets

```js
// The pattern is: <JQuery>.jqx<widgetName>(<widgetOptions>), E.g.
$('#data-grid').jqxGrid(
{
  width: '100%',
  //etc...
});
```
OK, so far so good. I understand that I'm attaching the Grid widget to the element designated by `$(#dataGrid)`. No major sins here. But what happens if I want to do anything with it?

## Problem 1: Method calls
As advertised in the [docs][1], you call methods like this:
```js
// Select row by index 1
$("#jqxgrid").jqxGrid('selectrow', 1);
```
This is awful (even if I ignore the non-camelcased `selectrow`). Why do I have to specify the method I want to call as a string argument to another function? A well designed APi would have let you obtain a reference to the grid, then call the method like so:
```js
gridReference.selectrow(1);
```
But how? There's no documented way to do this. (Keep this in mind for later.)

## Problem 2: Getting Properties
You get properties of the grid like this:
```js
var filter = $('#jqxGrid').jqxGrid('filter');
```
Looks fairly innocuous, I suppose. But wait... what if there was a zero-argument method called `filter` too? You guessed it. It would look the __exact same__. A sensible API would have let you do this:
```js
const filter = gridReference.filter;
```
No ambiguity whatsoever. Is there anything like this in the docs? No.

But wait, it gets worse.

## Problem 3: Setting Properties
OK, so if you get properties by calling the widget function with the string, how do you even set properties? There's two ways.
```js
// 1. Use the same thing as when you're initializing
$('#jqxGrid').jqxGrid({ selectionmode: 'none'}); 
// 2. Use a string as when you're getting the properties and calling methods
$('#jqxGrid').jqxGrid('selectionmode', 'none');
```
The first way, while less offensive, still is a problem because it's ambiguous whether you're creating a widget, or just setting a few options. The second way is completely stupid. Don't ever use it. Are you calling a 1-argument method or setting a property? You might think a sensible API would do this:
```js
gridRef.selectionmode = 'none';
```
But... you want the widget to refresh on update. So create a method for setting options. DevExtreme does this, for example:
```js
gridRef.option({prop: 'none'});
// Or even this;
gridRef.option('prop', 'none');
```
Do the docs show anything like this? Again, no.

## Problem 4: Static analysis and TypeScript
TypeScript is wonderful. It fixes a lot of the underlying problems that plain JS has. However, the benefit is nullified if you don't have good types to begin with. Because everything is done with strings in JQWidgets, it's extremely difficult to at least hack together a type declaration file that annotates the return type of all these methods. (Plus it's really not my job.) So it looks like there's no choice but to slap `any` everywhere...

...Or not? You see, they actually made a .d.ts file. It's [right here][2], tucked away in their Angular integration code. So that's it, right? All the bad times are over? Well, no. Even if you import it into your TS project, stuff like above is _still_ not recognized. It also has no JSDoc annotations, and `any`s a plenty. So back to square one.

Oh, wait... what's this method? Oh, my...

## Problem 5: Documentation Failure and mitigation strategies
So, yeah, remember all the times above when I said the "docs don't show a way to do this?" Well... I'd like to call your attention to [this little line][3] in the .d.ts file and a little method called `createInstance`.

### Instances
You can create instances like so:
```js
// Don't celebrate yet. This is still not great. Note that you have to specify 'jqxGrid' as a string and declare a bunch of types
// This is actually not too hard to mitigate, but requires your own .d.ts file to declare overrides. They didn't bother to do this.
const gridReference: jqwidgets.jqxGrid = jqwidgets.createInstance('#data-grid', 'jqxGrid', options as jqwidgets.GridOptions);
```
But what if you already have an instance in a different scope? You can get an existing instance like so:
```js
// I recommend adding an overload for this method in your .d.ts file as the only valid way to call `jqx`-anything.
// Stupidly, this also returns an `any` type, but you can override this in your own .d.ts file if you want.
const gridReference: jqwidgets.jqxGrid = $('#data-grid').jqxGrid('getInstance');
```

### Setting Properties
The only thing that's weird is setting properties. Like I mentioned above, setting properties doesn't refresh the widget. Luckily, they also have `setOptions()`:
```js
// Unfortunately, their .d.ts was somewhat incompetently made, so if you want to make this work, you'll have to modify the .d.ts file to fix their inheritence/implementation mess for them
gridReference.setOptions({option: value});
```

With this, you are one step closer to a healthier Web Dev Experience. The next would be to ditch it entirely. For most of these widgets, DevExtreme offers a lot of equivalents, but with decent TS/TSX integration. For everything else? Just do it yourself and save your dev team the headache.

# Appendix A: Even easier instantiation
You can actually create most widgets by doing this too:
```js
const gridReference = new jqwidgets.jqxGrid('#data-grid', options as jqwidgets.GridOptions);
```
Just be aware that this is not guaranteed to work, and you still have to add this to your own .d.ts file if you want types. The modifications required are less messy than the ones required to type every `createInstance` overload though. In my testing only instantiating `jqxTabs` this way is broken. However, everything else seems to work, so I just accept the risk and make an exception for `jqxTabs`.

# Appendix B: The overriding .d.ts file
I mentioned this several times in mitigation strategies above, but you can augment their .d.ts file with your own. 

I have added a skeleton in this repository: https://github.com/calculuswhiz/jqwidgets-review/blob/main/mitigations.d.ts

[1]: https://www.jqwidgets.com/jquery-widgets-documentation/documentation/jqxgrid/jquery-grid-api.htm
[2]: https://github.com/jqwidgets/jQWidgets/blob/9b1b78202c4182592fccfcf5dd0216728f1417bf/jqwidgets-ts/jqwidgets.d.ts
[3]: https://github.com/jqwidgets/jQWidgets/blob/9b1b78202c4182592fccfcf5dd0216728f1417bf/jqwidgets-ts/jqwidgets.d.ts#L16
