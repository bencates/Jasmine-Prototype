# jasmine-prototype

jasmine-prototype provides two extensions for [Jasmine](https://jasmine.github.io/) JavaScript Testing Framework:
  
- a set of custom matchers for Prototype elements
- an API for handling HTML fixtures in your specs

It is heavily based on [jasmine-jquery](http://github.com/velesin/jasmine-jquery) by Wojciech Zawistowski

## Prototype matchers

jasmine-prototype provides following custom matchers (in alphabetical order):

- `toBe(CssSelector)`
  - e.g. `expect($('<div id="some-id"></div>')).toBe('div#some-id')`
- `toBeAnElement()`
- `toBeChecked()`
  - only for tags that have checked attribute
  - e.g. `expect(new Element('input', { type: "checkbox", checked: true })).toBeChecked()` 
- `toBeDisabled()`
  - e.g. `expect(new Element('input', { type: "submit", disabled: true })).toBeDisabled()`
- `toBeEmpty()`  
- `toBeHidden()`
- `toBeSelected()`
  - only for tags that have selected attribute
  - e.g. `expect(new Element('option', { selected: true })).toBeSelected()`
- `toBeVisible()`
- `toContain(CssSelector)`
  - e.g. `expect(new Element('div').insert(<span class="some-class"></span>')).toContain('span.some-class')`
- `toHandle(eventName)`
- `toHandleWith(eventName, eventHandler)`
- `toHaveAttr(attributeName, attributeValue)`
  - attribute value is optional, if omitted it will check only if attribute exists
- `toHaveBeenTriggeredOn(selector)`
  - if event has been triggered on `selector` (see "Event Spies", below)
- `toHaveClass(className)`
  - e.g. `expect(new Element('div', { className: "some-class"})).toHaveClass("some-class")`  
- `toHaveData(key, value)`
  - value is optional, if omitted it will check only if an entry for that key exists
- `toHaveHtml(string)`
  - e.g. `expect(new Element('div').insert('<span></span>')).toHaveHtml('<span></span>')`
- `toHaveId(id)`
  - e.g. `expect(new Element('div', { id: "some-id"})).toHaveId("some-id")`
- `toHaveText(string)`
  - accepts a String or regular expression
  - e.g. `expect(new Element('div').insert('some text')).toHaveText(/text/)`
- `toHaveValue(value)`
  - only for tags that have value attribute
  - e.g. `expect(new Element('input', { type: "text", value: 'some text' })).toHaveValue('some text')`

The same as with standard Jasmine matchers, all of above custom matchers may be inverted by using `.not` prefix, e.g.:

    expect($('<div>some text</div>')).not.toHaveText(/other/)

## Fixtures

Fixture module of jasmine-prototype allows you to load HTML content to be used by your tests. The overall workflow is like follows:

In _myfixture.html_ file:

    <div id="my-fixture">some complex content here</div>

Inside your test:

    loadFixtures('myfixture.html');
    new MyTestedPlugin('my-fixture');
    expect($('my-fixture')).to...;

By default, fixtures are loaded from `spec/javascripts/fixtures`. You can configure this path: `jasmine.getFixtures().fixturesPath = 'my/new/path';`.

Your fixture is being loaded into `<div id="jasmine-fixtures"></div>` container that is automatically added to the DOM by the Fixture module (If you _REALLY_ must change id of this container, try: `jasmine.getFixtures().containerId = 'my-new-id';` in your test runner). To make tests fully independent, fixtures container is automatically cleaned-up between tests, so you don't have to worry about left-overs from fixtures loaded in preceeding test. Also, fixtures are internally cached by the Fixture module, so you can load the same fixture file in several tests without penalty to your test suite's speed.

To invoke fixture related methods, obtain Fixtures singleton through a factory and invoke a method on it:

    jasmine.getFixtures().load(...);

There are also global short cut functions available for the most used methods, so the above example can be rewritten to just:

    loadFixtures(...);

Several methods for loading fixtures are provided:

- `load(fixtureUrl[, fixtureUrl, ...])`
  - Loads fixture(s) from one or more files and automatically appends them to the DOM (to the fixtures container).
- `read(fixtureUrl[, fixtureUrl, ...])`
  - Loads fixture(s) from one or more files but instead of appending them to the DOM returns them as a string (useful if you want to process fixture's content directly in your test).
- `set(html)`
  - Doesn't load fixture from file, but instead gets it directly as a parameter (html parameter may be a string or a Prototype element, so both `set('<div></div>')` and `set(new Element('div'))` will work). Automatically appends fixture to the DOM (to the fixtures container). It is useful if your fixture is too simple to keep it in an external file or is constructed procedurally, but you still want Fixture module to automatically handle DOM insertion and clean-up between tests for you.
- `preload(fixtureUrl[, fixtureUrl, ...])`
  - Pre-loads fixture(s) from one or more files and stores them into cache, without returning them or appending them to the DOM. All subsequent calls to `load` or `read` methods will then get fixtures content from cache, without making any AJAX calls (unless cache is manually purged by using `clearCache` method). Pre-loading all fixtures before a test suite is run may be useful when working with libraries like jasmine-ajax that block or otherwise modify the inner workings of JS or Prototype AJAX calls.

All of above methods have matching global short cuts:

- `loadFixtures(fixtureUrl[, fixtureUrl, ...])`
- `readFixtures(fixtureUrl[, fixtureUrl, ...])`
- `setFixtures(html)`

Also, a helper method for creating HTML elements for your tests is provided:

- `sandbox([{attributeName: value[, attributeName: value, ...]}])`

It creates an empty DIV element with a default id="sandbox". If a hash of attributes is provided, they will be set for this DIV tag. If a hash of attributes contains id attribute it will override the default value. Custom attributes can also be set. So e.g.:

    sandbox();

Will return:

    <div id="sandbox"></div>    

And:

    sandbox({
      id: 'my-id',
      className: 'my-class',
      myattr: 'my-attr'
    });

Will return:

    <div id="my-id" class="my-class" myattr="my-attr"></div>

Sandbox method is useful if you want to quickly create simple fixtures in your tests without polluting them with HTML strings:

    setFixtures(sandbox({className: 'my-class'}));
    myClassRemoverFunction($('sandbox'));
    expect($('sandbox')).not.toHaveClass('my-class');

This method also has a global short cut available:

- `sandbox([{attributeName: value[, attributeName: value, ...]}])`

Additionally, two clean up methods are provided:

- `clearCache()`
  - purges Fixture module internal cache (you should need it only in very special cases; typically, if you need to use it, it may indicate a smell in your test code)
- `cleanUp()`
  - cleans-up fixtures container (this is done automatically between tests by Fixtures module, so there is no need to ever invoke this manually, unless you're testing a really fancy special case and need to clean-up fixtures in the middle of your test)

These two methods do not have global short cut functions.

## Event Spies and Mocks

Native events can be fired using the function `triggerEvent(element, event)`.

Spying on Prototype events can be done with `spyOnEvent` and
`assert(eventName).toHaveBeenTriggeredOn(selector)`. First, spy on the event:

    spyOnEvent($('some_element'), 'click');
    triggerEvent($('some_element'), 'click');
    expect('click').toHaveBeenTriggeredOn($('some_element'));
