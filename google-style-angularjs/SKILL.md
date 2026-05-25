---
name: google-style-angularjs
description: Google AngularJS Style Guide — Closure integration, dependency injection, controllers, directives, services, and best practices for AngularJS with Closure at Google. Use when writing, reviewing, or refactoring AngularJS code in Closure-based projects. Triggers on any AngularJS coding task.
---

# Google AngularJS Style Guide

This guide is for **AngularJS (1.x)** with **Closure Compiler** at Google. Supersedes the JS style guide for Angular-specific code.

---

## Dependencies

- Use `goog.provide` / `goog.require` for module management (not `<script>` tags or AMD).
- Every Angular file provides one namespace and requires its direct dependencies.

```javascript
goog.provide('myapp.MyController');

goog.require('myapp.MyService');
goog.require('myapp.templates');
```

## Modules

- Main app module defined in the root client directory.
- **Do not alter modules outside their definition file.** Mutating a module in a separate file creates action-at-a-distance.
- Reference modules using Angular's `.name` property — never use string literals across files.

```javascript
// Module definition
myapp.module = angular.module('myapp', [ngRoute.name, myapp.MyService.module.name]);

// Referencing elsewhere
angular.module(myapp.module.name)
    .controller('MyController', myapp.MyController);
```

## Externs

- Use a common Angular externs file so the Closure Compiler understands Angular APIs.
- Provides type safety and avoids compiler renaming of Angular identifiers.

## Compiler

- Use `ANGULAR_COMPILER_FLAGS_FULL` preset.
- Mark any method or property exposed to Angular templates / expressions with `@export`.

```javascript
/**
 * @export
 * @return {string}
 */
MyController.prototype.getTitle = function() {
  return this.title;
};
```

## Controllers

- Controllers are ES3/ES5 classes with methods on the prototype.
- Always use **'controller as'** syntax in the template.
- Required annotations: `@constructor`, `@ngInject`, `@export`.

```javascript
/**
 * @constructor
 * @ngInject
 * @export
 */
myapp.MyController = function($http, myService) {
  /** @export */ this.data = [];
  this.http_ = $http;
  this.service_ = myService;
};

/** @export */
myapp.MyController.prototype.load = function() {
  this.http_.get('/data').then(function(response) {
    this.data = response.data;
  }.bind(this));
};
```

Template:
```html
<div ng-controller="myapp.MyController as ctrl">
  <button ng-click="ctrl.load()">Load</button>
</div>
```

## Directives

- DOM manipulation **only** in directives — never in controllers or services.
- Keep directives small and composable.
- `goog.provide` a static function that returns the directive definition object.

```javascript
goog.provide('myapp.myDirective');

/**
 * @return {angular.Directive}
 */
myapp.myDirective = function() {
  return {
    restrict: 'E',
    scope: {},
    bindToController: {
      items: '='
    },
    controller: myapp.MyDirectiveController,
    controllerAs: 'ctrl',
    templateUrl: '/path/to/template.html',
    link: function(scope, element, attrs, ctrl) {
      // DOM manipulation here only
    }
  };
};
```

## Services

- Favor `module.service` (class-based) over `.provider` or `.factory`.
- Required annotations: `@constructor`, `@ngInject`.

```javascript
/**
 * @constructor
 * @ngInject
 * @export
 */
myapp.MyService = function($http) {
  this.http_ = $http;
};

/** @return {!angular.$q.Promise} */
myapp.MyService.prototype.getData = function() {
  return this.http_.get('/api/data');
};
```

## Style

- Reserve the `$` prefix for Angular / jQuery built-in parameters and properties.
- **Do not** prefix your own properties or variables with `$`.

```javascript
// Good
myapp.MyController = function($scope, myService) {
  this.myService_ = myService;
};

// Bad
myapp.MyController = function($scope, $myService) {
  this.$myService = $myService;
};
```

## Testing

- Jasmine + Karma are the recommended testing framework and runner.
- Use `angular.mock.module` to load modules in tests.
- Use `angular.mock.inject` to inject dependencies.

```javascript
describe('MyController', function() {
  var $controller;

  beforeEach(module(myapp.module.name));
  beforeEach(inject(function(_$controller_) {
    $controller = _$controller_;
  }));

  it('should create controller', function() {
    var ctrl = $controller('MyController');
    expect(ctrl).toBeDefined();
  });
});
```

---

Full source: `G:\styleguide\angularjs-google-style.html`
