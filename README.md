# New and Create Routes

## Lesson Objectives

1. Create a new route and page
1. Add interactivity to your site with forms
1. Create a post route
1. Define middleware
1. View body of a post request
1. Redirect the user to another page

## Setup

1. `cd ~/sei/express-fruits`
2. Open `express-fruits` in your editor.
3. `nodemon`

## Create a new route and page

1. Let's create a page with a form that will allow us to create a new fruit.
2. First, we'll need a route for displaying the page in our server.js file **IMPORTANT: put this above your show route, so that the show route doesn't accidentally pick up a /fruits/new request**

    ```js
    app.get('/fruits/newForm', (req, res) => {
        res.render('new.ejs');
    });
    ```

3. Now let's a file for our new form: `touch views/new.ejs`

4. In `views/new.ejs`, insert this HTML:

    ```html
    <!DOCTYPE html>
    <html>
        <head>
            <meta charset="utf-8">
            <title></title>
        </head>
        <body>
            <h1>New Fruit page</h1>
        </body>
    </html>
    ```

Visit http://localhost:3000/fruits/newForm to see if it works.

## Add interactivity to your site with forms

We can use forms to allow the user to enter their own data:

```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <title></title>
    </head>
    <body>
        <h1>New Fruit page</h1>
        <!--  NOTE: action will be the route, method will be the HTTP verb-->
        <form action="/fruits" method="POST">
            Name: <input type="text" name="name" /><br/>
            Color: <input type="text" name="color" /><br/>
            Is Ready To Eat: <input type="checkbox" name="readyToEat" /><br/>
            <input type="submit" value="Create Fruit"/>
        </form>
    </body>
</html>
```

**NOTE: the form element has an action and a method attribute. We get these values from our RESTful routes table. We'll need this info for the next step too.**

## Create a post route

Since the form in the last step tells the browser to create a POST request to /fruits, we'll need to set up a route handler for this kind of request

```js
app.post('/fruits', (req, res) => {
    res.send('hi');
});
```

## Define middleware

We can have a function execute for all routes. In `server.js`, above our routes:

```js
app.use((req, res, next) => {
    console.log('I run for all routes');
    next();
});
```

This is called "middleware." It runs in the middle of the request-response cycle; that is, it runs sometime after the request is received, but before the final route handler is called.

