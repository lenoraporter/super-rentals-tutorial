```run:server:start hidden=true cwd=super-rentals expect="Serving on http://localhost:4200/"
ember server
```

In this chapter, you will add interactivity to the page, allowing the user to click an image to enlarge or shrink it:

<!-- TODO: make this a gif instead -->

![The Super Rentals app by the end of the chapter (default image size)](/screenshots/06-interactive-components/rental-image-default@2x.png)

![The Super Rentals app by the end of the chapter (large image size)](/screenshots/06-interactive-components/rental-image-large@2x.png)

While doing so, you will learn about:

* Adding behavior to components with classes
* Accessing instance states from templates
* Managing state with tracked properties
* Using conditionals syntaxes in templates
* Responding to user interaction with actions
* Invoking element modifiers
* Testing user interactions

## Adding Behavior to Components with Classes

So far, all the components we have written are purely *[presentational][TODO: link to presentational]* &mdash; they are simply reusable snippets of markup. That's pretty cool! But in Ember, components can do so much more.

Sometimes, you want to associate some *[behavior][TODO: link to behavior]* with your components so that they can do more interesting things. For example, `<LinkTo>` can respond to clicks by changing the URL and navigating us to a different page.

Here, we are going to do just that! We are going to implement the "View Larger" and "View Smaller" functionality, which will allow our users to click on a property's image to view a larger version, and click on it again to return to the smaller version.

In other words, we want a way to _toggle_ the image between one of the two *[states][TODO: link to states]*. In order to do that, we need a way for the component to store two possible states, and to be aware of which state it is currently in.

Ember optionally allows us to associate JavaScript code with a component for exactly this purpose. We can add a JavaScript file for our `<Rental::Image>` component by running the `component-class` generator:

```run:command cwd=super-rentals
ember generate component-class rental/image
```

```run:command hidden=true cwd=super-rentals
yarn test
git add app/components/rental/image.js
```

This generated a JavaScript file with the same name as our component's template at `app/components/rentals/image.js`. It contains a *[JavaScript class][TODO: link to JavaScript class]*, *[inheriting][TODO: link to inheriting]* from `@glimmer/component`.

> Zoey says...
>
> `@glimmer/component`, or *[Glimmer component][TODO: link to Glimmer component]*, is one of the several component classes available to use. They are a great starting point whenever you want to add behavior to your components. In this tutorial, we will be using Glimmer components exclusively.
>
> In general, Glimmer components should be used whenever possible. However, you may also see `@ember/components`, or *[classic components][TODO: link to classic components]*, used in older apps. You can tell them apart by looking at their import path (which is helpful for looking up the respective documentation, as they have different and incompatible APIs).

Ember will create an *[instance][TODO: link to instance]* of the class whenever our component is invoked. We can use that instance to store our state:

```run:file:patch lang=js cwd=super-rentals filename=app/components/rental/image.js
@@ -3,2 +3,6 @@
 export default class RentalImageComponent extends Component {
+  constructor(...args) {
+    super(...args);
+    this.isLarge = false;
+  }
 }
```

Here, in the *[component's constructor][TODO: link to component's constructor]*, we *[initialized][TODO: link to initialized]* the *[instance variable][TODO: link to instance variable]* `this.isLarge` with the value `false`, since this is the default state that we want for our component.

## Accessing Instance States from Templates

Let's update our template to use this state we just added:

```run:file:patch lang=handlebars cwd=super-rentals filename=app/components/rental/image.hbs
@@ -1,3 +1,11 @@
-<div class="image">
-  <img ...attributes>
-</div>
+{{#if this.isLarge}}
+  <div class="image large">
+    <img ...attributes>
+    <small>View Smaller</small>
+  </div>
+{{else}}
+  <div class="image">
+    <img ...attributes>
+    <small>View Larger</small>
+  </div>
+{{/if}}
```

In the template, we have access to the component's instance variables. The `{{#if ...}}...{{else}}...{{/if}}` *[conditionals][TODO: link to conditionals]* syntax allows us to render different content based on a condition (in this case, the value of the instance variable `this.isLarge`). Combining these two features, we can render either the small or the large version of the image accordingly.

```run:command hidden=true cwd=super-rentals
yarn test
git add app/components/rental/image.hbs
git add app/components/rental/image.js
```

We can verify this works by temporarily changing the initial value in our JavaScript file. If we change `app/components/rental/image.js` to initialize `this.isLarge = true;` in the constructor, we should see the large version of the property image in the browser. Cool!

