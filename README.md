# ng-caching
### FIRST SOLUTION  
Supported by Angular 4.3.0 and above does supports interceptors.  
A better way to prevent caching requests is to set necessary HTTP headers in Angular.  
You need to specifically set the following headers:
```
"Cache-Control": "no-cache"
"Pragma": "no-cache"
"Expires": "Sat, 30 Mar 2024 00:00:00 GMT"
```
These headers must be set on every request sent to the server, specialy ```GET``` or ```FETCH``` requests.

The ideal way to implement an [HttpInterceptor](https://angular.io/api/common/http/HttpInterceptor#description), which would allow us to intercept **every HTTP request** and set **the headers above**. The following ```CacheInterceptor``` is an implementation example of ```HttpInterceptor```, just put it in a typescript file.  

```
import { HttpEvent, HttpHandler, HttpHeaders, HttpInterceptor, HttpRequest } from "@angular/common/http";
import { Injectable } from "@angular/core";
import { Observable } from "rxjs";

@Injectable()
export class CacheInterceptor implements HttpInterceptor {
    intercept(req: HttpRequest, next: HttpHandler): Observable<HttpEvent> {
        if (req.method === "GET") {
            const httpRequest = req.clone({
                headers: new HttpHeaders({
                    "Cache-Control": "no-cache",
                    "Pragma": "no-cache",
                    "Expires": "Sat, 30 Mar 2024 00:00:00 GMT"
                })
            });
            return next.handle(httpRequest);
        }
        return next.handle(req);
    }
}
```

With this custom interceptor, we need to hook it to the root component of the Angular app, or to the corresponding module in case your app is distributed. It is most likely that the root module is called ```app.module.ts```.

Let's go ahead to our module and add the following in the providers array to begin using the ```CacheInterceptor```.
```
..
providers: [
    {
        provide: HTTP_INTERCEPTORS,
        useClass: CacheInterceptor,
        multi: false
    }
...

```

```multi``` option allows you to specify multiple interceptors, executing in the order specified in the providers array. Set it to false, means you don’t have another interceptor.  
You may have noticed the ```if``` block for checking for request method. Essentially, we want to limit adding headers to ```GET``` requests because, for example, Internet Explorer does not cache ```POST``` requests, but unfortunately others do, well this refers to the kind of browser you are using too !  

### SECOND SOLUTION  
Supported by Angular2, cause it doesn't have (yet) interceptors. You can instead extend ```Http```, ```XHRBackend```, ```BaseRequestOptions```.  

```
import { Injectable } from '@angular/core';
import { BaseRequestOptions, Headers } from '@angular/http';

@Injectable()
export class CustomRequestOptions extends BaseRequestOptions {
    headers = new Headers({
        'Cache-Control': 'no-cache',
        'Pragma': 'no-cache',
        'Expires': 'Sat, 30 Mar 2024 00:00:00 GMT'
    });
}
```
And your module : 
```
@NgModule({
    ...
    providers: [
        ...
        { provide: RequestOptions, useClass: CustomRequestOptions }
    ]
})
```
