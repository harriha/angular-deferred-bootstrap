# angular-deferred-bootstrap [![Build Status](https://travis-ci.org/philippd/angular-deferred-bootstrap.svg?branch=master)](https://travis-ci.org/philippd/angular-deferred-bootstrap)

> Initialize your AngularJS app with constants loaded from the back-end.

This component provides a global resolve function for your app. It works similar to the resolve functions you may know from ngRoute or ui-router: You define what needs to be loaded from the back-end before your application can be started and the deferred bootstrapper takes care of loading the data and bootstrapping the application.

## Install

#### [Bower](http://bower.io)

```
bower install --save angular-deferred-bootstrap
```

## Usage

Instead of using the ng-app directive or angular.bootstrap(), use the deferredBootstrapper to initialize your app:
```js
deferredBootstrapper.bootstrap({
  element: document.body,
  module: 'MyApp',
  resolve: {
    APP_CONFIG: function ($http) {
      return $http.get('/api/demo-config');
    }
  }
});
```

This will make the response of your $http call available as a constant to your AngularJS app. This means you can now use dependency injection to access the APP_CONFIG wherever you need it in your app (even in config() blocks!).
```js
angular.module('MyApp', [])
  .config(function (APP_CONFIG) {
    console.log('APP_CONFIG is: ' + JSON.stringify(APP_CONFIG));
  })
```

## Loading and error indicators
To make it possible to conditionally show a loading indicator or an error message when the initialization fails, the following CSS classes are set on the HTML body:

* **deferred-bootstrap-loading** while the data is loading
* **deferred-bootstrap-error** if an error occurs in a resolve function and the app can not be bootstrapped

Have a look at the demo pages to see this in action.

## Advanced usage
You can have multiple constants resolved for your app and you can do in the resolve function whatever is necessary before the app is started. The only constraint is, that the function has to return a promise. It is important to note, that the arguments passed to your resolve functions are NOT dependency injected. You get access to the following services in the resolve function: $http, $q, injector

To handle exceptions when the promises are resolved, you can add an onError function to the configuration object.

Example:
```js
deferredBootstrapper.bootstrap({
  element: document.body,
  module: 'MyApp',
  resolve: {
    APP_CONFIG: function ($http) {
      return $http.get('/api/demo-config');
    },
    OTHER_CONSTANT: function ($http, $q, injector) {
      var deferred = $q.defer(), $timeout = injector.get('$timeout');
      $timeout(function () {
        deferred.resolve('MyConstant');
      }, 2000);
      return deferred.promise;
    }
  },
  onError: function (error) {
	alert('Could not bootstrap, error: ' + error);
  }
});
```

## Testing
Since the constants that deferredBootstrapper adds to your applications module are not available in your unit tests, it makes sense to provide them in a global beforeEach():
```js
beforeEach(function () {
  module(function ($provide) {
    $provide.constant('APP_CONFIG', { someUrl: '/dummyValue' });
  });
});
```

## License

[MIT](http://opensource.org/licenses/MIT) © Philipp Denzler
