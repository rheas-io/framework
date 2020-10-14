<p align='left'>
<a href='https://www.kaysy.io/' target='_blank'>
<img src="https://raw.githubusercontent.com/rheas-io/framework/master/images/rheas-full.svg" width="250px">
</a>
</p>
<p align='left'>
<a href="https://www.npmjs.com/package/@rheas/framework"><img src="https://img.shields.io/npm/dm/@rheas/framework" alt="Downloads"></a>
<a href="https://www.npmjs.com/package/@rheas/framework"><img src="https://img.shields.io/npm/v/@rheas/framework" alt="Version"></a>
<a href="https://www.npmjs.com/package/@rheas/framework"><img src="https://img.shields.io/npm/l/@rheas/framework" alt="License"></a>
</p>

<br>

## About Rheas

Rheas is a NodeJs framework created for developing web applications faster using an elegant syntax. This project is heavily inspired from the [Laravel](https://www.laravel.com/) framework.

## Why Rheas?

Rheas was created to solve one problem that we faced when working with other NodeJs frameworks - the need to install a handful of dependencies to show a single, functional web page.

- To handle cookies, we had to find and install one cookie middleware package.
- To handle sessions, we had to find and install another package.
- To handle authentication, we had to find and install another package.

The list goes on for many other core features that are essential for serving a website.

Rheas solves this for you. Rheas has built in functionality for almost all the features for running a web application like session handling, cookie handling, authentication, cache etc.

## Getting started

Creating a web server using Rheas framework is easy and it does not install any global packages. All the modules including the framework's CLI is scoped to project.

### Project initialization

Create a blank directory for your new project and initialize your npm project using 

```
npm init
```

### Installation

**To install the framework using npm**

```
npm install @rheas/framework
```

This package installs all the core modules needed to create a fully functional web server application. After installation, the framework will copy a `rheas` file on the current working directory, which acts as an entry point of the command line interface of the framework.

### Create new project

Rheas framework comes with a command line interface which can be used to do various operations like creating a new project, creating application keys, creating models, controllers etc.

**To create a new project, run**

```
node rheas new <project-name>
```

The above command will create a new project by creating all the core files and directories required for running the server and updates the `package.json` file with some `scripts` and `devDependencies`.

**Note:** The `new` project command won't replace any files if it already exists on the current working directory. So, it is safe to run the command, if you have accidentally deleted some core files in the project directory and want it back.

### Running the server

The `new` project command will update the `scripts` in the `package.json` file with commands required to build the project and start a server.

Rheas uses Typescript and thus the application has to be transpiled first before we can run it.

**Build the project**

```
npm run tsc
```

**Run the server**

```
npm run start
```

If you are using the default configurations, you will see the following message on your terminal.

`Listening on port 3000`

## Routes and controllers

When a new project is created using the CLI, a controllers directory `app/controllers` and a routes directory `routes` containing `api.ts` and `web.ts` files are created on the project root.

The `web.ts` file looks like

```
import { IRoute } from '@rheas/contracts/routes';

const webRoutes: IRoute[] = [
    // All the web routes
];

export default webRoutes;
```

As you can see, the default export is a collection of routes. Add new routes to this collection and it will be registered on the router when the application is loaded.

**Register a GET request to the homepage**
```
import { Route } from '@rheas/routing';
import { IRoute } from '@rheas/contracts/routes';

const webRoutes: IRoute[] = [
    Route.get('/', 'HomeController@index')
];

export default webRoutes;
```

In the above example `HomeController@index` is the controller action registered to the route, where `HomeController` depicts a file `HomeController.ts` in the `app/controllers` directory and `index` is the exported function.

When a request to the homepage `/` reaches our server, the router will execute the function named `index` on the `app/controllers/HomeController.ts` file.

**`/app/controlers/HomeController.ts`**

```
import { IRequest, IResponse } from '@rheas/contracts';

export async function index(request: IRequest, response: IResponse) {
  return response.set('Hello world!');
}
```

The above controller sends a **Hello World!** string back to the client as a response.

> A controller and middleware functions should always return the response object. To facilitate this, the content setting methods like `response.set()`, `response.json()` and `response.view()` returns the response object.

**Controller action on the route**

For something as simple as sending a response as shown in the above example, creating a file for just one controller function is an overkill. So, we support direct function registration on the Route itself.

```
import { Route } from '@rheas/routing';
import { IRoute } from '@rheas/contracts/routes';

const webRoutes: IRoute[] = [
    Route.get('/', async (req, res) => res.set('Hello World!))
];

export default webRoutes;
```

The above route will work in the same way as the example shown earlier.

## Architecture

The architecture of this framework is heavily inspired from Laravel, Symfony and ExpressJs. The API is designed in such a way that the learning curve is low and the developers with experience working on any of the above said framework's can transition smoothly.

### Lifecycle

Unlike PHP Laravel, lifecycle of a Rheas web application can be split into two - application lifecycle and request lifecycle.

Processes like connecting to the database, listening to each requests, managing queues, managing logs or in short the services that are independent of requests are all happening within the application lifecycle. This lifecycle begins when the Rheas application is statrted and ends when the application is terminated or when the process exits.

When it comes to request lifecycle, it manages the cookie handling, session handling, redirect services etc. From a frameworks point of view, we can say that the lifecycle of a Request is from the moment a request reaches our server and ends when a response is sent back.

> The actual request lifecycle begins when a new Request object aka `IncomingMessage` object is created by the node `http` or `https` server and is terminated only when it is garbage collected by NodeJs. So be wary of the references you keep on the Request object.

All the services/bindings registered on the application instance will persist until the application is terminated. And all the services/bindings registered on a request instance will persists on it until it (the request object) is garbage collected.

### Controller and Middleware design

Even though the framework uses ES6 classes throughout the framework, we have kept the exposed API to use Typescript pure functions only.

Most of the time, developers just have to work with controllers, middlewares and models only. Controllers and middlewares are pure functions, so that developers coming from an ExpressJs background would feel comfortable.

##### Controller function definition

```
import { StringObject } from '@rheas/contracts; 
import { IRequest, IResponse } from '@rheas/contracts';

export async function index(req: IRequest, res: IResponse, params: StringObject) {
  
  // Controler operations

  return res;
}
```

A controller function accepts three arguments - `request`, `response` and an optional `params` field. The third argument contains all the route parameter keys mapped to their corresponding value in the request.

Most of the time CRUD operations are written on a single file and thus Rheas resolves controller function using names. This is not the case with middlewares.

##### Middleware function definition

```
import { IRequest, IResponse } from '@rheas/contracts';
import { IRequestHandler } from '@rheas/contracts/routes; 

async function handle(req: IRequest, res: IResponse, next: IRequestHandler, ...params: any) {
  
  // Middleware operations

  return await next(req,res);
}

export default handle;
```

A middleware function accepts the `request` and `response` objects as the first two arguments. The third argument `next` is the next middleware in the pipeline. All the middlewares should **wait** for the next middlewares to finish processing. 

If we don't call the next middleware or if we don't wait for the next middleware promise to resolve, the middleware would return immediately breaking the whole middleware pipeline and sending invalid response to the client.

> Rheas resolves only the **default** export in a middleware file. So each middleware should be in a seperate file.

### Containers

Dependency injection is an essential design pattern to follow when we have to work with large number of seperate modules and services. So Rheas also supports dependency injection. That said, Rheas **does not** implement a full fledged dependency injector, yet. Why?

Because, our exposed API of controllers and middlewares are pure functions and they have a strict definition which accepts the `request` container as a parameter.

As mentioned in the Lifecycle section, Rheas framework has two lifecycles - an application lifecycle and a request lifecycle. To facilitate this, we have designed our `Application` instance as a container and each Request object is also a container.

When a request lifecycle begins, Rheas automatically registers the application container as a binding on the request container. This gives the request container access to all the services/bindings registered on the application container also.

So a contoller and middleware function can gather the dependencies from the `request` paramater.

```
import { IEncrypter } from '@rheas/security';
import { IRequest, IResponse } from '@rheas/contracts';

export async function index(req: IRequest, res: IResponse) {
  
  // Gets the encrypter service.
  const encrypter: IEncrypter = req.get('encrypt');

  const encryptedValue = await encrypter.encrypt('Hello world!');

  return res.set(encryptedValue);
}
```

In the above example, we needed the encryption service registered on the application container to return an encrypted value as response.

`req.get('encrypt');` will try to find any binding on the request container for the alias `encrypt` and if it is not available on the request container, it will search for one on the application container. If no binding is found, an exception will be thrown.

> For testing, you can inject a binding on the request container for the required aliases.

### Services
Services are the core part or in fact the central unit of Rheas framework. We can add new features and functions using services and the one we don't need can be removed as well. Thus, services provides expandability to the framework.

The framework comes with many core services like router, sessions handler, encrypter, hasher, cookies handler and several others to quickly create and run a web application server.

You can also create a new service and add it to the application.

A service can be registered on the application lifecycle or request lifecycle or on both. This is done by updating the config files.

> When you register a service on both the application and request lifecycle, they don't share the same service instance. A new service instance is created for each requests.

**`/configs/app.ts`**

```
providers: {
        // Core services required by the framework.
        ...
        ...
        ...
        
        // Register your service providers in here.
        
        newService : NewServiceProvider
}
```

Similarly there is a config file for request as well, which contains all the service provider mapped to their alias.

> Only the services available in these config files will be registered on the lifecycles. And, since the service provider list is a Javascript object, no two services can exist with same alias.

Even though we cache the service provider list when the app/request lifecycle begins, we don't initialize them until the application asks for it ie, services are lazy loaded.

Say you have a service `FetchUsers` mapped to the alias `fetch.users` and have it registered on the providers list of `/configs/request.ts`. An instance of the `FetchUsers` service will not be created until you call `request.get('fetch.users')` somewhere in your application. Once called, the instance will be cached and will be re-used throughout the request lifecycle.

#### Service Providers

Service providers are responsible for creation of services. Every service provider receives two parameters when they are initialized - service alias and the container in which the service has to be registered. Rheas injects these parameters automatically.

**`Rheas default Cookie service provider`**

```
import { IRequest } from '@rheas/contracts';
import { CookiesManager } from './cookiesManager';
import { ServiceProvider } from '@rheas/services';
import { InstanceHandler } from '@rheas/contracts/container';

export class CookieServiceProvider extends ServiceProvider {
    /**
     * Returns the cookies service resolver. The service returns
     * a cookies manager which parses incoming cookies, adds new
     * one's to the queue etc. Cookies service is registered on the
     * request cycle.
     *
     * @returns
     */
    public serviceResolver(): InstanceHandler {
        return (request) => new CookiesManager(request as IRequest);
    }
}
```

Rheas frameworks' in-built cookie service provider is shown above. As you can see the base class `ServiceProvider` takes away all the abstraction and only the service resolver is exposed on this class.

**What this service provider do?**

Say, you register this on the request lifecycle with the alias `cookie`. When you executes `request.get('cookie')` for the first time in the current lifecycle, the framework will create a singleton binding on the request container for the `CookieManager` object and maps it to the key `cookie`. The parameter of the `serviceResolver` is the the container in which the service has to be registered, in this case `request`.

The underlying `register` function of the base `ServiceProvider` class is 

```
public register(): void {
    this.container.singleton(this.name, this.serviceResolver());
}
```

If you don't want to use a singleton, override the `register` function on your service provider class.

## Roadmap

- Extensive testing of all the modules before releasing to production.
- Add commands to the CLI that creates controllers, models, services, middlewares etc.
- Create a detailed documentation on the framework.
- Add datastore backed queue handler. In this version, jobs are kept in memory until dispatched.
- Add datastore backed sessions. In this version files are used to store sessions.
- Add the authentication module to the framework before release.
- Move CPU intensive operations like encryption and decryption to worker threads.
- Add error view files before release.
- Add logging module before release.

## Sponsor

[Kaysy LLC](https://www.kaysy.io/) sponsors the development of this framework. Kaysy is an internet company working on SaaS products and Open-Source softwares.

## Community

**Author -** [Kalesh Kaladharan](https://twitter.com/kalesh_13/)  
**Twitter -** [Kaysy LLC](https://twitter.com/kaysyio/)

## License

The Rheas framework is open-sourced software licensed under the [MIT license](https://opensource.org/licenses/MIT).