```run:file:patch hidden=true cwd=super-rentals filename=app/components/rental/image.js
@@ -5,3 +5,3 @@
     super(...args);
-    this.isLarge = false;
+    this.isLarge = true;
   }
```

```run:screenshot width=1024 height=1500 retina=true filename=is-large-true.png alt="<Rental::Image> with this.isLarge set to true"
visit http://localhost:4200/
wait  .rentals li:nth-of-type(3) article.rental .image.large img
```

Once we've tested this out, we can change `this.isLarge` back to `false`.

```run:command hidden=true cwd=super-rentals
git checkout app/components/rental/image.js
```

Since this pattern of initializing instance variables in the constructor is pretty common, there happens to be a much more concise syntax for it:

```run:file:patch lang=js cwd=super-rentals filename=app/components/rental/image.js
@@ -3,6 +3,3 @@
 export default class RentalImageComponent extends Component {
-  constructor(...args) {
-    super(...args);
-    this.isLarge = false;
-  }
+  isLarge = false;
 }
```

This does exactly the same thing as before, but it's much shorter and less to type!

```run:command hidden=true cwd=super-rentals
yarn test
git add app/components/rental/image.js
```

Of course, our users cannot edit our source code, so we need a way for them to toggle the image size from the browser. Specifically, we want to toggle the value of `this.isLarge` whenever the user clicks on our component.

## Managing State with Tracked Properties

Let's modify our class to add a *[method][TODO: link to method]* for toggling the size:

```run:file:patch lang=js cwd=super-rentals filename=app/components/rental/image.js
@@ -1,5 +1,11 @@
 import Component from '@glimmer/component';
+import { tracked } from '@glimmer/tracking';
+import { action } from '@ember/object';

 export default class RentalImageComponent extends Component {
-  isLarge = false;
+  @tracked isLarge = false;
+
+  @action toggleSize() {
+    this.isLarge = !this.isLarge;
+  }
 }
```

We did a few things here, so let's break it down.

First, we added the `@tracked` *[decorator][TODO: link to decorator]* to the `isLarge` instance variable. This annotation tells Ember to monitor this variable for updates. Whenever this variable's value changes, Ember will automatically re-render any templates that depend on its value.

In our case, whenever we assign a new value to `this.isLarge`, the `@tracked` annotation will cause Ember to re-evaluate the `{{#if this.isLarge}}` conditional in our template, and will switch between the two *[blocks][TODO: link to blocks]* accordingly.

> Zoey says...
>
> Don't worry! If you reference a variable in the template but forget to add the `@tracked` decorator, you will get a helpful development mode error when you change its value!

## Responding to User Interaction with Actions

Next, we added a `toggleSize` method to our class that switches `this.isLarge` to the opposite of its current state (`false` becomes `true`, or `true` becomes `false`).

Finally, we added the `@action` decorator to our method. This indicates to Ember that we intend to use this method from our template. Without this, the method will not function properly as a callback function (in this case, a click handler).

> Zoey says...
>
> If you forget to do add the `@action` decorator, you will also get a helpful error when clicking on the button in development mode!

With that, it's time to wire this up in the template:

```run:file:patch lang=handlebars cwd=super-rentals filename=app/components/rental/image.hbs
@@ -1,11 +1,11 @@
 {{#if this.isLarge}}
-  <div class="image large">
+  <button type="button" class="image large" {{on "click" this.toggleSize}}>
     <img ...attributes>
     <small>View Smaller</small>
-  </div>
+  </button>
 {{else}}
-  <div class="image">
+  <button type="button" class="image" {{on "click" this.toggleSize}}>
     <img ...attributes>
     <small>View Larger</small>
-  </div>
+  </button>
 {{/if}}
```

We changed two things here.

First, since we wanted to make our component interactive, we switched the containing tag from `<div>` to `<button>` (this is important for accessibility reasons). By using the correct semantic tag, we will also get focusability and keyboard interaction handling "for free".

Next, we used the `{{on}}` *[modifier][TODO: link to modifier]* to attach `this.toggleSize` as a click handler on the button.

With that, we have created our first *[interactive][TODO: link to interactive]* component. Go ahead and try it in the browser!

<!-- TODO: make this a gif instead -->

```run:screenshot width=1024 retina=true filename=rental-image-default.png alt="<Rental::Image> (default size)"
visit http://localhost:4200/
wait  .rentals li:nth-of-type(3) article.rental .image img
```

