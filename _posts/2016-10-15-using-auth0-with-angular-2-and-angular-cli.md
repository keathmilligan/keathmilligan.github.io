---
layout: post
title: Using Auth0 with Angular 2 and angular-cli
date: 2016-10-15 16:14:00 -0500
categories: [dev]
tags: [angular, typescript]
featured-img: angular.png
---

<img src="/assets/images/angular.png" align="right">[Auth0‘s](https://auth0.com/) standard [Angular 2 quick-start](https://auth0.com/docs/quickstart/spa/angular2) uses SystemJS and loads the Auth0 Javascript files globally from index.html. That’s fine for demo purposes, but not ideal for production. I’ve created a [github repo](https://github.com/keathmilligan/angular2-cli-auth0-example) (and some additional notes in this [gist](https://gist.github.com/keathmilligan/92004bfb15d63f6989eb3ca738bd951f)) that demonstrates using Auth0 with an Angular 2 app generated with [angular-cli](https://github.com/angular/angular-cli). This example loads Auth0 as modules and lets Webpack bundle it with the rest of your Javascript.
<!--more-->

[Download the source](https://github.com/keathmilligan/angular2-cli-auth0-example).
