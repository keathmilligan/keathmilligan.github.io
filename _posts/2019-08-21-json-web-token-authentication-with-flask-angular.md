---
layout: post
title: JSON Web Tokens with Flask and Angular
date: 2019-08-21 17:52:16 -0500
categories: [dev]
tags: [security, angular, python, flask, typescript, jwt]
featured-img: flask-angular2-jwt.png
---

<img src="assets/images/flask-angular2-jwt.png" align="right">A simple end-to-end example of using [JSON Web Tokens](https://jwt.io/) (JWT) for authentication with token refresh in a Python Flask web server with an Angular front-end.
<!--more-->

This is an upate to an older post titled ["JWT authentication with Flask and Angular 2: a simple end-to-end example"](/jwt-authentication-with-flask-and-angular-2-a-simple-end-to-end-example) that provided a simple JWT example using Angular 2.x. In this update, we'll demonstrate JWT (and automatic token refresh) with the current versions of [Flask](https://palletsprojects.com/p/flask/) (1.1.1) and [Angular](https://angular.io/) (8.2.0).

The source code for this example can [found on Github](https://github.com/keathmilligan/angular-jwt-flask).

# The Server

> Refer to the server [README file](https://github.com/keathmilligan/angular-jwt-flask/tree/master/jwt_flask) for information on setting up and running the example server.

In this example, a [Flask](https://palletsprojects.com/p/flask/) web service will expose RESTful APIs using JWT for authentication. The [Flask-JWT-Extended](https://flask-jwt-extended.readthedocs.io/en/latest/) Flask extension is used to generate and validate the JSON web tokens.

## Authentication

To authenticate, the client posts the username/password to the `auth/login` API which calls the `authenticate_user` function in the `auth.py` service. This function checks the credentials and returns an *access_token* and a *refresh_token*.

### Access Tokens

The access token is required in order to access any resource that requires the user to have a valid login. Refer to the [JWT documentation](https://jwt.io/introduction/) for more information on access tokens.

### Refresh Tokens

This example is configured to issue access tokens that expire after 15 minutes. After the token expires, the client must use the refresh token (which has a much longer life-span - 30 days) to get a new access token by using the `auth/refresh` API. [This article](https://auth0.com/blog/refresh-tokens-what-are-they-and-when-to-use-them/) provides a pretty good explanation of this process.

## View Decorators

To control access to individual APIs, [Flask view decorators](https://flask.palletsprojects.com/en/1.1.x/patterns/viewdecorators) are used to verify the token supplied in the request header. For example:

```python
@app.route('/api/user-sample', methods=['GET', 'POST'])
@auth_required
def sample_api():
    """
    Example API
    """
    if request.method == 'GET':
        return make_response(jsonify({'example': 123}))
    elif request.method == 'POST':
        data = request.get_json()
        app.logger.debug('payload: %d', data['example'])
        return make_response(jsonify({'example': data['example'] * 2}))
    else:
        abort(405)
```

The `@auth_required` decorator insures that a valid access token has been supplied in the request's `Authorization` header. If the token is not present or is invalid, an "unauthorized" error (401) will be returned:

```python
def auth_required(func):
    """
    View decorator - require valid access token
    """
    @wraps(func)
    def wrapper(*args, **kwargs):
        verify_jwt_in_request()
        try:
            get_authenticated_user()
            return func(*args, **kwargs)
        except (UserNotFound, AccountInactive) as error:
            app.logger.error('authorization failed: %s', error)
            abort(403)
    return wrapper
```

> The Flask-JWT-Extended extension includes a built-in `@jwt_required` decorator that you could use as well, but this decorator only verifies that the token is valid. By providing your own implementation, you can perform additional checks to verify that the user account still exists, has not been disabled, etc.

## CORS

Because we will be running the development Flask server at a different URL than the Angular development server (http://localhost:5000 vs http://localhost:4200), the server needs to include some additonal headers to enable [cross-origin resource sharing](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS). Otherwise, the browser will refuse to make the request.

This is accomplished by using the `after_request` decorator to insert additonal headers into the response.

`jwt_flask/jwtapp/api/__init__.py`:
```python
@app.after_request
def after_request(response):
    """
    Post request processing - add CORS, cache control headers
    """
    # Enable CORS requests for local development
    # The following will allow the local angular-cli development environment to
    # make requests to this server (otherwise, you will get 403s due to same-
    # origin poly)
    response.headers.add('Access-Control-Allow-Origin',
                         'http://localhost:4200')
    response.headers.add('Access-Control-Allow-Credentials', 'true')
    response.headers.add('Access-Control-Allow-Headers',
                         'Content-Type,Authorization,Set-Cookie,Cookie,Cache-Control,Pragma,Expires')  # noqa
    response.headers.add('Access-Control-Allow-Methods',
                         'GET,PUT,POST,DELETE')

    # disable caching all requests
    response.cache_control.no_cache = True
    response.cache_control.no_store = True
    response.cache_control.must_revalidate = True
    return response
```

## Server To-Dos:

Some additional considerations for a real server:
* Authenticate using a real authorization service (Auth0, LDAP, etc.)
* Token revocation / black-listing for additional security

# The Client

> Refer to the client [README file](https://github.com/keathmilligan/angular-jwt-flask/tree/master/jwt_angular) for information on setting up and running the example web interface.

The client web application uses the [`@auth0/angular-jwt` library](https://github.com/auth0/angular2-jwt) to handle JWT tokens. This library includes a built-in HTTP interceptor, but in this example, we implement our own in order to provide *automatic token refresh*.

## Client Authentication

The client uses Angular [route guards](https://angular.io/guide/router#milestone-5-route-guards) to verify that the user has logged before attempting to request protected resources. If the user needs to login, they will be redirected to the `/login` route to login.

## Request Authorization

This example implements an `HttpClient` [request interceptor](https://angular.io/guide/http#intercepting-requests-and-responses) to automatically insert the `Authorizaton` header with the JWT `Bearer` token:

```typescript
@Injectable()
export class AuthInterceptor implements HttpInterceptor {

  constructor(private auth: AuthService,
              private router: Router) { }

  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    if (!/.*\/api\/auth\/.*/.test(req.url)) {
      return this.auth.getAccessToken().pipe(
        mergeMap((accessToken: string) => {
          const reqAuth = req.clone({ setHeaders: { Authorization: `Bearer ${accessToken}` } });
          return next.handle(reqAuth);
        }),
        catchError((err) => {
          console.error(err);
          this.router.navigate(['/login']);
          return throwError(err);
        })
      );
    } else {
      return next.handle(req);
    }
  }
}
```

## Automatic Token Refresh

The `getAccessToken` method in the `Auth` service will return the access token from `localStorage` if it is not expired. If it is expired, it will first attempt to refresh it by using the refresh token and the `auth/refresh` API:

```typescript
  // Get access token, automatically refresh if necessary
  getAccessToken(): Observable<string> {
    const accessToken = localStorage.getItem('accessToken');
    const refreshToken = localStorage.getItem('refreshToken');
    if (!this.jwt.isTokenExpired(accessToken)) {
      return new BehaviorSubject(accessToken);
    } else if (!this.jwt.isTokenExpired(refreshToken)) {
      console.log('refreshing access token');
      const opts = {
        headers: new HttpHeaders({
          Authorization: 'Bearer ' + refreshToken
        })
      };
      return this.http.post<RefreshResponse>(REFRESH_API, {}, opts).pipe(
        map(response => {
          localStorage.setItem('accessToken', response.accessToken);
          console.log('authentication refresh successful');
          return response.accessToken;
        })
      );
    } else {
      return throwError('refresh token is expired');
    }
  }
```

If the refresh token is also expired or the refresh attempt fails, the user will be redirected to the `/login` page.

## Additional Client Considerations

In this implementation scheme, there is a small possibility that the token could expire in between the time the client sends a request and the server checks the token. To avoid this possibility, the client could check the expiration date/time of the token and refresh it if it is within some minimum theshold (e.g. 1 minute).

### Source Code

<https://github.com/keathmilligan/angular-jwt-flask>
