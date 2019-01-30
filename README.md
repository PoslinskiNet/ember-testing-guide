## Table of Contents

 1. [Why I should test?](#Why-I-should-test?)
 2. [Testing good practices](#Testing-good-practices)
 3. [How to run tests](#How-to-run-tests)
 4. [Fake backend](#Fake-backend)
 5. [Unit tests](#unit-tests)
 6. [Integration tests](#Integration-tests)
 7. [Acceptance tests](#Acceptance-tests)
 8. [Page objects](#Page-objects)
 9. [Stubs](#Stubs)
 10. [Mocks](#Mocks)
 11. [Spies](#Spies)
 12. [Lookup & Register](#Lookup--Register)
 13. [Make sure that all assertions passed](#Make-sure-that-all-assertions-passed)
 14. [Test helpers](#Test-helpers)
 15. [Don’t overuse beforeEach/afterEach hooks](#Don’t-overuse-beforeEach/afterEach-hooks)
 16. [Blueprints](#Blueprints)
 17. [Simplify tests without implementation lock-in](#Simplify-tests-without-implementation-lock-in)
 18. [High-Level DOM Assertions for QUnit](#High-Level-DOM-Assertions-for-QUnit)
 19. [Type your code](#Type-your-code)
 20. [Time travelling](#Time-travelling)
 21. [Enforce locale](#Enforce-locale)
 22. [Accessibility / A11y](#Accessibility--A11y)

# Why I should test?
In the beginning, it may sound like a waste of time for people who are not get used to test the code, but in fact, it will pay off in the future when your code base grows. When your product will last for months or even years, it means that you have to do manual testing of your app every time when you introduce changes in the code. It is not a problem if you have some dedicated resources for that purpose, but at the end, a lot of part of work that is repeatable, instead of automated, is handled over and over by someone with a possibility of making a mistake (we are just humans).

Now imagine, that our team grows. A new team member who is not aware of every decision that other developers made joins the team. He can break already existed code base and there is a chance that someone can miss the breaking change and move it to the production code base if it is not well tested. Of course, tests are not the answer for every possible break in our application, however, can significantly reduce the changes that break the app over time. Also, they can bulletproof our business sensitive process - so we make sure that happy paths that are critical for our app are untouched. Moreover, an important benefit is that we save some time for manual testing because of our continuous integration can provide a feedback before anyone look at the code change.

<p align="right"><a href="#Table-of-Contents">back to top :arrow_up:</a></p>

* * *

# Testing good practices
1. Write test first, then implement the chunk of code that is tested. That will define component acceptance criteria. In this step, we should focus on the interface of our component (for instance). Your code is like a puzzle, that could be easily replaced or totally rewritten under the hood - the only part that should stay consisted is the public interface and behaviour.
2. Follow SOLID principles in your implementation (https://en.wikipedia.org/wiki/Single_responsibility_principle). One of the benefits: if we change the implementation of a small chunk of code, only tests that are related to this code can break - and we can easily fix all the problems instead of the looking at huge part of the code that can interact with our changes. It also reduces the changes in the code base - which simplifies tracking the changes for your teammates as well as it simplifies merge process in general. 
3. Keep the setup as tiny as possible and as explicit per each test as possible - avoid global before/after setup if it is not shared across all tests in the given file with tests. It also speeds up writing tests, because you don’t have to worry about “global” setup of your suite, instead of that you only focus on “local” setup. It usually means, that you also utilize your testing environment because you are not wasting resources when you don’t have to (for instance, global setup always sets a variable that is used in the scope of few tests instead of all tests).
4. Make sure you have tested all cases without false-positive effect (broken implementation can’t pass the tests - with false-positive it can).
5. Avoid using non-explicit dependencies, for instance - not publicly visible/defined dependencies of your code. Good example: our component shouldn’t rely on service objects that are globally injected in the application - they should be self-contained and independent, so we will avoid difficult debugging process and never forget about the proper setup of the application.

<p align="right"><a href="#Table-of-Contents">back to top :arrow_up:</a></p>

* * *

# How to run tests

Few useful commands that you should know:

```
// run the suite in the terminal
$ ember test

// run the suite in the terminal in watch mode - you can also inspect the app in that mode
$ ember test -s 

// filter the test suite and running only the tests that matches the filter
$ ember test -f 'some string' 
```

Also, by default test runner is available in `/tests` tab of your app: `http://localhost:4200/tests`

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
Let's prepare an automatic test of the creation of a new user. After user creation, app redirects to the listing on `/users` route. 

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

More info: https://guides.emberjs.com/v2.14.0/testing/acceptance/

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
We can stub methods, that are heavy, requires complex setup and usually are not the clue of our tests, but can improve readability of the test a lot.

```javascript
import sinon from 'sinon';

const store = this.owner.lookup('service:store');
sinon.stub(store, 'getUsersTeams').returns(store.peekAll('user'));
```

https://sinonjs.org/releases/v7.1.0/stubs/ 

<p align="right"><a href="#Table-of-Contents">back to top :arrow_up:</a></p>

* * *

# Mocks
Quite often in tests we need only simple objects rather than instances of full implementation of the given class. The good example: models where we are using mirage objects instead of Ember Data models, that differs with its inner implementation (which speeds up tests).

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

https://sinonjs.org/releases/v7.1.0/mocks/ 

<p align="right"><a href="#Table-of-Contents">back to top :arrow_up:</a></p>

* * *

# Spies
Spy allows you to check that our method was called with proper params. 

Example:

```javascript
let setValuesFunction = sinon.spy();
this.set('setValuesFunction', setValuesFunction);

await doSomething();

assert.equal(setValuesFunction.calledWith([1, 2, 3]), true);
```

https://sinonjs.org/releases/v7.1.0/spies/ 

<p align="right"><a href="#Table-of-Contents">back to top :arrow_up:</a></p>

* * *

# Lookup & Register

Let's imagine the scenario that we want to test the behaviour of our `users/` page that is accessible only for administrators. Our apps use `ember-simple-auth` for authentication as well as `ember-can` for authentication. We don't want to go through the whole login process in each acceptance test, because it slows down the test as well as it increases the risk of breaking many tests when the login page is broken only.

### What we can do about it?

Usually, our logic relies on the state of service objects. We need to make sure that we will stub a proper method or override a proper variable of the specific service object. How we can achieve that? We can use lookups. Lookup can find a reference of the specific factory for us, so in a test, we can do something like this:

```javascript
this.owner.lookup('service:store') // returns store
this.owner.lookup('component:my-component') // returns specific component
```

Ok, what if we don't have service in place in our application but it is still needed. We can register our own service easily. We can do the same for other types such as components, controllers, helpers etc. How? By using `.register` method:

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

It is very useful for instance if we stub some method, where we put assertion in it, to make sure that we called that method on an object or we need to do some extra check inside the stubbed method on the object that it provides (for instance when we stub action that should receive some complex object). Sometimes we can use spy instead in such scenarios (for simpler objects).

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

Allows to easily setup search phrase into ember power select input. When you see that some add-on misses test helper that could be useful, don't hesitate to contribute to it and add it on your own!

<p align="right"><a href="#Table-of-Contents">back to top :arrow_up:</a></p>

* * *

# Don’t overuse beforeEach/afterEach hooks

Those hooks should be fulfilled with setup only for the most shared part of the given test file. If you need to set up that is very specific to an only single test in your test file, you should always put it into the specific test, rather than in general setup that can lead to many problems and reduce the readability of the test. Also, it makes it harder to change tests, because you can break some tests when you modify those hooks.

<p align="right"><a href="#Table-of-Contents">back to top :arrow_up:</a></p>

* * *

# Blueprints
You can save your time with blueprints. Available blueprints are listed here:

https://github.com/emberjs/ember.js/tree/master/blueprints

Some useful one in terms of tests:

```
$ ember generate component-test component-name
$ ember generate route-test route-name
$ ember generate unit-test unit-tested-thing-name
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

What `ember-test-selectors` does, it allows us to add a proper data attribute within our templates which will work as an identifier in our tests. The cool thing about it is the fact, that we won't have to rely on id's or classes of our DOM elements that can change over time. Instead of that, we will use data attributes that will be enabled only in a testing environment (in the production build they are stripped) and even if we change the styling of our component it can still work without any extra changes in the test - which is the goal.

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

Especially useful part for us: https://github.com/simplabs/qunit-dom/blob/master/API.md

<p align="right"><a href="#Table-of-Contents">back to top :arrow_up:</a></p>

* * *

# Type your code

It may not be connected with testing at the first glance, but using type system for your code can be sort of a testing as well.

In Ember ecosystem, we have an add-on that adds easy support for **TypeScript**. We can write our code in **.ts** and then it will be automatically compiled into compatible js code with type check.

It may be very useful for unit-level tests, where part of the issues may be only connected with an incorrect type of the variable passed to the interface. Thanks to strong typing, you will get an feedback from your code before the runtime.

More info can be found in add-on repo: https://github.com/typed-ember/ember-cli-typescript

<p align="right"><a href="#Table-of-Contents">back to top :arrow_up:</a></p>

* * *

# Time travelling

From time to time, we need to ensure that our code will behave differently depending on the actuall date context. With that said, for instance, our component presents date in the custom date format and we want to make sure it will work well in any given month - we need to let our test runner know that we are time travel to the given date. How to do it? We can use a proper add-on that wraps Timecop library:

```
ember install ember-cli-timecop
```

More info how to use it you can find here: https://github.com/jamesarosen/Timecop.js

You may also consider alternative: https://github.com/Ticketfly/ember-mockdate-shim

<p align="right"><a href="#Table-of-Contents">back to top :arrow_up:</a></p>

* * *

# Enforce locale

Similar case like we have above with the time may happen in terms of locale of our app - to enforce the proper locale we can use **setupIntl** or **setLocale** from **Intl** add-on. 

More details with examples: https://github.com/ember-intl/ember-intl/blob/fde23d17ec7b3d7d8df602778a6c0c79c060e1fd/docs/testing.md

<p align="right"><a href="#Table-of-Contents">back to top :arrow_up:</a></p>

* * *

# Accessibility / A11y

Our apps should also support accessibility features so for instance, our application supports screen readers well. Luckily there are a ton of ready to use tools that we can use for that. Furthermore, there is a dedicated organization on Github that combines every well-documented a11y add-ons guidelines in a single place: https://github.com/ember-a11y.

Ok, but is there anything that we can do right now even without diving into all documentation details and get some meaningful feedback about the current state of accessibility in our application? Actually, yes.

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