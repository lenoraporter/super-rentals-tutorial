```run:server:start hidden=true cwd=super-rentals expect="Serving on http://localhost:4200/"
ember server
```

In this chapter, we will remove the hard-coded data from our `<Rental>` component. By the end, your app would finally be displaying real data that came from the server:

![The Super Rentals app by the end of the chapter](/screenshots/08-working-with-data/three-properties@2x.png)

In this chapter, you will learn about:

* Working with route files
* Returning local data from the model hook
* Accessing route models from templates
* Mocking server data with static JSON files
* Fetching remote data from the model hook
* Adapting server data
* Loops and local variables in templates with `{{#each}}`

## Working with Route Files

So far, we've been hard-coding everything into our `<Rental>` component. But that's probably not very sustainable, since eventually, we want our data to come from a server instead. Let's go ahead and move some of those hard-coded values out of the component in preparation for that.

We want to start working towards a place where we can eventually fetch data from the server, and then render the requested data as dynamic content from the templates. In order to do that, we will need a place where we can write the code for fetching data and loading it into the routes.

In Ember, *[route files][TODO: link to route files]* are the place to do that. We haven't needed them yet, because all our routes are essentially just rendering static pages up until this point, but we are about to change that.

Let's start by creating a route file for the index route. We will create a new file at `app/routes/index.js` with the following content:

```run:file:create lang=js cwd=super-rentals filename=app/routes/index.js
import Route from '@ember/routing/route';

export default class IndexRoute extends Route {
  async model() {
    return {
      title: 'Grand Old Mansion',
      owner: 'Veruca Salt',
      city: 'San Francisco',
      location: {
        lat: 37.7749,
        lng: -122.4194,
      },
      category: 'Estate',
      type: 'Standalone',
      bedrooms: 15,
      image: 'https://upload.wikimedia.org/wikipedia/commons/c/cb/Crane_estate_(5).jpg',
      description: 'This grand old mansion sits on over 100 acres of rolling hills and dense redwood forests.',
    };
  }
}
```

There's a lot happening here that we haven't seen before, so let's walk through this. First, we're importing the *[`Route` class][TODO: link to Route class]* into the file. This class is used as a starting point for adding functionality to a route, such as loading data.

Then, we are extending the `Route` class into our _own_ `IndexRoute`, which we also *[`export`][TODO: link to export]* so that the rest of the application can use it.

## Returning Local Data from the Model Hook

So far, so good. But what's happening inside of this route class? We implemented an *[async method][TODO: link to async method]* called `model()`. This method is also known as the *[model hook][TODO: link to model hook]*.

The model hook is responsible for fetching and preparing any data that you need for your route. Ember will automatically call this hook when entering a route, so that you can have an opportunity to run your own code to get the data you need. The object returned from this hook is known as the *[model][TODO: link to model]* for the route (surprise!).

Usually, this is where we'd fetch data from a server. Since fetching data is usually an asynchronous operation, the model hook is marked as `async`. This gives us the option of using the `await` keyword to wait for the data fetching operations to finish.

We'll get to that bit later on. At the moment, we are just returning the same hard-coding model data, extracted from the `<Rental>` component, but in a JavaScript object (also known as *[POJO][TODO: link to POJO]*) format.

## Accessing Route Models from Templates

So, now that we've prepared some model data for our route, let's use it in our template. In route templates, we can access the model for the route as `@model`. In our case, that would contain the POJO returned from our model hook.

To test that this is working, let's modify our template and try to render the `title` property from our model:

```run:file:patch lang=handlebars cwd=super-rentals filename=app/templates/index.hbs
@@ -6,2 +6,4 @@

+<h1>{{@model.title}}</h1>
+
 <div class="rentals">
```

If we look at our page in the browser, we should see our model data reflected as a new header.

```run:screenshot width=1024 height=512 retina=true filename=model-header.png alt="New header using the @model data"
visit http://localhost:4200/
wait  .rentals li:nth-of-type(3) article.rental
```

Awesome!

```run:command hidden=true cwd=super-rentals
yarn test
git add app/routes/index.js
git add app/templates/index.hbs
```