To register middleware, we use the `.use()` Express method. It typically takes at least at least one callback, and sometimes takes a URL path as an option (we'll see this when we move our controllers). The `next()` function with no arguments allows the middleware to act like it didn't handle the route so that the next, natural route can handle it instead.

Be sure to put middleware toward the top of your `server.js` file above your routes, so that none of the routes can handle the request and send the response before the middleware has a chance to be executed.

Most of the time, you won't write your own middleware, but a lot of plugins and extended functionality of express exist as middleware.

But there's at least one piece of middleware we'll need for most of our Express apps.

But, first, let's modify the test middleware we just wrote to include some useful information:

```js
app.use((req, res, next) => {
    console.log(`${req.method} ${req.originalUrl}`);
    next();
});
```

This will print out the HTTP method and the URL path for every request to our terminal, which can be very helpful for testing.

## View body of a post request

When we submit our new fruit form, it sends a POST request to our server, and that request has data in it (name, color, readyToEat, etc).

Just like URL params are added to `req.params` and query params are added to `req.query`, data that's sent from a form will be added to `req.body`.

The way this works is that, when the data is collected from the form, the values from each `input` are collected into key-value pairs. The keys are from the `name` attribute of each `input`, and the values are what the user enters into those `input` fields.

But, in order to parse the data from the request and add it to `req.body`, we'll need some middleware called `body-parser`.

As of Express 4.16, `body-parser` was included in the `express` library, so we don't have to install it separately because we already did!

We just need to register it in our app.

### Registering the middleware

In `server.js`:

```js
// this should be near the top, above the routes
app.use(express.urlencoded({ extended: false }));
```

The `express.urlencoded` method checks to see if the request's headers include a Content-Type of application/x-www-form-urlencoded. If it comes from a form submission (POST and PUT), then it will be. Thus, that request content will need to be parsed and added as an object to the request object, specifically in the body (i.e., `req.body`). Setting `extended` to `true` means that we can send nested objects as part of our data object if we'd like. Since we don't need this, we'll set it to `false`.

### Setting up the route

Inside the create route, we can do the following:

```js
app.post('/fruits', (req, res) => {
    console.log(req.body);
    res.send('data received');
});
```

Note that, when we check the "Is Ready To Eat:" checkbox, `readyToEat` is added to the `req.body` object with a value of `"on"`. But, when it's not checked, `readyToEat` isn't added to `req.body` at all.

So far, we're just printing out the data from the form, but what we really want is to push the new data into our `fruits` array.

```js
// create route
// this route will catch POST requests to /fruits
// and, after creating new data, respond by redirecting
// the user to the index route
app.post('/fruits', (req, res) => {
    if(req.body.readyToEat === 'on'){
        req.body.readyToEat = true;
    } else {
        req.body.readyToEat = false;
    }
    fruits.push(req.body);
    console.log(fruits);
    res.send('data received');
});
```

Because the `readyToEat` property is a boolean for all our other fruits, we need to edit the `req.body` object before inserting it into our data (i.e., the `fruits` array).


## Redirect the user to another page

The data has been added to our fruits array, so where should the user go next?

In most apps, after we click Submit and create something new, we're either redirected to a page that shows only that newly-created item (i.e., a show route), or we're redirected to a page that shows all our items, which includes that newly-created item (i.e., an index route).

Let's send the user back to the fruits index page upon completion so they can see all the fruits, including their new fruit.

Fortunately, we already have a route that lists all the fruits, so why re-invent the route? Instead, we can just redirect them to another route using the `res.redirect()` method. All we need to pass to this method is the URL path. This would be the same path that we typed into our browser's URL field to hit the index route in the first place.

```js
app.post('/fruits', (req, res)=>{
    if(req.body.readyToEat === 'on'){
        req.body.readyToEat = true;
    } else {
        req.body.readyToEat = false;
    }
    fruits.push(req.body);
     // redirect the user to the index route
     // since the index route is listening for GET requests
     // with a URL path of '/fruits', we just need to include
     // the URL path as the argument since the .redirect() method
     // has a default HTTP verb of GET.
    res.redirect('/fruits');
});
```


## Create an index page

We don't have a view to display all fruits yet, so let's refactor the index route and create an index view.

First, let's refactor the index route:

```js
app.get('/fruits', (req, res) => {
    res.render('index.ejs', {
        allFruits: fruits
    })
});
```

Remember that the key (`allFruits`) is what is available to the view as a reference to `fruits`.

We'll need the view: `touch views/index.ejs`.

Inside `views/index.ejs`:

```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <title></title>
    </head>
    <body>
        <h1>Fruits index page</h1>
        <ul>
            <% for(let i = 0; i < allFruits.length; i++){ %>
                <li>
                    <%= allFruits[i].name %>
                </li>
            <% } %>
        </ul>
        <nav>
            <a href="/fruits/newForm">Create a New Fruit</a>
        </nav>
    </body>
</html>
```

In our show route, the `oneFruit` was an object, which meant that we could immediately reference its properties (e.g., `oneFruit.name`). By contrast, `allFruits` is an array, so we use a loop to iterate through all the objects, putting each fruit's name on the page.

But, when we see a list like this, it's typical to be able to click on a fruit and be taken to a show page.

Remember that our show route URL is `/fruits/:fruitIndex`. Here in the view, the `:fruitIndex` part of the URL needs to be a real value like 0, 1, 2, etc. Fortunately, we already have a variable that holds those values, namely, `i` in the loop.

With those pieces in place, let's wrap each fruit name in a link:

```html
<a href="/fruits/<%= i %>">
    <%= allFruits[i].name %>
</a>
```

Remember that the `<%= %>` syntax doesn't mean that, whatever is between those tags can be seen on the page by the user, but that the value becomes a part of the HTML.

Let's go to http://localhost:3000/fruits and inspect our page in the Elements tab to see what those dynamically-created links look like.
