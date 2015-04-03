# inject-type
Simple & neat IOC container for JavaScript, TypeScript and CoffeeScript

## Features
- neat API
  - no need to register classes
  - no need to inject dependencies with string identifiers
- available binding for a test purpose
- minification ready
- works in the front-end (bower support) and on node.js (npm support)

## Usage

### `inject()` function

`inject()` always provide the same instance of the given constructor:

```javascript
function Notifier() {
}

var notifierA = inject(Notifier);
var notifierB = inject(Notifier);

notifierA instanceof Notifier // true
notifierA === notifierB; // true
```

### Injecting service classes
For injecting singleton dependencies we always use `inject()`.

Example dependency injection in JavaScript
```javascript
function AlertService() {
}

AlertService.prototype.showAlert = function(text) {
  alert(text);
};


function Notifier() {
  this.alertService = inject(AlertService);
}

Notifier.prototype.warn = function(details) {
  this.alertService.showAlert(details);
};
```
Dependency injection in TypeScript.
```typescript
class AlertService {

  public showAlert(text:string):void {
    alert(text);
  }

}

class Notifier {

  private alertService:AlertService = inject(AlertService);

  public warn(details:string):void {
    this.alertService.showAlert(details);
  }

}
```

Dependency injection in CoffeeScript
```coffeescript
class AlertService

  showAlert: (text) ->
    alert(text)
  

class Notifier

  constructor: -> 
    this.alertService = inject(AlertService)

  warn: (details) ->
    this.alertService.showAlert(details)
  
```
### Testing services
So, how to test a service in an isolated environment? We have to mock all it's dependencies using `inject.bind()`. Example below uses jasmine, sinon and chai.

```javascript
it('calls alert to be shown', function() { 
  // given
  inject.bind(AlertService, alertServiceMock);
  var notifier = inject(Notifier);
  var warning = 'some warning';
  // when
  notifier.warn(warning);
  // then
  alertServiceMock.showAlert.should.have.been.calledWith(warning);
});
```
To reset bindings, we have to call `inject.resetBindings()`. It's useful to call this before each test, to make sure that they are independent. Example test setup:
```javascript
var alertServiceMock;
var notifier;
beforeEach(function() {
  inject.resetBindings();
  alertServiceMock = sinon.createStubInstance(AlertService);
  inject.bind(AlertService, alertServiceMock);
  notifier = inject(Notifier);
});
```
then our example test gets simpler:
```javascript
it('calls alert to be shown', function() { 
  // given
  var warning = 'some warning';
  // when
  notifier.warn(warning);
  // then
  alertServiceMock.showAlert.should.have.been.calledWith(warning);
});
```

### Creating new instance
To create new instance of some class on demand, we create factory service which uses injector to construct new instance.
```javascript
// defining product class

function Product (data) {
  this.name = data.name;
  this.size = data.size;
}

// defining factory service
function ProductFactory() {
  this.injector = inject(inject.Injector);
}

ProductFactory.prototype.createProductFrom = function(data) {
  return this.injector.resolve(Product, data);
}

```
But, why can't we create new instance just using `new Product(data)`? That's because we want to be able to test this statement.

### Testing factory services
To test factory service and the way it constructs our products, we have to mock the Injector and check calls on it.
```javascript
it('proviedes data to product constructor', function() { 
  // given
  inject.bind(inject.Injector, injectorMock);
  injectorMock.resolve.returns(productMock);
  var productFactory = inject(ProductFactory);
  var data = {some: 'data'};
  // when
  var result = productFactory.createProductFrom(data);
  // then
  injectorMock.resolve.should.have.been.calledWith(Product, data);
  result.should.equal(productMock);
});
```
