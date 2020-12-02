# Topic: *CSRF-Security*
### Author: *Galaju Margareta*
------

## Theory :
One of the very dangerous attacks that an adversary can perform is a Cross-Site Request
Forgery (CSRF) attack. A CSRF attack allows the adversary to perform unwanted actions on
a web application in which a user is currently authenticated.

## Objectives :
CSRF attacks bring the most damage when executed by a logged user (especially an admin).
To implement mechanisms of protection from CSRF, you’ll first need to create a web application
that would allow a user to authenticate themselves. After applying the defence mechanisms of
your choice, you’ll need to show their effectiveness via a video. To summarize, you’ll need to:
__1. Create a web application where the user can authenticate;__
__2. Implement CSRF protection for your web application;__
__3. Prove the effectiveness of implemented mechanisms (via video).__

## Used technologies:
 ASP.NET Core 3.1 & Angular 9

## Implementation:

The API has just two endpoints/routes to demonstrate authenticating with JWT and accessing a restricted route with JWT:

/users/authenticate - public route that accepts HTTP POST requests containing the username and password in the body. If the username and password are correct then a JWT authentication token and the user details are returned.
/users - secure route that accepts HTTP GET requests and returns a list of all the users in the application


__.net core middleware changes to generate and send XSRF token__
```

public void ConfigureServices(IServiceCollection services)
{
 services.AddAntiforgery(options =>{
 options.HeaderName = "X-XSRF-TOKEN";
 options.SuppressXFrameOptionsHeader = false;
 });
}
public void Configure(IApplicationBuilder app,IAntiforgery antiforgery)
{
   app.Use(next => context =>
    {
     if( context.Request.Path.Value.IndexOf("/api/Property/geocodes", StringComparison.OrdinalIgnoreCase) != -1)
{var tokens = antiforgery.GetAndStoreTokens(context);
context.Response.Cookies.Append("XSRF-TOKEN", tokens.RequestToken,new CookieOptions() {HttpOnly = false,Path="/"});
//setting HttpOnly=true,Secure=true will make browser cookie secure 
}
return next(context);
});

```

__Angular changes to intercept the request and append the xsrf token to the header.__

```
@Injectable()
export class InsertAuthTokenInterceptor implements HttpInterceptor {
 constructor(private adal: MsAdalAngular6Service, private     xsrfTokenExtractor: HttpXsrfTokenExtractor) {
}
intercept(req:HttpRequest<any>, next:HttpHandler) {
  let xsrfToken = this.xsrfTokenExtractor.getToken();
  if (req.method == "POST") {
    const authorizedRequest = req.clone({
      withCredentials: true,
      headers: req.headers.set("X-XSRF-TOKEN", xsrfToken)
    });
      return next.handle(authorizedRequest);
    } else {
     next.handle(req);
    }
  }
}
```
