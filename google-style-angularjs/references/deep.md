# Google AngularJS Style Guide — Deep Reference

## 1. Background

This guide documents how AngularJS (1.x) is used at Google in conjunction with the Closure Compiler and Closure Library. It supplements the general JavaScript style guide with Angular-specific conventions. Engineers working on AngularJS code in Closure-based projects should follow this guide.

Key motivations:
- **Closure Compiler integration** requires explicit annotations (`@constructor`, `@ngInject`, `@export`) for proper type checking and minification.
- **`goog.provide` / `goog.require`** replaces script tags and AMD for dependency management.
- **Action-at-a-distance** is avoided — modules are defined in one place and not mutated elsewhere.
- **`controller as`** syntax aligns Angular templates with standard JavaScript class usage, making the code more readable and testable.

---

## 2. Angular Language Rules

### 2.1 Dependency Management with `goog.provide` / `goog.require`

Every Angular file must provide exactly one namespace and require all direct dependencies. This ensures the Closure Compiler can resolve the dependency graph at compile time.

```javascript
goog.provide('myapp.todos.TodoController');

goog.require('myapp.todos.TodoService');
goog.require('myapp.todos.TodoConstants');
```

**Rules:**
- One `goog.provide` per file, matching the file path and primary export.
- `goog.require` every type used directly in the file.
- Do not use `goog.require` for types only passed through (e.g., in `$http` responses).
- Never use `<script>` tags or AMD loaders for Angular code in a Closure project.

### 2.2 Modules — Define Once, Reference by `.name`

The Angular module is the unit of organization. Each application has a single root module defined in the client root directory.

```javascript
goog.provide('myapp');

myapp.module = angular.module('myapp', [
  'ngRoute',
  myapp.todos.TodoService.module.name
]);
```

**Critical rule: Do not alter modules outside their definition.** Never do:

```javascript
// BAD — action-at-a-distance
angular.module('myapp').controller('SomeController', ...);
```

Instead, if a module needs modification, do it in the file where it was defined, or use a separate module as a dependency.

Reference modules using `.name`:

```javascript
// GOOD
angular.module(myapp.module.name)
    .controller('SomeController', myapp.SomeController);

// BAD — fragile string literal
angular.module('myapp')
    .controller('SomeController', myapp.SomeController);
```

### 2.3 Externs File

An Angular externs file tells the Closure Compiler about Angular's API surface so it does not rename `$scope`, `$http`, `angular.module`, etc.

- Use a shared, versioned Angular externs file across the project.
- Keep it updated with the Angular version in use.
- The externs file covers all public Angular APIs; custom externs may be needed for third-party Angular libraries.

### 2.4 Compiler Flags

Use `ANGULAR_COMPILER_FLAGS_FULL` — a preset that includes all Angular-specific compiler optimizations and safety checks.

Key flags:
- `--angular_pass`: enables Angular-specific optimizations (inlineable annotations, chained calls).
- Property renaming is disabled for properties accessed in templates.
- `@export` is required on any method or property reachable from an Angular expression or template.

```javascript
/**
 * @export
 * @type {string}
 */
MyController.prototype.title;

/**
 * @export
 */
MyController.prototype.submit = function() {
  // exposed to template via ng-click="ctrl.submit()"
};
```

### 2.5 Controllers as Classes with `controller as`

Controllers are **constructor functions** with methods on the prototype. The `controller as` syntax binds the controller instance to a named variable in the template.

```javascript
/**
 * @constructor
 * @ngInject
 * @export
 */
myapp.todos.TodoController = function($http, todoService) {
  /** @export @type {!Array<!Object>} */
  this.todos = [];

  /** @private {!angular.$http} */
  this.http_ = $http;

  /** @private {!myapp.todos.TodoService} */
  this.todoService_ = todoService;
};

/** @export */
myapp.todos.TodoController.prototype.addTodo = function(title) {
  this.todoService_.create(title).then(function(todo) {
    this.todos.push(todo);
  }.bind(this));
};
```

**Template:**
```html
<div ng-controller="myapp.todos.TodoController as ctrl">
  <ul>
    <li ng-repeat="todo in ctrl.todos">{{todo.title}}</li>
  </ul>
  <button ng-click="ctrl.addTodo('New')">Add</button>
</div>
```