```run:screenshot width=1024 height=1500 retina=true filename=rental-image-large.png alt="<Rental::Image> (large size)"
visit http://localhost:4200/
wait  .rentals li:nth-of-type(3) article.rental .image img
click .rentals li:first-of-type article.rental .image
wait  .rentals li:first-of-type article.rental .image.large img
```

```run:command hidden=true cwd=super-rentals
yarn test
git add app/components/rental/image.hbs
git add app/components/rental/image.js
```

## Testing User Interactions

Finally, let's write a test for this new behavior:

```run:file:patch lang=js cwd=super-rentals filename=tests/integration/components/rental/image-test.js
@@ -2,3 +2,3 @@
 import { setupRenderingTest } from 'ember-qunit';
-import { render } from '@ember/test-helpers';
+import { render, click } from '@ember/test-helpers';
 import { hbs } from 'ember-cli-htmlbars';
@@ -20,2 +21,26 @@
   });
+
+  test('clicking on the component toggles its size', async function(assert) {
+    await render(hbs`
+      <Rental::Image
+        src="/assets/teaching-tomster.png"
+        alt="Teaching Tomster"
+      />
+    `);
+
+    assert.dom('button.image').exists();
+
+    assert.dom('.image').doesNotHaveClass('large');
+    assert.dom('.image small').hasText('View Larger');
+
+    await click('button.image');
+
+    assert.dom('.image').hasClass('large')
+    assert.dom('.image small').hasText('View Smaller');
+
+    await click('button.image');
+
+    assert.dom('.image').doesNotHaveClass('large');
+    assert.dom('.image small').hasText('View Larger');
+  });
 });
```

```run:command hidden=true cwd=super-rentals
yarn test
git add tests/integration/components/rental/image-test.js
```

```run:screenshot width=1024 height=512 retina=true filename=pass.png alt="Tests passing with the new <Rental::Image> test"
visit http://localhost:4200/tests?nocontainer&deterministic
wait  #qunit-banner.qunit-pass
```

Let's clean up our template before moving on. We introduced a lot of duplication when we added the conditional in the template. If we look closely, the only things that are different between the two blocks are:

1. The presence of the `"large"` CSS class on the `<button>` tag.
2. The "View Larger" and "View Smaller" text.

These changes are buried deep within the large amount of duplicated code. We can reduce the duplication by using an `{{if}}` *[expression][TODO: link to expression]* instead:

```run:file:patch lang=handlebars cwd=super-rentals filename=app/components/rental/image.hbs
@@ -1,11 +1,8 @@
-{{#if this.isLarge}}
-  <button type="button" class="image large" {{on "click" this.toggleSize}}>
-    <img ...attributes>
+<button type="button" class="image {{if this.isLarge "large"}}" {{on "click" this.toggleSize}}>
+  <img ...attributes>
+  {{#if this.isLarge}}
     <small>View Smaller</small>
-  </button>
-{{else}}
-  <button type="button" class="image" {{on "click" this.toggleSize}}>
-    <img ...attributes>
+  {{else}}
     <small>View Larger</small>
-  </button>
-{{/if}}
+  {{/if}}
+</button>
```

The expression version of `{{if}}` takes two arguments. The first argument is the *[condition][TODO: link to condition]*. The second argument is the expression that should be evaluated if the condition is true.

```run:command hidden=true cwd=super-rentals
yarn test
git add app/components/rental/image.hbs
```

Optionally, `{{if}}` can take a third argument for what the expression should evaluate into if the condition is false. This means we could rewrite the button label like so:

```run:file:patch lang=handlebars cwd=super-rentals filename=app/components/rental/image.hbs
@@ -2,7 +2,3 @@
   <img ...attributes>
-  {{#if this.isLarge}}
-    <small>View Smaller</small>
-  {{else}}
-    <small>View Larger</small>
-  {{/if}}
+  <small>View {{if this.isLarge "Smaller" "Larger"}}</small>
 </button>
```

Whether or not this is an improvement in the clarity of our code is mostly a matter of taste. Either way, we have significantly reduced the duplication in our code, and made the important bits of logic stand out from the rest.

Run the test suite one last time to confirm our refactor didn't break anything unexpectedly, and we will be ready for the next challenge!

```run:command hidden=true cwd=super-rentals
yarn test
git add app/components/rental/image.hbs
```

```run:screenshot width=1024 height=512 retina=true filename=pass-2.png alt="Tests still passing after the refactor"
visit http://localhost:4200/tests?nocontainer&deterministic
wait  #qunit-banner.qunit-pass
```

```run:server:stop
ember server
```

```run:checkpoint cwd=super-rentals
Chapter 6
```
