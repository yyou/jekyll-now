---
layout: post
title: A JSONP demo with ASP.NET Web API
---

JSONP is the technical to be used to request data from a server or service which resides in the different domain rather than the clients. There are a lot of posts explaining the idea of JSONP on Internet like Cameron Spear's blog [Exactly What IS JSONP?](https://cameronspear.com/blog/exactly-what-is-jsonp/). 

This post provides a demostration of JSONP with service implemented by ASP.NET WebAPI (I will user 'webapi' below to represent ASP.NET WEB API).

Create a webapi project (in this post, it's 'WebApplication3' here) in Visual Studio and rename the file 'Controllers\ValuesController.cs' to 'Controllers\UsersController.cs'. And remove all the methods and create a new public method as below.

```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Net;
using System.Net.Http;
using System.Web.Http;
using Newtonsoft.Json;

namespace WebApplication3.Controllers
{
    public class UsersController : ApiController
    {
        public HttpResponseMessage Get(string callback, string id)
        {
            //Query DB to get the user with given id. 
            //For simplicity, we just create a new user here
            var p = new User() { id = id, name = "john smith" };

            var jscode = callback + "(" + JsonConvert.SerializeObject(p) + ");";
            var response = new HttpResponseMessage(HttpStatusCode.OK);
            response.Content = new StringContent(jscode, System.Text.Encoding.UTF8, "text/plain");
            return response;
        }
    }

    public class User
    {
        public string id { get; set; }
        public string name { get; set; }
    }
}
```

This method is to return an user based on id sent from client. The value of variable 'callback' will be the client-side handler function.

Now we build the whole solution in VS and if there is no error, we'll deploy it in IIS. Since JSONP is the technical to break the limit of Same-Origin Policy. So we try to deploy this web api into a separate origin. From [Definition of an origin](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy), we know that if 2 URLs differ in any of these parts below, we can say that they are in different origin:

* protocol
* domain
* port

We try to create a separate origin in local development PC, the easist way is to create another website and bind it with another port number instead of default 80. So I create a new website in IIS with name 'anothersite' and bind port number 81 to the new site.

![IIS]({{ site.baseurl }}/images/2017-11-09-jsonp-webapi/iis.png)

Then we create a new Applicaton for the webapi project in the 'anothersite'.

![Create new website]({{ site.baseurl }}/images/2017-11-09-jsonp-webapi/iis2.png)

Then we create a HTML file index.html and save it into folder 'C:\inetpub\wwwroot' (default folder for Default WebSite in IIS).

```html
<!DOCTYPE html>

<html lang="en" xmlns="http://www.w3.org/1999/xhtml">
<head>
    <meta charset="utf-8" />
    <title></title>    
</head>
<body>
    <button id="id1">Click me</button>
    <script>    
        function dosometing(person) {
            alert("NAME=" + person.name);
        }
        
        var btn = document.getElementById('id1');
        btn.onclick = function () {
            var script = document.createElement('script');
            script.type = "text/javascript";
            script.src = "http://localhost:81/webapplication3/api/users?callback=dosometing&id=1";
            script.onload = function () {
                this.remove();
            };
            document.getElementsByTagName('body')[0].appendChild(script);
        };
    </script>
</body>
</html>
```

Now we open the html file in brower and click the button. However, we got error as below shows.

![Error]({{ site.baseurl }}/images/2017-11-09-jsonp-webapi/error.png)

This error is caused by the .NET framework version of the application pool of the newly-created 'anothersite'. The default .NET framework version of new application pool is v2.0. Change it to v4.0 will solve this problem.

![.NET Version]({{ site.baseurl }}/images/2017-11-09-jsonp-webapi/iis3.png)

Now we click the button and its behavior is as what we expect.

![Expected result]({{ site.baseurl }}/images/2017-11-09-jsonp-webapi/result.png)
