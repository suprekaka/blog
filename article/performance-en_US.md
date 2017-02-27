- Preface
  - Performance Model
- Basic
  - Browser Rendering Principles
  - Render
    - Regular Rendering Model
  - Layout (Reflow in Firefox)
    - Forced synchronous layouts
  - Paint
  - Composite
  - Jank or Janky
  - Rendering Process
    - Reflow Process
    - Re-paint Process
    - Re-composite Process
    - Nature of Property
  - Optimize Target
  - Optimization
- ExtJS
  - Event Listeners
    - 1. Ensure that event monitoring is only bind once
    - 2. Once listener
    - 3. render
  - Remove `doLayout()` and `doComponentLayout()` calls
    - Mechanism of `doLayout()`
  - Reduce container nesting
  - Reduce DOM reads and writes
    - Suspend Layout
    - Suspend layout for `Container`
- airbnb
- React
  - `Virtual DOM`
  - Performance Optimization Entry

----

# Preface
Performance issues related to a lot of aspects:

![image](../asset/images/category.png)

This article focuses on the `DOM` part

## Performance Model
![image](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/images/analysis-dom-css-js.png)


# Basic

## Browser Rendering Principles
![render](http://www.ruanyifeng.com/blogimg/asset/2015/bg2015091502.png)

## Render
Divide into:
- Layout
- Paint

### Regular Rendering Model
![image](https://developers.google.com/web/fundamentals/performance/rendering/images/intro/frame-full.jpg)

## Layout (Reflow in Firefox)
It's the process by which the browser calculates the position and size of all the elements on the page

This means that an element can affect other elements, such as child elements or sibling elements or even parent elements

Triggering conditions:
- Resize the browser window
- The JS code involves `computed values`([w3c link][link1])
- Add or remove elements from the DOM
- Change `layout` properties of an element

### Forced synchronous layouts
Today's browsers are very intelligent, they will try to focus all the `DOM` changes as a queue, then one-time processing to avoid multiple `reflow`.

Example:
```js
div.style.color = 'blue';
div.style.marginTop = '30px';
```
> Only trigger `reflow` once

But,
```js
div.style.color = 'blue';
var margin = parseInt(div.style.marginTop);
div.style.marginTop = (margin + 10) + 'px';
```
> Since the second line is to read the properties of the `DOM`, it causes the browser has to `reflow`, which eventually trigger `reflow` twice.

If you request a geometry from the DOM before a `frame` completes, it will trigger `Forced synchronous layouts`

This is a huge performance bottleneck

## Paint
The process of fill the pixel, including drawing text/color/image/border/shadow etc. In general, this drawing process is done on multiple layers.

## Composite
After drawing on multiple layers, merge all layers into a single layer in a reasonable order and then display them on the screen.

For cascading elements (cascading style sheets), the order of the merged layers affects the display of elements

## Jank or Janky
They are mean bad performance point.

----

## Rendering Process

### Reflow Process
![image](https://developers.google.com/web/fundamentals/performance/rendering/images/intro/frame-full.jpg)

If you modify the `layout` property of a DOM, such as the width and height etc., then the browser will check which elements need to be re-layout, trigger a` reflow` to re-layout, then `paint`, finally` composite`

### Re-paint Process
![image](https://developers.google.com/web/fundamentals/performance/rendering/images/intro/frame-no-layout.jpg)

If you modify the `paint only` property of a DOM, such as background images, text colors, shadows, etc. The browser will skip the` layout`.

### Re-composite Process
![image](https://developers.google.com/web/fundamentals/performance/rendering/images/intro/frame-no-layout-paint.jpg)

If you modify the `composite only` property of a DOM, the browser will skip` layout` and `paint`, directly merge render layers, such as` transform` / `opacity`


### Nature of Property
Look at [csstriggers](https://csstriggers.com/)

We should use the `composite only` properties whenever possible, followed by the` paint only` properties

## Optimize Target
At present most of the display refresh rate of 60Hz, in order to save power, the browser maximum rendering frequency also follow 60FPS (frame per second)

60 times per second `render` (if needed), every `render` time does not exceed 16.66ms

## Optimization
1. Separate the DOM property from reading and writing
2. Try to change the style at once by `class`
3. Use the `Document Fragment`, after the operation, add the `Fragment` to the DOM
4. Try to use `cloneNode ()`
5. If you want to do a lot of DOM operations, at first set element to `display: none` (1 `reflow`), then the DOM operation, the last element will be displayed (1 `reflow`)
6. Fewer performance overhead when if `position` property is `absolute` or `fixed`, because there is no need to consider the effect on other elements
7. Use `Web Worker`, the main thread for UI rendering, and other tasks to Worker thread processing
8. Use `requestAnimationFrame ()` and `requestIdleCallback ()`
9. Use `React`, solve the above problem

----

# ExtJS

## Event Listeners

### 1. Ensure that event monitoring is only bind once
Multiple bindings can cause serious performance problems

Especially pay attention to the listening binding behavior in class initialization functions

### 2. Once listener
If only need to execute `handler` once, use `single: true`

```js
listeners: {
    load: onFirstLoadData,
    single: true
}
```

### 3. render
`afterrender` event fires after all DOM elements already exist. Changing elements after rendering causes reflows, slowing your application.

Instead, adjust classes and set styles before rendering takes place using `beforerender` so that the element renders with the correct classes and styles.

Some code that must run `afterrender` may require the element size to have been acquired.


## Remove `doLayout()` and `doComponentLayout()` calls
Remove these expensive calls whenever possible.

The only time doLayout or doComponentLayout should be needed for non-bug reasons is when the DOM is directly changed by application code. 

Since the framework is unaware of such changes, these calls are needed to update the affected layouts.

### Mechanism of `doLayout()`
Triggering conditions:
- Resize container
- Add or reomve child components

The `doLayout` method is fully recursive, so any of the Container's children will have their doLayout method called as well.

## Reduce container nesting
It is important to remember that each container takes time to initialize, render, and layout, so the more you can get rid of unneeded nested containers like this, the faster your application will run.


## Reduce DOM reads and writes
Avoid using setStyle, addCls, removeCls, and other direct DOM element modifications that cause writes. 

As a general rule, for better performance, try to manipulate the DOM in batches of reads and writes when writes are needed.

### Suspend Layout
`Ext.suspendLayouts()` 与 `Ext.resumeLayouts()`

Help you coordinate updates to multiple components and containers.

```js
Ext.suspendLayouts();
// batch of updates
Ext.resumeLayouts(true);
```

> Adding two items in rapid succession to two containers, for example, causes multiple layout and render operations to be performed. If you use Ext.suspendLayouts before you add these items the framework will no longer perform the layout for each individual item. Once you’re done making your additions, use Ext.resumeLayouts and the framework will perform a single render and layout operation.


### Suspend layout for `Container`
`Container.suspendLayout` property

```js
var containerPanel = Ext.create('Ext.panel.Panel', {
    renderTo: Ext.getBody(),
    width: 400,
    height: 200,
    title: 'Container Panel',
    layout: 'column',
    suspendLayout: true // Suspend automatic layouts while we do several different things that could trigger a layout on their own
});
// Add a couple of child items.  We could add these both at the same time by passing an array to add(),
// but lets pretend we needed to add them separately for some reason.
containerPanel.add({
    xtype: 'panel',
    title: 'Child Panel 1',
    height: 100,
    columnWidth: 0.5
});
containerPanel.add({
    xtype: 'panel',
    title: 'Child Panel 2',
    height: 100,
    columnWidth: 0.5
});
// Turn the suspendLayout flag off.
containerPanel.suspendLayout = false;
// Trigger a layout.
containerPanel.doLayout();
```

----

# airbnb
> About ECMAScript part

[Link][link2]

----

# React

## `Virtual DOM`
Shielding the manual DOM operation, all for the operation of the DOM are on the `Virtual DOM`, then `React` automatically update the real DOM

Transform DOM performance issues into `render ()` optimization

## Performance Optimization Entry
`shouldComponentUpdate(nextProps:Object, nextState:Object): boolean`

This is a `leftcycle` method, and invoked before rendering when new props or state are being received. Defaults to `true`. This method is not called for the `initial render` or when `forceUpdate()` is used.

Currently, if `shouldComponentUpdate()` returns `false`, then `componentWillUpdate()`, `render()`, and `componentDidUpdate()` will not be invoked.

This is a good time for performance optimization. If you think `newProps` or` newState` does not need to update the component, you can return `false` in the method to prevent` render () `from being triggered.

----

[link1]: https://www.w3.org/TR/1998/REC-CSS2-19980512/cascade.html#computed-value
[link2]: https://github.com/airbnb/javascript#performance