Okay, now that we know that we have a model to use at our disposal, let's remove some of the hard-coding that we did earlier! Instead of explicitly hard-coding the rental information into our `<Rental>` component, we can pass the model object to our component instead.

Let's try it out.

First, let's pass in our model to our `<Rental>` component as the `@rental` argument. We will also remove the extraneous `<h1>` tag we added earlier, now that we know things are working:

```run:file:patch lang=handlebars cwd=super-rentals filename=app/templates/index.hbs
@@ -6,9 +6,7 @@

-<h1>{{@model.title}}</h1>
-
 <div class="rentals">
   <ul class="results">
-    <li><Rental /></li>
-    <li><Rental /></li>
-    <li><Rental /></li>
+    <li><Rental @rental={{@model}} /></li>
+    <li><Rental @rental={{@model}} /></li>
+    <li><Rental @rental={{@model}} /></li>
   </ul>
```

By passing in `@model` into the `<Rental>` component as the `@rental` argument, we will have access to our "Grand Old Mansion" model object in the `<Rental>` component's template! Now, we can replace our hard-coded values in this component by using the values that live on our `@rental` model.

```run:file:patch lang=handlebars cwd=super-rentals filename=app/components/rental.hbs
@@ -2,18 +2,18 @@
   <Rental::Image
-    src="https://upload.wikimedia.org/wikipedia/commons/c/cb/Crane_estate_(5).jpg"
-    alt="A picture of Grand Old Mansion"
+    src={{@rental.image}}
+    alt="A picture of {{@rental.title}}"
   />
   <div class="details">
-    <h3>Grand Old Mansion</h3>
+    <h3>{{@rental.title}}</h3>
     <div class="detail owner">
-      <span>Owner:</span> Veruca Salt
+      <span>Owner:</span> {{@rental.owner}}
     </div>
     <div class="detail type">
-      <span>Type:</span> Standalone
+      <span>Type:</span> {{@rental.type}}
     </div>
     <div class="detail location">
-      <span>Location:</span> San Francisco
+      <span>Location:</span> {{@rental.city}}
     </div>
     <div class="detail bedrooms">
-      <span>Number of bedrooms:</span> 15
+      <span>Number of bedrooms:</span> {{@rental.bedrooms}}
     </div>
@@ -21,8 +21,8 @@
   <Map
-    @lat="37.7749"
-    @lng="-122.4194"
-    @zoom="9"
-    @width="150"
-    @height="150"
-    alt="A map of Grand Old Mansion"
+    @lat={{@rental.location.lat}}
+    @lng={{@rental.location.lng}}
+    @zoom="9"
+    @width="150"
+    @height="150"
+    alt="A map of {{@rental.title}}"
   />
```

Since the model object contains exactly the same data as the previously-hard-coded "Grand Old Mansion", the page should look exactly the same as before after the change.

```run:screenshot width=1024 height=512 retina=true filename=using-model-data.png alt="New header using the @model data"
visit http://localhost:4200/
wait  .rentals li:nth-of-type(3) article.rental
```

Now, we have one last thing to do: update the tests to reflect this change.

Because component tests are meant to render and test a single component in isolation from the rest of the app, they do not perform any routing, which means we won't have access to the same data returned from the `model` hook.

Therefore, in our `<Rental>` component's test, we will have to feed the data into it some other way. We can do this using the `setProperties` we learned about from the [previous chapter](../07-reusable-components/).

```run:file:patch lang=js cwd=super-rentals filename=tests/integration/components/rental-test.js
@@ -9,3 +9,20 @@
   test('it renders information about a rental property', async function(assert) {
-    await render(hbs`<Rental />`);
+    this.setProperties({
+      rental: {
+        title: 'Grand Old Mansion',
+        owner: 'Veruca Salt',
+        city: 'San Francisco',
+        location: {
+          lat: 37.7749,
+          lng: -122.4194,
+        },
+        category: 'Estate',
+        type: 'Standalone',
+        bedrooms: 15,
+        image: 'https://upload.wikimedia.org/wikipedia/commons/c/cb/Crane_estate_(5).jpg',
+        description: 'This grand old mansion sits on over 100 acres of rolling hills and dense redwood forests.',
+      }
+    });
+
+    await render(hbs`<Rental @rental={{this.rental}} />`);

```

