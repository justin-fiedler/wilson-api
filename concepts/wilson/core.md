# Wilson Core

Each app that uses Wilson has a global "wilson" instance that is decorated onto the window object.
This is where your primary interfacing with Wilson will take place. To the developer, the global "wilson" 
instance should feel like a simple wrapper for angularjs declarations and a means of utility, logging, 
and config access (with a few exceptions for advanced use cases).

> A detailed typescript definition of wilson can be found [here](https://raw.githubusercontent.com/hightail/wilson/master/wilson.d.ts). NOTE: this file can be referenced in your 
> IDE to provide syntax highlighting and auto-completion for wilson projects.


Wilson Interface
====

* [setAppConfig](#setappconfig)
* [getActivePage](#getactivepage)
* [getActiveComponent](#getactivecomponent)
* [getActiveComponentList](#getactivecomponentlist)
* [component](#component)
* [behavior](#behavior)
* [service](#service)
* [factory](#factory)
* [resource](#resource)
* [class](#class)
* [utility](#utility)
* [filter](#filter)
* [config](#wilsonconfig)
* [utilities](../utilities/utilities.md)
* [log](../logging/logging.md)
  
  
## setAppConfig

Set the wilson instance config to a given object. This method takes a plain javascript object and sets it
as the config. Read more about the wilson config [here](#config). 

```typescript
function setAppConfig(config: Object): void;
```
Usage Example:
```js
wilson.setAppConfig({
  app: {
    version:    '1.0.0',
    name:       'MyApp',
    mountpath:  '/wilson'
  }
});
```



## getActivePage

Returns the name of the currently active page component. Effectively, which page are you on.

```typescript
function getActivePage(): string;
```


## getActiveComponent

Return relevant object references and information for a given active wilsonComponentId. The returned
object contains a stored reference to $scope and controller for the given wilsonComponentId. This is useful
for tying $element references back to our components without needing to use angularjs's [$compileProvider.debugInfoEnabled](https://docs.angularjs.org/guide/production) option.

```typescript
function getActiveComponent(componentId: string): Object;
```
Usage Example:
```js
// Assume we've have an instance of a component with a full name of 'ht-my-component' in the DOM

function someHandler() {
  var componentId = $('.ht-my-component').data('wilsonComponentId');
  var component   = wilson.getActiveComponent(componentId);
  
  console.log(component.name);        // --> The simple name of our component "my-component"
  console.log(component.scope);       // --> The isolateScope instance for this component
  console.log(component.controller);  // --> The controller instance for this component
}

```


## getActiveComponentList

Returns a list of all active component objects on wilson.

```typescript
function getActiveComponentList(): Object[];
```


## component

Declare a new component onto the wilson app module. This method is a wrapper for declaring a
"component-style" directive onto angularjs. Wilson components are intended to have isolateScope and
and explicit controller (which is ensured by wilson upon declaration). 

In terms of the config, all standard angularjs directive configuration properties are supported, please 
see the angularjs [directive api](https://docs.angularjs.org/api/ng/service/$compile). More specifics of 
Wilson components can be found [here](../components/components.md).

```typescript
function component(name: string, config: Object): void;
```
Usage Example:
```js
wilson.component('my-component', {
  controller: ['$scope', '$element', function($scope, $element) {
    
  }]
});
```


## behavior

Declare a new behavior onto the wilson app module. This method is a wrapper for declaring an
"attribute-style" directive onto angularjs. There are no restrictions or assumptions made for wilson
behaviors, all standard angularjs directive configuration properties are supported, please see the 
angularjs [directive api](https://docs.angularjs.org/api/ng/service/$compile). 

```typescript
function behavior(name: string, definition: any[]|Function): void;
```
Usage Example:
```js
wilson.behavior('my-behavior-directive', ['$window', function($window) {
  return {
    link: function($scope, $element, $attrs, controller) {
      $window.alert('my-behavior-directive has been activated!');
    }
  };
}]);
```


## service

Declare a new service onto the wilson app module. This method is wrapper for declaring
factory-style services on angularjs. Definitions for services are exactly the same as if they
were declared on angularjs directly. See the angularjs documentation for [$provide.factory](https://docs.angularjs.org/api/auto/service/$provide#factory). 

```typescript
function service(name: string, definition: any[]|Function): void;
```
Usage Example:
```js
wilson.service('AuthService', ['$http', function($http) {
  return {
    login: function(email, password) {
      return $http.post('/api/login', { email: email, password: password });
    }
  };
}]);
```


## factory

*Alias* for **service()**. Allows semantic distinction between the service types of a wilson app.

```typescript
function factory(name: string, definition: any[]|Function): void;
```


## resource

*Alias* for **service()**. Allows semantic distinction between the service types of a wilson app.

```typescript
function resource(name: string, definition: any[]|Function): void;
```


## class

*Alias* for **service()**. Allows semantic distinction between the service types of a wilson app.

```typescript
function class(name: string, definition: any[]|Function): void;
```


## utility

*Alias* for **service()**. Allows semantic distinction between the service types of a wilson app.

```typescript
function utility(name: string, definition: any[]|Function): void;
```


Wilson Config
====

An application using wilson is required to provide a configuration object to the wilson instance. This
config object contains properties that represent version, selector prefixes, mountpath for the wilson server
application and other values that allow each application to use wilson flexibly.

There are 4 required sections of this config from wilson's perspective:

| Section          | Description                   |
| ---------------- | ----------------------------- |
| **app**          | *Core configuration settings* | 
| **routes**       | *Application URL routes*      | 
| **tags**         | *Client identity info*        |
| **i18n**         | *i18next configuration*       |


It is recommended that apps using wilson adopt the wilson.config object as their primary client-side
app config. Any config-type data that an application requires during runtime can be designated onto this 
config up-front, and will then be accessible via the global wilson instance from anywhere.

The wilson server instance ultimately sets most of the values on the config upon serving it to the browser. Although
there is a method available to setAppConfig directly on the wilson instance, the config is pre-set when the config
module is fetched from the server (required as the first step of wilson bootstrapping).

Example Config:

```json
{
  "app": {
    "name":               "MyApplication",
    "version":            "1.0.0",
    "connectionFilters":  "EYJwhgdgJgvAzgBzAYwKZwD6gPYHc6ogzIAWI2AtqhlKgG4CWaMFKGANpAOYCuYXqGKggYEnAC4AzbCAoxacANbjsCIA"
  },
  "routes": [
    {
      "path":       "/",
      "component":  "home",
      "title":      "Home Page",
      "options":    {}
    },
    {
      "path":       null,
      "component":  "404",
      "title":      "Not Found",
      "options":    {}
    }
  ],
  "tags": {
    "brand":      "my-app",
    "browser":    "chrome",
    "device":     "mac",
    "platform":   "desktop",
    "language":   "en"
  },
  "i18n": {
    "clientSafeNamespaces":         "all",
    "defaultLng":                   "en",
    "fallbackLng":                  "en",
    "lng":                          "en",
    "localStorageExpirationTime":   604800000,
    "namespaces":                   [],
    "resStore":                     {},
    "sendMissing":                  true,      
    "supportedLngs":                ["en"],
    "useLocalStorage":              false
  }
}
```

## App Config

App settings are mostly descriptive, including the name and version of the app. The one notable setting here 
is the **connectionFilters** property.

**connectionFilters** is a hash of the keys/values in the **tags** section of the config. Specifically the server uses the
[lz-string](https://github.com/pieroxy/lz-string) library's [compressToBase64()](http://pieroxy.net/blog/pages/lz-string/guide.html#inline_menu_4) method
to create a hash that can be appended to server request URLs for component bundles (this allows the wilson server to serve back templates
based on the user's **tags**. NOTE: This is also vital for caching when using a CDN).

Code-based Explanation:
```js

var connectionFilters = wilson.config.app.connectionFilters;
var rawFilters        = LZString.decompressFromBase64(connectionFilters);
var tags              = wilson.config.tags;

console.log(connectionFilters);   // --> "EYJwhgdgJgvAzgBzAYwKZwD6gPYHc6ogzIAWI2AtqhlKgG4CWaMFKGANpAOYCuYXqGKggYEnAC4AzbCAoxacANbjsCIA"
 
console.log(rawFilters);          // --> "brand=spaces|browser=chrome|device=mac|language=en|platform=desktop"

console.log(tags.brand);          // --> "spaces"
 
console.log(tags.browser);        // --> "chrome"
 
console.log(tags.device);         // --> "mac"
 
console.log(tags.platform);       // --> "desktop"
 
console.log(tags.language);       // --> "en"
```


## Route Config

All URL routes for an application are defined on wilson.config.routes. At bootstrap time, wilson declares all routes onto 
angular using the [ngRoute module](). Similar to routing in the new angular.io, route entries on wilson map a URL "path" 
to a "component". 

#### [Detailed Wilson routing api](../routing/routing.md)


## Tags Config

The tags properties in the wilson config represent identifying information for the currently active client. This info 
includes device-type, platform, browser, language and even branding. This means that client wilson applications to not 
have to use javascript tactics to detect these characteristics, but can simply imply them from the global wilson config.


## i18n Config

The i18n settings follow the [i18next](https://github.com/i18next/i18next/tree/2.2.0) library configuration. Wilson provides 
i18n support via the use of this library both server-side and client-side. On the wilson server, i18next is used to pre-internationalie
markup templates before they make it to the client browser. On the client, wilson provides translation methods for dynamic
strings that are templated during runtime of the application.