**Annotations required:**
- `@constructor` — marks the function as a class for Closure Compiler.
- `@ngInject` — enables Angular dependency injection annotations at compile time.
- `@export` — prevents renaming of properties/methods used in templates.

### 2.6 Directives — DOM Manipulation Only

Controllers and services must not perform DOM manipulation. All DOM access belongs in directives.

**Directive definition pattern:** A `goog.provide`'d static function returning a directive definition object.

```javascript
goog.provide('myapp.todos.TodoFocus');

/**
 * @return {angular.Directive}
 */
myapp.todos.TodoFocus = function() {
  return {
    restrict: 'A',
    link: function(scope, element, attrs) {
      scope.$watch(attrs.todoFocus, function(val) {
        if (val) {
          element[0].focus();
        }
      });
    }
  };
};
```

**Best practices:**
- Directives should be small and composable — one concern per directive.
- Prefer `scope: {}` (isolate scope) with `bindToController` over inherited scope.
- Use `controllerAs` in directives with isolate scope.
- Use `require` to communicate between directives on the same element.
- Compile function only when shared DOM manipulation is needed across instances.

### 2.7 Services — Prefer `module.service`

Use `module.service` which registers a constructor function (instantiated once). Avoid `.provider` (overkill for most cases) and `.factory` (encourages object literals over classes).

```javascript
goog.provide('myapp.todos.TodoService');

/**
 * @constructor
 * @ngInject
 * @export
 */
myapp.todos.TodoService = function($http) {
  /** @private {!angular.$http} */
  this.http_ = $http;
};

/** @return {!angular.$q.Promise} */
myapp.todos.TodoService.prototype.list = function() {
  return this.http_.get('/api/todos').then(function(res) {
    return res.data;
  });
};

// Registration
myapp.TodoService.module = angular.module('myapp.todos.TodoService', [])
    .service('TodoService', myapp.todos.TodoService);
```

**Why `module.service` over `.factory`:**
- Consistent with Closure's class-based patterns.
- Easier to test — the constructor is directly invocable.
- Clear lifecycle — instantiated once, lazily.
- Works naturally with `@constructor` and `@ngInject` annotations.

---

## 3. Angular Style Rules

### 3.1 Reserve `$` for Angular / jQuery Builtins

Angular and jQuery use the `$` prefix for their own identifiers (`$scope`, `$http`, `$q`, etc.). To avoid confusion:

```javascript
// GOOD — Angular builtins prefixed with $
myapp.MyController = function($scope, $http, $timeout) {
  this.scope_ = $scope;
};

// BAD — user service prefixed with $
myapp.MyController = function($scope, $myService) {
  this.myService_ = $myService;
};

// BAD — user property prefixed with $
this.$myData = [];
```

**Rule:** Any `$`-prefixed parameter or property is assumed to be an Angular or jQuery builtin. Never prefix your own identifiers with `$`.

### 3.2 Custom Elements in IE8

AngularJS 1.x supports IE8, but custom elements (`<my-directive>`) are not natively supported. Use attribute restrictions:

```javascript
// Works in IE8
<div my-directive></div>

// May fail in IE8
<my-directive></my-directive>
```

Use `restrict: 'A'` (attribute) or `restrict: 'C'` (class) instead of `restrict: 'E'` when IE8 support is required.

---

## 4. Tips, Tricks, and Best Practices

### 4.1 Testing with Jasmine + Karma and `angular.mock`

The standard testing setup uses Jasmine (test framework) and Karma (test runner). Angular provides the `angular.mock` module for test utilities.

**Loading modules in tests:**
```javascript
describe('TodoController', function() {
  var $controller;
  var $httpBackend;
  var ctrl;

  beforeEach(module(myapp.todos.TodoController.module.name));
  beforeEach(module('ngRoute'));

  beforeEach(inject(function(_$controller_, _$httpBackend_) {
    $controller = _$controller_;
    $httpBackend = _$httpBackend_;
  }));

  beforeEach(function() {
    ctrl = $controller('TodoController', {});
  });

  afterEach(function() {
    $httpBackend.verifyNoOutstandingExpectation();
    $httpBackend.verifyNoOutstandingRequest();
  });

  it('should load todos on init', function() {
    $httpBackend.expectGET('/api/todos').respond([{id: 1, title: 'Test'}]);
    ctrl.load();
    $httpBackend.flush();
    expect(ctrl.todos.length).toBe(1);
  });
});
```

