---
layout: post
title: Create reusable chart components with Angular 2 and D3.js version 4
date: 2016-10-17 20:01:00 -0500
categories: [dev]
tags: [angular, typescript, d3]
image: /assets/images/angular2-d3v4.gif
---

![Angular/D3.js](/assets/images/angular2-d3v4.gif)

Integrate D3.js (version 4) with the Angular 2 component life cycle to create reusable charts and other visualizations that support animation and dynamic data.

<!--more-->

*Update August 26, 2017:* This post covers integrating D3.js version 4 and later with Angular 2. For Angular 4 and later, please see [this updated post](/create-a-reusable-chart-component-with-angular-and-d3-js).

*Updated January 5, 2017:* since this post was originally written, the official D3 typings bundle was updated for D3.js 4.x. This makes life much easier and I’ve updated this post to reflect the changes.

In this example, we’ll integrate D3 version 4 with an Angular 2 app created using angular-cli to create a reusable bar-chart component.

This complete source code for this example is [available on github](https://github.com/keathmilligan/angular2-d3-v4).

# Create App Scaffolding

The core of the example is a generic CLI-generated app with a home page and default route (see app/home and app/app.routes.ts). I’ll assume you are familiar with basic component creation with angular-cli and routing at this point.

# Install D3 and Necessary Typings

Install d3 with:

`npm install --save d3`

This will install the latest version (4.4.0 at the time of this writing).

Next install the d3 typings bundle with:

`npm install --save-dev @types/d3`

# Create the Chart Component

Create a shared bar chart component:

`ng g component shared/barchart`

Edit the template (barchart.component.html):

```html
<div class="d3-chart" #chart></div>
```

The #chart id will be used to get a reference to the native element.

The components size, colors and other attributes can be controlled with CSS. The default component styles are defined barchart.component.css:

```css
.d3-chart {
  width: 100%;
  height: 400px;
}

.d3-chart .axis path,
.d3-chart .axis line {
  stroke: #999;
}

.d3-chart .axis text {
  fill: #999;
}
```

The containing component that uses this chart can override these styles.

Now have a look at the barchart.component.ts typescript source. The goal for this component is to display a chart based on data provided externally and then re-render the chart if that data changes. To accomplish this, declare an @Input data value:

```typescript
@Input() private data: Array<any>;
```

and then implement the OnChanges interface:

```typescript
  ngOnChanges() {
    if (this.chart) {
      this.updateChart();
    }
  }
```

In this case, the data is assumed to be a simple two-dimensional array of numbers.

Some other things to note:

* View encapsulation is set to “none” – this makes styling the chart component easier, but you need to be careful with class names to avoid collisions with other component styles.
* `@ViewChild` is used to get an element reference to the div container defined in the HTML. D3 manipulates the DOM directly.

The remainder of the implementation is d3-specific. Using this integration approach, you can implement a wide variety d3-based visualizations in your Angular 2 app. The [d3 documentation](https://github.com/d3/d3/wiki) has many examples and tutorials.
