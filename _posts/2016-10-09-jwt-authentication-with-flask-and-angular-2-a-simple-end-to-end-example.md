---
layout: post
title: "JWT authentication with Flask and Angular 2: a simple end-to-end example"
date: 2016-10-09 13:26:00 -0500
categories: [dev]
tags: [flask, angular, jwt, python, javascript, typescript]
featured-img: flask-angular2-jwt.png
---

<img src="/assets/images/flask-angular2-jwt.png" align="right">JSON Web Tokens are a standard method of securing exchanges between two parties (such as web server app and client) that has a number of advantages over other methods of securing exchanges such as cookie-based sessions – enhanced security, less overhead and statelessness to name a few.
<!--more-->

In this post, well take a look at using JWT with Flask and Angular 2 and build a simple end-to-end example.

[Download or clone the complete source code for this example from github](https://github.com/keathmilligan/flask-ng2-jwt-example).

# The Server

First, let’s start with the server. To provide JWT functionality, I’m using Flask-JWT. This extension enhances Flask by providing a @jwt_required decorator we can attach to any routes we want to protect. The example server code below is pretty similar to the quick start example shown in the Flask-JWT documentation.

*flask-jwt/app.py:*
```python
from flask import Flask, jsonify, request
from flask_jwt import JWT, jwt_required, current_identity


class User(object):
    def __init__(self, id, username, password):
        self.id = id
        self.username = username
        self.password = password

    def __str__(self):
        return "User(id='%s')" % self.id

user = User(1, 'user', 'password')


def authenticate(username, password):
    if username == user.username and password == user.password:
        return user

def identity(payload):
    return user


app = Flask(__name__)
app.debug = True
app.config['SECRET_KEY'] = 'super-secret'

jwt = JWT(app, authenticate, identity)


# send CORS headers
@app.after_request
def after_request(response):
    response.headers.add('Access-Control-Allow-Origin', '*')
    if request.method == 'OPTIONS':
        response.headers['Access-Control-Allow-Methods'] = 'DELETE, GET, POST, PUT'
        headers = request.headers.get('Access-Control-Request-Headers')
        if headers:
            response.headers['Access-Control-Allow-Headers'] = headers
    return response


@app.route('/unprotected')
def unprotected():
    return jsonify({
        'message': 'This is an unprotected resource.'
    })


@app.route('/protected')
@jwt_required()
def protected():
    return jsonify({
        'message': 'This is a protected resource.',
        'current_identity': str(current_identity)
    })
```

Let’s break down the app a bit:

The first thing we do is define a simple user model to represent an “identity”:

```python
class User(object):
    def __init__(self, id, username, password):
        self.id = id
        self.username = username
        self.password = password

    def __str__(self):
        return "User(id='%s')" % self.id

user = User(1, 'user', 'password')
```

In a real world example, you would probably be connecting to a database, directory or some other external service for a user, but for the sake of this simple example, we’ll just hard code a single user.

Next, we declare a couple of functions that Flask-JWT will use to work with our user object:

```python
def authenticate(username, password):
    if username == user.username and password == user.password:
        return user

def identity(payload):
    return user
```

The `authenticate` function is called by Flask-JWT when the login API is invoked with a username and password. This function authenticates the user and returns a user object if successful (or None if not).

The `identity` function is called by Flask-JWT to look up a user by id. In this case, we simply return the one user.

Refer to the [official docs](https://pythonhosted.org/Flask-JWT/) for more information on the Flask-JWT API.

Create and configure the Flask app and create the jwt object:

```python
app = Flask(__name__)
app.debug = True
app.config['SECRET_KEY'] = 'super-secret'

jwt = JWT(app, authenticate, identity)
```

The SECRET_KEY configuration item is used to digitally-sign the tokens, so in a real-world scenario, set this to some appropriately difficult to guess value (e.g., a random string of characters) and ensure that it is not possible for users to access it. See *How to generate good secret keys* in the [Flask Sessions documentation](http://flask.pocoo.org/docs/0.11/quickstart/#sessions).

## Send CORS Headers

Since the debug instance of the Flask server will be running at “http://localhost:5000” and the Angular 2 front-end will be served at “http://localhost:3000”, we need to have the server send some additional headers to enable Cross-Origin Resource Sharing (CORS). Without these headers, the browser will refuse to load any data from the Flask server because it is running at a different URL due to the same-origin policy.

```python
@app.after_request
def after_request(response):
    response.headers.add('Access-Control-Allow-Origin', '*')
    if request.method == 'OPTIONS':
        response.headers['Access-Control-Allow-Methods'] = 'DELETE, GET, POST, PUT'
        headers = request.headers.get('Access-Control-Request-Headers')
        if headers:
            response.headers['Access-Control-Allow-Headers'] = headers
    return response
```

We could use Flask’s ability to serve static assets to serve the Angular 2 app and avoid having to add the CORS headers, but if we did that, we’d lose automatic rebuild on file changes and other benefits we gain from using node development server.

Finally, let’s declare some resources:

```python
@app.route('/unprotected')
def unprotected():
    return jsonify({
        'message': 'This is an unprotected resource.'
    })

@app.route('/protected')
@jwt_required()
def protected():
    return jsonify({
        'message': 'This is a protected resource.',
        'current_identity': str(current_identity)
    })
```

The first resource is unprotected and can be accessed without a token. The second requires a valid token and return a 403 without it.

# The Front End

Now let’s have a look at at the front-end code. Angular 2 apps require several configuration files, I’m not going to go through each of these as they are well covered [elsewhere](https://angular.io/docs/ts/latest/quickstart.html). All files necessary to run the example are available in the [github repo](https://github.com/keathmilligan/flask-ng2-jwt-example).

In this example, I’m going to use the angular2-jwt library from auth0. While it isn’t absolutely necessary to use a library for JWT support in your Angular app — you could simply treat the token as opaque and generate the headers yourself — the [angular2-jwt library](https://github.com/auth0/angular2-jwt) provides some nice functionality, including the ability to decode tokens, check their expiration dates, etc. Note that angular2-jwt will work with any JWT backend, an auth0 account is not required.

*app.module.ts:*
```typescript
import { NgModule }       from '@angular/core';
import { BrowserModule }  from '@angular/platform-browser';
import { HttpModule }     from '@angular/http';
import { provideAuth }    from 'angular2-jwt';
import { AppComponent }   from './app.component';

@NgModule({
  imports: [ BrowserModule, HttpModule ],
  providers: [
    provideAuth({
      headerPrefix: 'JWT'
    })
  ],
  declarations: [ AppComponent ],
  bootstrap: [ AppComponent ]
})
export class AppModule { }
```

The main thing we are doing here besides including the augular2-jwt providers is configuring its prefix to “JWT”, which is what Flask-JWT uses as a default.

*app.component.ts:*

```typescript
import { Component, OnInit } from '@angular/core';
import { Http, Response, RequestOptions, Headers } from '@angular/http';
import { AuthHttp, JwtHelper } from 'angular2-jwt';
import 'rxjs/add/operator/map';

const APP_SERVER = 'http://localhost:5000/';
const USERNAME = 'user';
const PASSWORD = 'password';

@Component({
  selector: 'my-app',
  template: `
    <h1>Flask/Angular 2 JWT Example</h1>
    <div *ngFor="let message of messages">
      {{ message }}
    </div>
  `
})
export class AppComponent implements OnInit {
  private messages: Array<string> = [];

  constructor(private http: Http,
              private authHttp: AuthHttp) { }

  ngOnInit() {
    this.getUnprotected();
  }

  // get the unprotected resource
  getUnprotected() {
    this.messages.push('Requesting unprotected resource');
    this.http
      .get(APP_SERVER + 'unprotected')
      .map((response: Response) => response.json())
      .subscribe(
      (data) => {
        this.messages.push(`Got unprotected response: ${data.message}`);

        this.login();
      },
      (error) => {
        this.messages.push(`Unprotected request error: ${error}`);
      }
      );
  }

  // send username/password and get the token
  login() {
    let options: RequestOptions = new RequestOptions({
      headers: new Headers({ 'Content-Type': 'application/json' })
    });
    this.messages.push('Logging in');
    this.http
      .post(APP_SERVER + 'auth',
      JSON.stringify({ 'username': USERNAME, 'password': PASSWORD }),
      options)
      .map((response: Response) => response.json())
      .subscribe(
      (data) => {
        // save the token in local storage
        let token = data.access_token;
        localStorage.setItem('id_token', token);
        this.messages.push(`Login successful, token saved.`);

        let jwtHelper: JwtHelper = new JwtHelper();
        this.messages.push(`expiration: ${jwtHelper.getTokenExpirationDate(token)}`);
        this.messages.push(`is expired: ${jwtHelper.isTokenExpired(token)}`);
        this.messages.push(`decoded: ${JSON.stringify(jwtHelper.decodeToken(token))}`);

        // now get the protected resource
        this.getProtected();
      },
      (error) => {
        this.messages.push(`Login failed: ${error}`);
      }
      );
  }

  // get a protected resource the the token
  getProtected() {
    this.messages.push('Requesting protected resource');
    this.authHttp
      .get(APP_SERVER + 'protected')
      .map((response: Response) => response.json())
      .subscribe(
      (data) => {
        this.messages.push(`Got protected resource: msg: ${data.message}, identity: ${data.current_identity}`);
      },
      (error) => {
        this.messages.push(`Protected request failed: ${error}`);
      }
      );
  }
}
```

This is a very simple component that displays a message log. The first thing it does is make a regular unauthenticated request to the unprotected resource. This is just to insure the basics are working:

```typescript
  ngOnInit() {
    this.getUnprotected();
  }

  // get the unprotected resource
  getUnprotected() {
    this.messages.push('Requesting unprotected resource');
    this.http
      .get(APP_SERVER + 'unprotected')
      .map((response: Response) => response.json())
      .subscribe(
      (data) => {
        this.messages.push(`Got unprotected response: ${data.message}`);

        this.login();
      },
      (error) => {
        this.messages.push(`Unprotected request error: ${error}`);
      }
      );
  }
```

If this request is successful, we proceed with trying to login:

```typescript
  // send username/password and get the token
  login() {
    let options: RequestOptions = new RequestOptions({
      headers: new Headers({ 'Content-Type': 'application/json' })
    });
    this.messages.push('Logging in');
    this.http
      .post(APP_SERVER + 'auth',
      JSON.stringify({ 'username': USERNAME, 'password': PASSWORD }),
      options)
      .map((response: Response) => response.json())
      .subscribe(
      (data) => {
        // save the token in local storage
        let token = data.access_token;
        localStorage.setItem('id_token', token);
        this.messages.push(`Login successful, token saved.`);

        let jwtHelper: JwtHelper = new JwtHelper();
        this.messages.push(`expiration: ${jwtHelper.getTokenExpirationDate(token)}`);
        this.messages.push(`is expired: ${jwtHelper.isTokenExpired(token)}`);
        this.messages.push(`decoded: ${JSON.stringify(jwtHelper.decodeToken(token))}`);

        // now get the protected resource
        this.getProtected();
      },
      (error) => {
        this.messages.push(`Login failed: ${error}`);
      }
      );
  }
```

If the login is successful, we save the token in local storage and then use some of the utilities that angular-jwt provides to show some information about the token we just received.

Finally, we request the protected resource using angular-jwt’s AuthHttp provider:

```typescript
  // get a protected resource the the token
  getProtected() {
    this.messages.push('Requesting protected resource');
    this.authHttp
      .get(APP_SERVER + 'protected')
      .map((response: Response) => response.json())
      .subscribe(
      (data) => {
        this.messages.push(`Got protected resource: msg: ${data.message}, identity: ${data.current_identity}`);
      },
      (error) => {
        this.messages.push(`Protected request failed: ${error}`);
      }
      );
  }
```

AuthHttp works just like Angular’s standard Http provider, but will automatically include the “Authorization:” header with the token we previously saved.

If everything works as expected, you should see something like this in your browser:

![Flask/Angular 2 Example](/assets/images/flask-ng2-jwt-example.png)
