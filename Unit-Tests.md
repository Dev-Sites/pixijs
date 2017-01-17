## Perserving Mocha Scope

Tests are written in Mocha, while the tests support ES6 features, it's important to retain the Mocha scope when writing tests. Avoid the use of fat-arrows. For instance:

```js
describe('ClassName', function() {
  it('methodName1', () => { // incorrect
    // cannot use "this" for Mocha API
  });

  it('methodName', function() { // correct
     this.timeout(4000); // preserve Mocha scope
  });
});
```