**Key patterns:**
- Use `module()` (alias for `angular.mock.module`) to load modules.
- Use `inject()` (alias for `angular.mock.inject`) to get dependencies.
- Underscore wrapping (`_$controller_`) lets Angular strip underscores to match the parameter name.
- Use `$httpBackend` for mocking HTTP — always call `flush()` and verify.

### 4.2 App Directory Structure

```
client/
  app.js                  // Root module definition
  controllers/
    todo_controller.js
  directives/
    todo_focus.js
    todo_item.js
  services/
    todo_service.js
  filters/
    todo_filter.js
  templates/
    todo_list.html
    todo_item.html
```

- Files map 1:1 with `goog.provide` namespaces.
- Directory structure mirrors the namespace hierarchy.
- Template files (HTML) are typically compiled into `goog.provide`'d JavaScript via `ng-closure-runner` or similar.

### 4.3 Scope Inheritance Awareness

Angular's scope prototypal inheritance can cause subtle bugs:

```javascript
// BAD — ng-if creates child scope, assignment creates shadowing property
<div ng-if="ctrl.visible">
  <input ng-model="ctrl.name">
</div>
```

Use `controller as` to avoid scope inheritance issues entirely — properties are on the controller object, not the scope.

### 4.4 `@ngInject` for Dependency Injection Compilation

The `@ngInject` annotation enables the Angular Compiler pass to generate `$inject` property arrays, making the code minification-safe:

```javascript
/**
 * @constructor
 * @ngInject
 */
myapp.MyController = function($http, $scope) {
  // $inject: ['$http', '$scope'] is auto-generated at compile time
};
```

Without `@ngInject`, you must manually write:
```javascript
myapp.MyController.$inject = ['$http', '$scope'];
```

### 4.5 Additional Patterns

- **Controller lifecycle:** Use `$onInit`, `$onChanges`, `$onDestroy` (1.5+) for component lifecycle hooks.
- **One-way bindings:** Use `<` (1-way) and `&` (expression) over `=` (2-way) in `bindToController` to reduce watchers and data flow complexity.
- **Component helper:** In Angular 1.5+, prefer `.component()` over `.directive()` for new code — it provides a more opinionated, component-based API.

---

## 5. Best Practices Links

- [Angular Best Practices wiki (Google-internal)](https://sites.google.com/a/google.com/angularjs/best-practices)
- [AngularJS Meetup Video: Best Practices (Google-internal)](https://drive.google.com/...)
- Closure Compiler Angular Pass documentation (internal)
- AngularJS 1.x Developer Guide — Components (external)

---

## 6. Review Checklist

Use this checklist when reviewing AngularJS code:

- [ ] Every file has exactly one `goog.provide` and appropriate `goog.require` statements.
- [ ] Modules are defined in one place and referenced via `.name`, not string literals.
- [ ] No module mutation outside the definition file.
- [ ] Controllers use `controller as` syntax with `@constructor`, `@ngInject`, `@export`.
- [ ] Controllers do not contain DOM manipulation.
- [ ] Directives are small, single-purpose, and defined as `goog.provide`'d factory functions.
- [ ] Services use `module.service` with class constructors.
- [ ] All methods/properties exposed to templates are marked `@export`.
- [ ] User code does not use `$` prefix for non-Angular identifiers.
- [ ] Tests use `angular.mock.module` and `angular.mock.inject`.
- [ ] `ANGULAR_COMPILER_FLAGS_FULL` is configured in the build system.
- [ ] A shared Angular externs file is versioned and up to date.
- [ ] No 2-way binding (`=`) in `bindToController` unless strictly necessary.
- [ ] Templates reference controller properties via the `controller as` alias.
- [ ] No unused `$scope` — use `controller as` instead.

---

Full source: `G:\styleguide\angularjs-google-style.html`