Notice that we also need to update the invocation of the `<Rental>` component in the `render` function call to also have a `@rental` argument passed into it. If we run our tests now, they should all pass!

```run:command hidden=true cwd=super-rentals
yarn test
git add app/components/rental.hbs
git add app/templates/index.hbs
git add tests/integration/components/rental-test.js
```

```run:screenshot width=1024 height=768 retina=true filename=pass.png alt="All our tests are passing"
visit http://localhost:4200/tests?nocontainer&deterministic
wait  #qunit-banner.qunit-pass
```

## Mocking Server Data with Static JSON Files

Now that we have things in place, let's do the fun part of removing *all* our hard-coded values from the model hook and actually fetch some data from the server!

In a production app, the data that we'd fetch would most likely come from a remote API server. To avoid setting up an API server just for this tutorial, we will put some JSON data into the `public` folder instead. That way, we can still request this JSON data with regular HTTP requests &mdash; just like we would with a real API server  &mdash; but without having to write any server logic.

But where will the data come from? You can <a href="/downloads/data.zip" download="data.zip">download this data file</a>, where we have prepared some JSON data and bundled it into a `.zip` file format. Extract its content into the `public` folder.

When you are done, your `public` folder should now have the following content:

```run:file:copy hidden=true src=downloads/data cwd=super-rentals filename=public
```

```run:command lang=plain cwd=super-rentals captureCommand=false
# Tree uses UTF-8 "non-breaking space" which gets turned into &nbsp;
# Somewhere in the guides repo's markdown pipeline there is a bug
# that further turns them into &amp;nbsp;

#[cfg(unix)]
tree public --dirsfirst | sed 's/\xC2\xA0/ /g'

#[cfg(windows)]
tree public /F
```

```run:command hidden=true cwd=super-rentals
git add public/api
```

You can verify that everything is working correctly by navigating to `http://localhost:4200/api/rentals.json`.

```run:screenshot width=1024 height=512 retina=true filename=data.png alt="Our server serving up our rental properties as JSON data"
visit http://localhost:4200/api/rentals.json
```

Awesome! Our "server" is now up and running, serving up our rental properties as JSON data.

## Fetching Remote Data from the Model Hook

Now, let's turn our attention to our model hook again. We need to change it so that we actually fetch the data from the server.

```run:file:patch lang=js cwd=super-rentals filename=app/routes/index.js
@@ -4,16 +4,5 @@
   async model() {
-    return {
-      title: 'Grand Old Mansion',
-      owner: 'Veruca Salt',
-      city: 'San Francisco',
-      location: {
-        lat: 37.7749,
-        lng: -122.4194,
-      },
-      category: 'Estate',
-      type: 'Standalone',
-      bedrooms: 15,
-      image: 'https://upload.wikimedia.org/wikipedia/commons/c/cb/Crane_estate_(5).jpg',
-      description: 'This grand old mansion sits on over 100 acres of rolling hills and dense redwood forests.',
-    };
+    let response = await fetch('/api/rentals.json');
+    let parsed = await response.json();
+    return parsed;
   }
```

What's happening here?

First off, we're using the browser's *[Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)* to request that JSON data from our server's API at `public/api/rentals.json`, the same URL we visited earlier.

As mentioned above, fetching data from the server is usually an asynchronous operation. The Fetch API takes this into account, which is why `fetch` is an `async` function, just like our model hook. To consume its response, we will have to pair it with the `await` keyword.

The Fetch API returns a *[response object][TODO: link to response object]* asynchronously. Once we have this object, we can convert the server's response into whatever format we need; in our case, we knew the server sent the data using the JSON format, so we can use the `json()` method to *[parse][TODO: link to parse]* the response data accordingly. Parsing the response data is _also_ an asynchronous operation, so we'll just use the `await` keyword here, too.

```run:command hidden=true cwd=super-rentals
git add app/routes/index.js
```

## Adapting Server Data

Before we go any further, let's pause for a second to look at the our server's data again.

```run:file:show lang=json cwd=super-rentals filename=public/api/rentals.json
```

This data follows the *[JSON:API][TODO: link to JSON:API]* format, which is _slightly_ different than the hard-coded data that we were returning from the model hook before.

