## Table of Contents

 1. [Why I should test?](#Why-I-should-test)
 2. [Testing good practices](#Testing-good-practices)
 3. [Classification of tests](#Classification-of-tests)
 4. [How to run tests](#How-to-run-tests)
 5. [Fake backend](#Fake-backend)
 6. [Unit tests](#unit-tests)
 7. [Integration tests](#Integration-tests)
 8. [Acceptance tests](#Acceptance-tests)
 9. [Page objects](#Page-objects)
 10. [Stubs](#Stubs)
 11. [Mocks](#Mocks)
 12. [Spies](#Spies)
 13. [Lookup & Register](#Lookup--Register)
 14. [Debugging](#Debugging)
 15. [Make sure that all assertions passed](#Make-sure-that-all-assertions-passed)
 16. [Test helpers](#Test-helpers)
 17. [Don’t overuse beforeEach/afterEach hooks](#Don’t-overuse-beforeEach/afterEach-hooks)
 18. [Blueprints](#Blueprints)
 19. [Simplify tests without implementation lock-in](#Simplify-tests-without-implementation-lock-in)
 20. [High-Level DOM Assertions for QUnit](#High-Level-DOM-Assertions-for-QUnit)
 21. [Check the flow](#Check-the-flow)
 22. [Type your code](#Type-your-code)
 23. [Time travelling](#Time-travelling)
 24. [Enforce locale](#Enforce-locale)
 25. [Accessibility / A11y](#Accessibility--A11y)
 26. [Engines](#Engines)
 27. [Animations](#Animations)
 28. [Visual testing](#Visual-testing)

# Why I should test?
In the beginning, it may sound like a waste of time for people who are not used to testing code, but in fact, it will pay off in the future when your code base grows. If your product will operate for months or even years, it means that you have to do manual testing of your app every time you introduce changes in the code. It is not a problem if you have dedicated resources for that purpose, but in the end, a lot of the work that is repeatable, instead of being automated, is handled over and over by someone introducing a possibility of making a mistake (we are just humans after all).

Now imagine that our team grows. A new team member who is not aware of every decision that other developers had made joins the team. He can break the already existing code base and there is a chance that someone can miss this breaking change and move it to the production code base if it is not well-tested. Of course, tests are not the answer for every possible error in our application, however, they can significantly reduce the changes that break the app over time. Also, they can bulletproof our business sensitive process - so we make sure that the happy paths that are critical for our app are untouched. Moreover, an important benefit is that we save time on manual testing because our continuous integration can provide feedback before anyone can look at the code change.

<p align="right"><a href="#Table-of-Contents">back to top :arrow_up:</a></p>

* * *

# Testing good practices
1. Write the test first, then implement the chunk of code that is tested. That will define the component acceptance criteria. In this step, we should focus on the interface of our component (for instance). Your code is like a puzzle, that could be easily replaced or totally rewritten under the hood - the only part that should stay consistent is the public interface and behaviour.
2. Follow SOLID principles in your implementation (https://en.wikipedia.org/wiki/Single_responsibility_principle). One of the benefits: if we change the implementation of a small chunk of code, only the tests that are related to this code can break - and we can easily fix all the problems instead of looking at huge parts of a code that can interact with our changes. It also reduces the changes in the code base - which simplifies tracking those changes for your teammates as well as the merging process in general.
3. Keep the setup as tiny and as explicit per each test as possible - avoid global before/after setup if it is not shared across all tests in the given file with tests. It also speeds up the process of writing tests, because you don’t have to worry about the “global” setup of your suite, instead of that you only focus on the “local” setup. It usually means that you also utilize your testing environment because you are not wasting resources when you don’t have to (for instance, global setup always sets a variable that is used in the scope of few tests instead of all tests).
4. Make sure you have tested all cases without false-positive effect (broken implementation can’t pass the tests - with false-positive it can).
5. Avoid using non-explicit dependencies, for instance - not publicly visible/defined dependencies of your code. Good example: our component shouldn’t rely on service objects that are globally injected in the application - they should be self-contained and independent, so we will avoid difficult debugging process and never forget about the proper setup of the application. Avoid coupling is the way to go, but sometimes you need to rely on other things - if you do so, try to be as transparent for other developers as possible.

<p align="right"><a href="#Table-of-Contents">back to top :arrow_up:</a></p>

* * *

# Classification of tests

In general, in terms of frontend development tests can be split into the following categories:

- **Manual tests** - which relies on critical scenarios so-called acceptance criteria and are requires human effort to check the given solution works as expected.
- **Acceptance tests** - automatic tests that check the whole application behaviour - similar to manual tests, however, they are usually more expensive at the beginning to introduce to the project.
- **Integration tests** - lighter than acceptance tests - usually tests integration between multiple parts of the application, for instance, interaction with the component that has some extra dependency involved. In terms of integration test, we are focusing on the behaviour of the given component - without knowing how it is implemented underneath.
- **Unit tests** - this type of automated tests are the fastest and are coupled with the implementation. With that said, if we are changing our implementation, for instance, our code has changed the inner or outer interface - the corresponding test should fail.

It is worth noting that there are subcategories available within the groups above, for instance:

- **Visual tests** - which can be automated and detects breaking changes in terms of the visual aspect of our application. Usually works on comparison of expected design of the application - based on the screenshot, which is compared to the current state of the application after changes were introduced.

There is a pretty good classification chart that describes the relationship between the cost of each type of test in correlation to business value:

<p align="center"><img src="http://1.bp.blogspot.com/-RQLtpTssY_o/UZ9CYqzflqI/AAAAAAAAAyo/9kIx6aGwSaU/s320/TestingTrianglePished.png"></p>

Find out more about testing clasification and Testing Pyramid: https://dzone.com/articles/testing-triangle-circle-and

<p align="right"><a href="#Table-of-Contents">back to top :arrow_up:</a></p>

* * *

# How to run tests

A few useful commands that you should know:

```
// run the suite in the terminal
$ ember test

// run the suite in the terminal in watch mode - you can also inspect the app in that mode
$ ember test -s

// filter the test suite and running only the tests that matches the filter
$ ember test -f 'some string'
```

Also, by default, test runner is available in /tests tab of your app: http://localhost:4200/tests

<p align="right"><a href="#Table-of-Contents">back to top :arrow_up:</a></p>

* * *

# Fake backend
Let's write a simple backend for our app so we can test it easily and we don't have to wait for the real backend to be up and running. Let's add a proper add-on:

```
$ ember install ember-cli-mirage
```

Let's set up the simplest possible endpoint, that will return some users for us via `/api/authors`:

```javascript
// mirage/config.js
export default function() {
  this.namespace = 'api';

  this.get('/authors', () => {
    return {
      authors: [
        {id: 1, name: 'John'},
        {id: 2, name: 'Doe'}
      ]
    };
  });
}
```

Make sure that your application adapter has the proper namespace property set:

```javascript
namespace: 'api'
```

More about mirage: https://www.ember-cli-mirage.com

<p align="right"><a href="#Table-of-Contents">back to top :arrow_up:</a></p>

* * *

# Unit tests
Let's generate a service test for our theme manager service:

```
$ ember generate service-test theme-switcher
```

Sample service that switches theme could look like that:

```javascript
// app/services/theme-switcher.js
import Service from '@ember/service';

export default Service.extend({
  darkMode: false,

  toggleDarkMode() {
    this.toggleProperty('darkMode')
  }
});
```

A test for it

```javascript
// tests/unit/services/theme-switcher-test.js
import { module, test } from 'qunit';
import { setupTest } from 'ember-qunit';

module('Unit | Service | Theme switcher', function(hooks) {
  setupTest(hooks);

  test('should switch dark mode on & off', function(assert) {
    const themeManager = this.owner.lookup('service:theme-switcher');

    // default
    assert.equal(themeManager.get('darkMode'), false);

    themeManager.toggleDarkMode(); // first run
    assert.equal(themeManager.get('darkMode'), true);

    themeManager.toggleDarkMode(); // second run
    assert.equal(themeManager.get('darkMode'), false);
  });
});
```

More info: https://guides.emberjs.com/release/testing/unit-testing-basics/

<p align="right"><a href="#Table-of-Contents">back to top :arrow_up:</a></p>

* * *

# Integration tests
Let's generate a component test for our user-details component:

```
$ ember generate component-test user-details
```

A sample component that prints info about our user & test for it can look as follows:

```handlebars
<!-- app/components/user-details/template.hbs -->
{{user.name}} | Tasks: {{user.tasks.length}} | Comments: {{user.comments.length}}
```

```javascript
// tests/integration/component/user-details-test.js
import { module, test } from 'qunit';
import { setupRenderingTest } from 'ember-qunit';
import { render } from '@ember/test-helpers';
import hbs from 'htmlbars-inline-precompile';

module('Integration | Component | user details', function(hooks) {
  setupRenderingTest(hooks);

  test('it renders', async function(assert) {
    assert.expect(1);

    this.set('user', {
        name: 'John Doe',
        tasks: { length: 5 },
        comments: { length: 3 }
    });

    await render(hbs`{{user-details user=user}}`);

    assert.dom(this.element).hasText('John Doe | Tasks: 5 | Comments: 3');
  });
});
```

More info: https://guides.emberjs.com/release/testing/testing-components/

<p align="right"><a href="#Table-of-Contents">back to top :arrow_up:</a></p>

* * *

# Acceptance tests
Let's prepare an automatic test for the creation of a new user. After user creation, app redirects to the listing on `/users` route.

We can use a blueprint to create a new acceptance test:

```
$ ember generate acceptance-test add-user
```

Sample test can look as follows:

```javascript
import { module, test } from 'qunit';
import { click, fillIn, visit } from '@ember/test-helpers';
import { setupApplicationTest } from 'ember-qunit';

module('Acceptance | users', function(hooks) {
  setupApplicationTest(hooks);

  test('should add a new user', async function(assert) {
    await visit('/users/new');
    await fillIn('input.name', 'John Doe');
    await click('button.submit');

    assert.dom('ul.users li').hasText('John Doe');
    assert.equal(currentURL(), '/users');
  });
});
```

More info: https://guides.emberjs.com/release/testing/acceptance/

More info about `test-helpers` you can find in [API reference](https://github.com/emberjs/ember-test-helpers/blob/master/API.md)

<p align="right"><a href="#Table-of-Contents">back to top :arrow_up:</a></p>

* * *

# Page objects

Page objects offer an extra layer for our acceptance tests that simplifies them, improves readability and allows to write tests quicker. Page objects hide the implementation details behind the interface of class so-called page-object.

Let's install the proper addon:

```
$ ember install ember-cli-page-object
```

Let's take our last acceptance test and clean it up with page object:

```javascript
// tests/page-objects/users/new.js
import { create, visitable, fillable, clickable, text } from 'ember-cli-page-object';

export default create({
  visit: visitable('/users/new'),

  username: fillable('input.name'),

  submit: clickable('button.submit'),
  error: text('.errors')
});
```

```javascript
// tests/page-objects/users.js
import { create, visitable, fillable, clickable, text } from 'ember-cli-page-object';

export default create({
  visit: visitable('/users'),

  firstUser: text('ul.users:first-child')
});
```

```javascript
// tests/acceptance/users-test.js
import { module, test } from 'qunit';
import { click, fillIn, visit } from '@ember/test-helpers';
import { setupApplicationTest } from 'ember-qunit';
import usersNewPage from '../page-objects/users/new';
import usersPage from '../page-objects/users';

module('Acceptance | users', function(hooks) {
    setupApplicationTest(hooks);
    test('my awesome readable login test', async function(assert) {
      await usersNewPage
        .visit()
        .username('John Doe')
        .submit();

      assert.equal(currentURL(), '/users');
      assert.equal(usersPage.firstUser(), 'John Doe');
    });
});
```

More info: http://ember-cli-page-object.js.org

<p align="right"><a href="#Table-of-Contents">back to top :arrow_up:</a></p>

* * *

# Stubs
We can stub methods that are heavy, require complex setup and usually are not the clue of our tests, but can improve their readability a lot.

```javascript
import sinon from 'sinon';

const store = this.owner.lookup('service:store');
sinon.stub(store, 'getUsersTeams').returns(store.peekAll('user'));
```

https://sinonjs.org/releases/latest/stubs/

<p align="right"><a href="#Table-of-Contents">back to top :arrow_up:</a></p>

* * *

# Mocks
Quite often in tests, we need only simple objects rather than instances of full implementation of the given class. A good example: models where we are using mirage objects instead of Ember Data models, that differ with their inner implementation (which speeds up tests).

If we have a mirage in our application, we have a globally available server that can help us create new models for us.

```javascript
const user = {
        id: '1',
        firstName: 'Test',
        lastName: 'User'
      };
const mockedUser = server.create('user', user);
```

You can even mock complete objects with Sinon (without Mirage) or use plain JS objects:

https://sinonjs.org/releases/latest/mocks/

<p align="right"><a href="#Table-of-Contents">back to top :arrow_up:</a></p>

* * *

# Spies
Spy allows you to check if your method was called with proper params.

Example:

```javascript
let setValuesFunction = sinon.spy();
this.set('setValuesFunction', setValuesFunction);

await doSomething();

assert.equal(setValuesFunction.calledWith([1, 2, 3]), true);
```

https://sinonjs.org/releases/latest/spies/

<p align="right"><a href="#Table-of-Contents">back to top :arrow_up:</a></p>

* * *

# Debugging

QUnit brings us a utility method that helps us examine and stop the execution of our spec:

```javascript
await this.pauseTest();
```

Whenever we will put it, we are able not just to see the intermediate state of the application inside the certain spec, but also we can just use a browser console to do some extra checks and experiments.

<p align="right"><a href="#Table-of-Contents">back to top :arrow_up:</a></p>

* * *

# Lookup & Register

Let's imagine the scenario that we want to test the behaviour of our `users/` page that is accessible only for administrators. Our apps use `ember-simple-auth` for authentication as well as ember-can. We don't want to go through the whole login process in each acceptance test, because it slows down the test and increases the risk of breaking many tests when only the login page is broken.


### What can we do about it?

Usually, our logic relies on the state of service objects. We need to make sure that we will stub a proper method or override a proper variable of the specific service object. How can we achieve that? We can use lookups. A lookup can find a reference of the specific factory for us so in a test, we can do something like this:

```javascript
this.owner.lookup('service:store') // returns store
this.owner.lookup('component:my-component') // returns specific component
```

Ok, what if we don't have the service in place in our application but it is still needed. We can register our own service easily. We can do the same for other types such as components, controllers, helpers etc. How? By using `.register` method:

```javascript
this.owner.register('service:my-service', EmberObject.create({})
this.owner.register('component:my-component' Component.create({}) // etc...
```

<p align="right"><a href="#Table-of-Contents">back to top :arrow_up:</a></p>

* * *

# Make sure that all assertions passed

QUnit allows us to make sure that all assertions passed in the given test:

```javascript
assert.expect(2);
```

It is very useful, for instance, if we stub some method, where we put assertion in, to make sure that we called the method on an object or we need to do some extra check inside the stubbed method on the object that it provides (for instance, when we stub an action that should receive some complex object). Sometimes, in such scenarios, we can use spy instead (for simpler objects).

Example:

```javascript
module('when skiping', function() {
  test('fires the on-skip action', async function(assert) {
    assert.expect(1);

    this.set('onSkip', (step) => {
      assert.equal(step, this.steps[0])
    });

    await render(hbs`{{intro-js steps=steps start-if=true on-skip=(action onSkip)}}`);

    await introJSSkip();
  })
});
```

<p align="right"><a href="#Table-of-Contents">back to top :arrow_up:</a></p>

* * *

# Test helpers
Quite often bigger packages allow their own set of helpers that are very useful in our tests.

## Example:

```javascript
import {
  typeInSearch,
} from 'ember-power-select/test-support/helpers';

await typeInSearch('Test phrase');
```

This allows to easily set up the search phrase into ember power select input. When you see that an add-on misses a test helper that could be useful, don't hesitate to contribute to it and add it on your own!

<p align="right"><a href="#Table-of-Contents">back to top :arrow_up:</a></p>

* * *

# Don’t overuse beforeEach/afterEach hooks

Those hooks should be fulfilled with setup only for the most shared part of the given test file. If you need a set up that is very specific to only a single test in your test file, you should always put it into the specific test, rather than in the general setup, or else, it can lead to many problems and reduce the readability of the test. Also, it makes it harder to change tests, because you can break some of them when you modify those hooks.

<p align="right"><a href="#Table-of-Contents">back to top :arrow_up:</a></p>

* * *

# Blueprints
You can save your time with blueprints. The blueprints available are listed here:

https://github.com/emberjs/ember.js/tree/master/blueprints

Some useful ones in terms of tests:

```
$ ember generate component-test component-name
$ ember generate route-test route-name
$ ember generate util-test util-tested-thing-name
$ ember generate acceptance-test acceptance-tested-thing-name
$ ember generate service-test service-name
$ ember generate helper-test helper-name
```

<p align="right"><a href="#Table-of-Contents">back to top :arrow_up:</a></p>

* * *

# Simplify tests without implementation lock-in

First, we need to install the proper add-on:

```
ember install ember-test-selectors
```

What `ember-test-selectors` does, it allows us to add a proper data attribute within our templates which will work as an identifier in our tests. The cool thing about it is the fact, that we won't have to rely on id's or classes of our DOM elements that can change over time. Instead of that, we will use data attributes that will be enabled only in the testing environment (in the production build they are stripped) and even if we change the styling of our component it can still work without any extra changes in the test - which is the goal.

In your templates you are now able to use `data-test-*` attributes, which are automatically removed from production builds:

```handlebars
<article>
  <h1 data-test-post-title data-test-resource-id="{{post.id}}">{{post.title}}</h1>
  <p>{{post.body}}</p>
</article>
```

Once you've done that you can use attribute selectors to look up the elements:

```javascript
// in Acceptance Tests:
find('[data-test-post-title]')
find('[data-test-resource-id="2"]')

// in Component Integration Tests:
this.querySelector('[data-test-post-title]').click()
this.querySelector('[data-test-resource-id="2"]').click()
```

More info on how to use it can be found here: https://github.com/simplabs/ember-test-selectors

<p align="right"><a href="#Table-of-Contents">back to top :arrow_up:</a></p>

* * *

# High-Level DOM Assertions for QUnit

To clean up your tests, you should use `qunit-dom` add-on:

```bash
$ npm install --save-dev qunit-dom
# or
$ yarn add --dev qunit-dom
```

It allows you to change this:

```javascript
assert.equal(this.querySelector('#title').textContent, "My header");
```

into this:

```javascript
assert.dom('#title').hasText('My header');
```

Full documentation can be found here: https://github.com/simplabs/qunit-dom

The especially useful part for us: https://github.com/simplabs/qunit-dom/blob/master/API.md

<p align="right"><a href="#Table-of-Contents">back to top :arrow_up:</a></p>

* * *

# Check the flow

Sometimes you need to verify the sequence of calls in your code. There is a very handy tool available that could handle this check for you and its called **steps**.

Let's take a look at the example:

```javascript
const displayResults = () => assert.step(['displayResults']);
const validateForm = () => assert.step(['validateForm']);
const prepareData = () => assert.step(['prepareData']);

const submitForm = () => {
  prepareData();
  validateForm();
  displayResults();
}

submitForm();

assert.verifySteps([
  'prepareData',
  'validateForm',
  'displayResults'
]);
```

You can also imagine async use cases that could benefit from **steps**.

<p align="right"><a href="#Table-of-Contents">back to top :arrow_up:</a></p>

* * *

# Type your code

It may not be connected with testing at a first glance, but using a type system for your code can be a sort of testing as well.

In the Ember ecosystem, we have an add-on that adds easy support for **TypeScript**. We can write our code in **.ts** and then it will be automatically compiled into a compatible js code with a type check.

It may be very useful for unit-level tests, where part of the issues may be only connected to an incorrect type of the variable passed to the interface. Thanks to strong typing, you will get feedback from your code before the runtime.

More info can be found in an add-on repo: https://github.com/typed-ember/ember-cli-typescript

<p align="right"><a href="#Table-of-Contents">back to top :arrow_up:</a></p>

* * *

# Time travelling

From time to time, we need to ensure that our code will behave differently depending on the actual date context. With that said, for instance, our component presents date in the custom date format and we want to make sure it will work well in any given month - we need to let our test runner know that we are time traveling to the given date. How to do it? We can use a proper add-on that wraps **Timecop** library:

```
ember install ember-cli-timecop
```

More info on how to use it can be found here: https://github.com/jamesarosen/Timecop.js

You may also consider alternative: https://github.com/Ticketfly/ember-mockdate-shim

<p align="right"><a href="#Table-of-Contents">back to top :arrow_up:</a></p>

* * *

# Enforce locale

A similar case to the one above, may happen in terms of the locale of our app - to enforce the proper locale we can use **setupIntl** or **setLocale** from **Intl** add-on.

More details with examples: https://github.com/ember-intl/ember-intl/blob/fde23d17ec7b3d7d8df602778a6c0c79c060e1fd/docs/testing.md

<p align="right"><a href="#Table-of-Contents">back to top :arrow_up:</a></p>

* * *

# Accessibility / A11y

Our apps should also support accessibility features so, for instance, our application supports screen readers well. Luckily, there are a ton of ready to use tools that we can use for that. Furthermore, there is a dedicated organization on Github that combines every well-documented a11y add-ons guidelines in one place: https://github.com/ember-a11y.

Ok, but is there anything that we can do right now even without diving into all the documentation details and get some meaningful feedback about the current state of the accessibility in our application? Actually, yes.

Let's install a proper add-on:

```bash
ember install ember-a11y-testing
```

Now, in our acceptance test we can use audit helper:

```javascript
import a11yAudit from 'ember-a11y-testing/test-support/audit';
// ...
test('accessibility check of the our app root', async function (assert) {
  await visit('/');
  await a11yAudit();
  assert.ok(true, 'no a11y errors found!');
});
```

As a good starting point, I recommend starting from the following article by **Jen Weber**: https://www.emberjs.com/blog/2018/06/17/ember-accessibility-and-a11y-tools.html

<p align="right"><a href="#Table-of-Contents">back to top :arrow_up:</a></p>

* * *

# Engines

When you deal with `ember-engines` and you would like to test your engine code for instance: components, you need to make sure that in test environment your dummy application has access to the certain engine component. To make sure it does, you need to replace the default resolver, to proper engine resolver from an addon:

```javascript
import engineResolverFor from 'ember-engines/test-support/engine-resolver-for';
```

To use it, you pass it as a param to your **setup** call, for instance in case of unit tests you replace default:

```javascript
setupTest(hooks);
```

to:

```javascript
setupTest(hooks, {
  resolver: engineResolverFor('your-engine-name')
});
```

<p align="right"><a href="#Table-of-Contents">back to top :arrow_up:</a></p>

* * *

# Animations

When we need to make sure that our app finished all the animations we need to use the proper test helper. Don't worry, `ember-test-helpers` covers this case. To do so, just use

```javascript
await settled()
```

We can even get more information about the state of our app when animations are present - more info can be found [here](https://github.com/emberjs/ember-test-helpers/blob/master/API.md#settled)

<p align="right"><a href="#Table-of-Contents">back to top :arrow_up:</a></p>

* * *

# Visual testing

Guess what - Ember got you covered again and thanks to Percy with the [proper add-on](https://github.com/percy/ember-percy), visual regression testing is extremely easy to integrate with our acceptance tests. All you need to do to create and compare snapshot in terms of your acceptance test is to add within the specific test the following code:

```javascript
percySnapshot('homepage');
```

Of course, it also requires some additional setup related to the Percy service itself. To find out more, check out the [Percy page](https://docs.percy.io/docs/ember).

Also, you can check out the alternative **ember-backstop** described in [the article](https://www.linkedin.com/pulse/ember-backstop-visual-regression-testing-tutorial-garris-shipon/).

<p align="right"><a href="#Table-of-Contents">back to top :arrow_up:</a></p>

* * *
