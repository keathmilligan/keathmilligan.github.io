---
layout: post
title: Lazy-loading content with angular-cli
date: 2017-01-12 20:15:00 -0500
categories: [dev]
tags: [angular, typescript]
featured-img: angular.png
---

A quick example of creating a lazy-loaded module with angular-cli.


Lazy-loading can dramatically improve the performance of your Angular 2 application by loading content only when the user requests it. This is a simple example of lazy-loading using angular-cli.
<!--more-->

*April 30, 2017:* Updated for latest @angular/cli.

You will need a recent version of node/npm and the angular-cli tool installed globally. See https://cli.angular.io/.

# Create the App

Create the application scaffolding with:

`ng new --routing true angular-cli-lazyload`

Create a “home” component that will be the default (not lazy-loaded) route:

`ng g component Home`

# Add the Lazy-Loaded Content

Add the module to be lazy-loaded with:

`ng g module --routing true Lazy`

Add a component to be used as the router outlet for the lazy-loaded pages:

`ng g component --flat lazy`

Adding the --flat option here is a matter of structural preference. I like to put the “container” component source with the module declaration so it mirrors the main app module’s structure, but you can opt to place it in a subdirectory if you like.

Finally, add a lazy-loaded “page” component to the lazy module:

`ng g component lazy/LazyPage`

Edit the src/app/lazy/lazy-routing.module.ts file and add the routes to be lazy-loaded:

```typescript
import { NgModule } from '@angular/core';
import { Routes, RouterModule } from '@angular/router';
import { LazyComponent } from './lazy.component';
import { LazyPageComponent } from './lazy-page/lazy-page.component';
 
const routes: Routes = [
  {
    path: '',
    component: LazyComponent,
    children: [
      {
        path: 'lazypage',
        component: LazyPageComponent
      }
    ]
  }
];
 
@NgModule({
  imports: [RouterModule.forChild(routes)],
  exports: [RouterModule]
})
export class LazyRoutingModule { }
```

Edit the src/app/lazy/lazy.component.html file and add the router outlet:

```html
<h2>Lazy Module</h2>
 
<router-outlet></router-outlet>
```

# Update Application Routing

Edit the file src/app/app-routing.module.ts and add the default home page and lazy-loaded routes:

```typescript
import { ModuleWithProviders }         from '@angular/core';
import { Routes, RouterModule }        from '@angular/router';
import { HomeComponent }               from './home/home.component';
 
const appRoutes: Routes = [
  { path: '', component: HomeComponent },
  { path: 'lazy', loadChildren: './lazy/lazy.module#LazyModule' },
  { path: '**', redirectTo: '' }
];
 
export const appRoutingProviders: any[] = [
];
 
export const routing: ModuleWithProviders = RouterModule.forRoot(appRoutes);
```

Add some navigation to src/app/app.component.html:

```html
<h1>
  {{title}}
</h1>
 
<p>Navigation:
  <a [routerLink]="['/']">Home</a>
  <a [routerLink]="['lazy/lazypage']">Lazy Page</a>
</p>
 
<h3>Current Route:</h3>
 
<router-outlet></router-outlet>
```

# Test

Compile the app and start the angular-cli server with:

`ng serve`

In Chrome, open the developer tools window and select the “Network” tab so you can see all of the HTTP requests being made.

Now navigate to http://localhost:4200. You see the default home page and the navigation links.

![Angular Lazy Load Example](/assets/images/ng2lazyload.png)

Note the requests that were made to load the app in the debugger:

![Lazy Load Debugger](/assets/images/ng2lazyload-chrometools.png)

Now click the “Lazy Page” link. You should see the “lazy page works” message:

![Lazy Page](/assets/images/ng2lazyload-module.png)

In the debugger, note that an additional request was made to load “0.chunk.js” to render the lazy-loaded page:

![Lazy Load Chunk](/assets/images/ng2lazyload-chunk0.png)

### Source:

Source code on [Github](https://github.com/keathmilligan/angular-cli-lazyload).