First off, the JSON:API format returns an array nested under the `"data"` key, rather than a just the data for a single rental property. If we think about this, though, it makes sense; we now want to show a whole list of rental properties that are coming from our server, not just one, so an array of rental property objects is just what we need.

The rental property objects contained in the array also have a slightly different structure. Every data object has a `type` and `id`, which we don't intend to use in our template (yet!). For now, the only data we really need is nested within the `attributes` key.

There's one more key difference here, which perhaps only those with very sharp eyes will be able to catch: the data coming from the server is missing the `type` property, which previously existed on our hard-coded model object. The `type` property could either be `"Standalone"` or `"Community"`, depending on the type of rental property, which is required by our `<Rental>` component.

In [Part 2](../10-part-2/) of this tutorial, we will learn about a much more convenient to consume data in the JSON:API format. For now, we can just fix up the data and deal with these differences in formats ourselves.

We can handle it all in our model hook:

```run:file:patch lang=js cwd=super-rentals filename=app/routes/index.js
@@ -2,7 +2,25 @@

+const COMMUNITY_CATEGORIES = [
+  'Condo',
+  'Townhouse',
+  'Apartment'
+];
+
 export default class IndexRoute extends Route {
   async model() {
     let response = await fetch('/api/rentals.json');
-    let parsed = await response.json();
-    return parsed;
+    let { data } = await response.json();
+
+    return data.map(model => {
+      let { attributes } = model;
+      let type;
+
+      if (COMMUNITY_CATEGORIES.includes(attributes.category)) {
+        type = 'Community';
+      } else {
+        type = 'Standalone';
+      }
+
+      return { type, ...attributes };
+    });
   }
```

After parsing the JSON data, we extracted the nested `attributes` object, added back the missing `type` attribute manually, then returned it from the model hook. That way, the rest of our app will have no idea that this difference ever existed.

Awesome! Now we're in business.

## Loops and Local Variables in Templates with `{{#each}}`

The last change we'll need to make is to our `index.hbs` route template, where we invoke our `<Rental>` components. Previously, we were passing in `@rental` as `@model` to our components. However, `@model` is no longer a single object, but rather, an array! So, we'll need to change this template to account for that.

Let's see how.

```run:file:patch lang=handlebars cwd=super-rentals filename=app/templates/index.hbs
@@ -8,5 +8,5 @@
   <ul class="results">
-    <li><Rental @rental={{@model}} /></li>
-    <li><Rental @rental={{@model}} /></li>
-    <li><Rental @rental={{@model}} /></li>
+    {{#each @model as |rental|}}
+      <li><Rental @rental={{rental}} /></li>
+    {{/each}}
   </ul>
```

We can use the `{{#each}}...{{/each}}` syntax to iterate and loop through the array returned by the model hook. For each iteration through the array &mdash; for each item in the array &mdash; we will render the block that is passed to it once. In our case, the block is our `<Rental>` component, surrounded by `<li>` tags.

Inside of the block we have access to the item of the _current_ iteration with the `{{rental}}` variable. But why `rental`? Well, because we named it that! This variable comes from the `as |rental|` declaration of the `each` loop. We could have just as easily called it something else, like `as |property|`, in which case we would have to access the current item through the `{{property}}` variable.

Now, let's go over to our browser and see what our index route looks like with this change.

```run:screenshot width=1024 retina=true filename=three-properties.png alt="Three different rental properties"
visit http://localhost:4200/
wait  .rentals li:nth-of-type(3) article.rental
```

Hooray! Finally we're seeing different rental properties in our list. And we got to play with `fetch` and write a loop. Pretty productive, if you ask me.

Better yet, all of our tests are still passing too!

```run:command hidden=true cwd=super-rentals
yarn test
git add app/routes/index.js
git add app/templates/index.hbs
```

```run:screenshot width=1024 height=768 retina=true filename=pass-2.png alt="All our tests are passing"
visit http://localhost:4200/tests?nocontainer&deterministic
wait  #qunit-banner.qunit-pass
```

```run:server:stop
ember server
```

```run:checkpoint cwd=super-rentals
Chapter 8
```
