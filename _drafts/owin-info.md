OWIN

History
- System.Web.dll (aka ASP.NET)
- Primarily WebForms
- Monolithic framework
- Heavily dependent on IIS
-- IIS isn't a problem per-se
-- It's just that System.Web is "slow" and monolithic

Open Web Interface for .NET (OWIN)
- It's just a spec. for building a web server.
- How the server exposes a request to code.
- These modules are called "middleware".
- Heavily abstracted from the host/server.

The Spec:
- exposes an "Environment" which is the current HTTP request. Provided to code.
-- modeled as an IDictionary<string,object>
-- strings are keys to lookup something about the request/response
-- the spec defines what should be returned for each string
- AppFunc, plugin, how code is run in this environment
-- The "middleware" must expose a funct to call: Func<IDictionary<string,object>, Task>
-- Async by design
-- And that's pretty much it!

Katana
- MS implementation of OWIN spec
- Includes IIS or self-host
- Convience classes (OwinContext/OwinRequest/etc)
-- similiar API to System.Web's HTTPContext/Request/Response
- Set of middleware for common features - authentication, static content
-- This will be how MS will deliver new features for ASP.NET

OWIN Defines an Architecture
- Host: It's the "process".
- Server: implements HTTP and the OWIN API.
- Middleware: plugs into Server via registration.

Programming Model
- Requires startup class with: public void Configuration(IAppBuilder app)
- Register middleware in IAppBuilder
-- either via Run() which accepts Func<IOwinContext, Task> as the handler
-- or Use() which accepts Func<IOwinContext, Func<Task>, Task> which allows you to insert a handler tha calls Next();
-- or use Middleware "module". A class that implements a convention based on core API
-- middleware can be mapped to a specific path, allows for branched pipeline with separate configuration
-- Base References: Owin, Microsoft.Owin
-- IIS Host: Microsoft.Owin.Host.SystemWeb
