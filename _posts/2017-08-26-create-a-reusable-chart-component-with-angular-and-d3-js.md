---
layout: post
title: Create a reusable chart component with Angular and D3.js
date: 2017-08-26 21:13:31 -0500
categories: [dev]
tags: [angular, typescript, d3]
featured-img: angular-d3js.gif
---

![Angular D3.js](/assets/images/angular-d3js.gif)

Integrating D3.js with Angular to create reusable chart components. This is an updated version of the [original post](/create-reusable-chart-components-with-angular-2-and-d3-js-version-4) that covered integrating D3.js (version 4) with Angular 2. This version covers the latest Angular version (currently 4.2.4).
<!--more-->

Integrating D3.js with Angular to create reusable chart components. This is an updated version of the original post that covered integrating D3.js (version 4) with Angular 2. This version covers the latest Angular version (currently 4.2.4).


In this example, we’ll create a reusable Angular chart component using D3.js and @angular/cli.

# Create App Scaffolding

If you have not updated your @angular/cli installation lately, you should do so before proceeding. This will insure you get the correct Angular package versions.

Create a new project with:

`ng new angular-d3js`

# Install D3.js and Typings

Install the current version of D3.js and its typings with:

`npm install --save d3`
`npm install --save-dev @types/d3`

# Create the Chart Component

Now generate the chart component stub with:

`ng g component shared/BarChart`

I’ve elected to place it in a “shared” directory, but you can put it anywhere you like. For example, you might elect to put it in a loadable module along with other component.

Edit the template as follows:

```html
<div class="d3-chart" #chart></div>
```

The #chart id will be used to get a reference to the native element.

The components size, colors and other attributes can be controlled with CSS. The default component styles are defined bar-chart.component.css:

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

Now have a look at the bar-chart.component.ts typescript source. The goal for this component is to display a chart based on data provided externally and then re-render the chart if that data changes:

```typescript
import { Component, OnInit, OnChanges, ViewChild, ElementRef, Input, ViewEncapsulation } from '@angular/core';
import * as d3 from 'd3';

@Component({
  selector: 'app-bar-chart',
  templateUrl: './bar-chart.component.html',
  styleUrls: ['./bar-chart.component.css'],
  encapsulation: ViewEncapsulation.None
})
export class BarChartComponent implements OnInit, OnChanges {
  @ViewChild('chart') private chartContainer: ElementRef;
  @Input() private data: Array<any>;
  private margin: any = { top: 20, bottom: 20, left: 20, right: 20};
  private chart: any;
  private width: number;
  private height: number;
  private xScale: any;
  private yScale: any;
  private colors: any;
  private xAxis: any;
  private yAxis: any;

  constructor() { }

  ngOnInit() {
    this.createChart();
    if (this.data) {
      this.updateChart();
    }
  }

  ngOnChanges() {
    if (this.chart) {
      this.updateChart();
    }
  }

  createChart() {
    const element = this.chartContainer.nativeElement;
    this.width = element.offsetWidth - this.margin.left - this.margin.right;
    this.height = element.offsetHeight - this.margin.top - this.margin.bottom;
    const svg = d3.select(element).append('svg')
      .attr('width', element.offsetWidth)
      .attr('height', element.offsetHeight);

    // chart plot area
    this.chart = svg.append('g')
      .attr('class', 'bars')
      .attr('transform', `translate(${this.margin.left}, ${this.margin.top})`);

    // define X & Y domains
    const xDomain = this.data.map(d => d[0]);
    const yDomain = [0, d3.max(this.data, d => d[1])];

    // create scales
    this.xScale = d3.scaleBand().padding(0.1).domain(xDomain).rangeRound([0, this.width]);
    this.yScale = d3.scaleLinear().domain(yDomain).range([this.height, 0]);

    // bar colors
    this.colors = d3.scaleLinear().domain([0, this.data.length]).range(<any[]>['red', 'blue']);

    // x & y axis
    this.xAxis = svg.append('g')
      .attr('class', 'axis axis-x')
      .attr('transform', `translate(${this.margin.left}, ${this.margin.top + this.height})`)
      .call(d3.axisBottom(this.xScale));
    this.yAxis = svg.append('g')
      .attr('class', 'axis axis-y')
      .attr('transform', `translate(${this.margin.left}, ${this.margin.top})`)
      .call(d3.axisLeft(this.yScale));
  }

  updateChart() {
    // update scales & axis
    this.xScale.domain(this.data.map(d => d[0]));
    this.yScale.domain([0, d3.max(this.data, d => d[1])]);
    this.colors.domain([0, this.data.length]);
    this.xAxis.transition().call(d3.axisBottom(this.xScale));
    this.yAxis.transition().call(d3.axisLeft(this.yScale));

    const update = this.chart.selectAll('.bar')
      .data(this.data);

    // remove exiting bars
    update.exit().remove();

    // update existing bars
    this.chart.selectAll('.bar').transition()
      .attr('x', d => this.xScale(d[0]))
      .attr('y', d => this.yScale(d[1]))
      .attr('width', d => this.xScale.bandwidth())
      .attr('height', d => this.height - this.yScale(d[1]))
      .style('fill', (d, i) => this.colors(i));

    // add new bars
    update
      .enter()
      .append('rect')
      .attr('class', 'bar')
      .attr('x', d => this.xScale(d[0]))
      .attr('y', d => this.yScale(0))
      .attr('width', this.xScale.bandwidth())
      .attr('height', 0)
      .style('fill', (d, i) => this.colors(i))
      .transition()
      .delay((d, i) => i * 10)
      .attr('y', d => this.yScale(d[1]))
      .attr('height', d => this.height - this.yScale(d[1]));
  }
}
```

Let’s take a look at this component:

* First off, you’ll notice that we set `encapsulation` to `ViewEncapsulation.None`. This allows D3 to access the CSS classes defined above.
* The chart container is declared as a `@ViewChild`. This allows the component to directly access the native element (which D3 needs).
* The `OnChanges` interface is implemented to allow the chart to be re-rendered (with animation) when the data changes.
* The chart is initially created in the `createChart` method.
* When the chart data has changed, the `updateData` method is invoked to re-render the chart.
* The `createChart` and `updateData` methods are fairly generic to D3 and should look familiar if you have studied other D3 (non-Angular) examples.

## Use the Component

Now let’s use the chart component. Edit the app component as follows:

```html
<h3>Bar Chart</h3>
<app-bar-chart *ngIf="chartData" [data]="chartData"></app-bar-chart>
```

The `*ngIf="chartData"` statement prevents the chart from being rendered before its data is available.

The test data is generated in the app component:

```typescript
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent implements OnInit {
  private chartData: Array<any>;

  constructor() {}

  ngOnInit() {
    // give everything a chance to get loaded before starting the animation to reduce choppiness
    setTimeout(() => {
      this.generateData();

      // change the data periodically
      setInterval(() => this.generateData(), 3000);
    }, 1000);
  }

  generateData() {
    this.chartData = [];
    for (let i = 0; i < (8 + Math.floor(Math.random() * 10)); i++) {
      this.chartData.push([
        `Index ${i}`,
        Math.floor(Math.random() * 100)
      ]);
    }
  }
}
```

In this case, a random array of numbers is regenerated every three seconds to demonstrate dynamic data updates. You can imagine a real-world scenario where this data is loaded from an API.

# Test

Now run the example with:

`ng serve`

![Angular D3.js](/assets/images/angular-d3js.gif)

### Get the Code

The source for this example is [available on GitHub](https://github.com/keathmilligan/angular-d3js).
