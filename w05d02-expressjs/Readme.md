# Day's Title 

## Learning Competencies
- Understand what `express` is and how to create a simple application
- Define what server side templating is
- Render templates using `pug`
- Use `pug` to create dynamic templates

- Define what CRUD means
- Define what REST and RESTful routing are
- Override the default method to allow for PATCH and DELETE requests
- Build a CRUD app using RESTful routing

- Define what middleware is
- Write custom middleware to handle errors
- Install and use helpful middleware including morgan
- Create express applications faster using the express-generator

- To understand how write forms to get data from users, and update the database with this data.

- Move routing logic into a separate file using the express router
- Use the express router to refactor routing code


## Overview

### ExpressJS
We previously saw the core HTTP module which allows us to make requests and issue responses, but we also mentioned that using the HTTP module requires a fair amount code and complexity. Thankfully `express` abstracts away much of this complexity. So let's get started with it! In the terminal, let's type the following:
```
# create a folder and cd into it
mkdir first_express_app && cd first_express_app
# create a file called app.js (doing this before npm init is important, we will see why later!)
touch app.js
# create a package json file
npm init -y
# install the express module and save the module name and version to our package.json so that when we work with other developers they can easily install all of our dependencies
npm install --save express
```
Inside of our `app.js` let's add the following:
```js
// require the express module
var express = require("express");
// create an object from the express function which we contains methods for making requests and starting the server
var app = express();

// create a route for a GET request to '/' - when that route is reached, run a function
app.get('/', function(request, response){
    // inside of this callback we have two large objects, request and response
        // request - can contain data about the request (query string, url parameters, form data)
        // response - contains useful methods for determining how to respond (with html, text, json, etc.)
    // let's respond by sending the text Hello World!
    response.send('Hello World!');
})

// let's tell our server to listen on port 3000 and when the server starts, run a callback function that console.log's a message
app.listen(3000, function(){
    console.log("The server has started on port 3000. Head to localhost:3000 in the browser and see what's there!");
});
```
Now let's start the server using node `app.js` and head over to `localhost:3000`. To stop the server press `control + c`. Remember that when the server is running, you will not be able to type commands in that tab in the terminal. If you want to run other command line commands, open a different tab.

#### `nodemon`
Nodemon is very useful for restarting the server when it goes down so that you don't have to manually restart it yourself. To install `nodemon` type the following command anywhere in the terminal `npm install -g nodemon` if you are getting errors when installing this, you can use `sudo npm install -g nodemon` and type in your password and press enter to install it. (If you need to use `sudo` you've got a permissions issue with `npm`; follow [these](https://docs.npmjs.com/getting-started/fixing-npm-permissions) instructions to fix the problem.) To start the server now you can use `nodemon app.js`.

#### url parameters
So far we have just seen how to build static routes, but what makes server-side programming so powerful is that we can create dynamic responses based on the type of request. This is how applications can have one single route that provides many different kinds of responses depending on the data that is passed in the URL. Dynamic values to be passed in the URL are called "URL parameters" and are stored in the `params` object which exists in `request` object given to us in our callback functions.

To specify that a part of a URL will be a "parameter", we add the `:` character and then give our URL parameter a name. This name will be the key in the `params` object. Let's see what that looks like below. It is also important to note that all of our URL parameters are strings! Let's take a look at an `app.js` file that has a route with URL parameters (notice the `:`)
```js
// same pattern as above, require express, invoke the express function
var express = require("express");
var app = express();

// same as above, listen for a GET request
app.get('/', function(request, response){
    response.send('Hello World!')
})

// when a request comes in to /instructors/ANYTHING
app.get('/instructor/:first_name', function(request,response){
    // let's capture the "dynamic" part of the URL, which we are called "first_name". The name that we give to this dynamic part of the URL will become a key in the params object which exists on the request object.

    // let's send back some text with whatever data came in the URL!
    response.send(`The name of this instructor is ${request.params.first_name}`)
});

app.listen(3000, function(){
    console.log("The server has started on port 3000. Head to localhost:3000 in the browser and see what's there!")
})
```
To run this application we can either type `node app.js`, but if we need to make any changes to the file we would need to restart the server to see this change. Thankfully we have `nodemon` to help us manage starting and restarting the server when a change happens, so we can type `nodemon app.js` or just `nodemon` and our server will start!

Now if we head over to `localhost:3000/instructor/elie` and then `localhost:3000/instructor/matt` and then `localhost:3000/instructor/tim`, what do you notice? Even though we created one single route, we can now issue different responses depending on what the user has typed in the browser! This is the foundation for how to build dynamic applications (the url parameter could be a value we look up in a database to display different profiles for the same route). It is also important to note that **ALL** URL parameters are strings, so if we try to work with anything inside of `request.params`, it will always be a `string`.

#### render
So far we have only seen how to respond by sending text, but when building server-side applications, we commonly want to send back HTML templates with data created on the server. If we want to simply send HTML back we can use a method on the `response` object called `sendFile`, but since we are using server-side templates, we will be using a method called `render`.

In order to do that, we need to make use of a templating engine (a tool for creating HTML that includes server side data). There are quite a few we can use with Node.js including `ejs`, but we will be using one called `pug` (formerly known as `jade`). Before we introduce the pug syntax, let's first start a new project and see what we need to include. By default, express expects a folder containing templates to be called `views`. Instead of overriding that default, we will also create a folder called `views`.
```
mkdir learn_pug && cd learn_pug
touch app.js
npm init -y
npm install --save express pug # notice we are installing pug as well!
mkdir views
touch views/index.pug
```
Let's now get started in our `app.js`:
```js
var express = require("express");
var app = express();

app.set('view engine', 'pug') // notice here we are telling express to render views using the pug templating engine.

app.get('/', function(request, response){
    response.render('index') // here we are rendering a template called index.pug inside of the views folder
})

app.listen(3000, function(){
    console.log("The server has started on port 3000. Head to localhost:3000 in the browser and see what's there!")
})
```
#### Working with pug
The most important thing to understand about `pug` is that it is an indentation based syntax. To nest elements inside of each other we simply indent the inner elements.
```js
var express = require("express");
var app = express();

var colors = ["red", "green", "blue"]

app.set('view engine', 'pug') // notice here we are telling express to render views using the pug templating engine.

app.get('/', function(request, response){
    var firstName = "Elie"
    response.render('index', {name: firstName}) // here we are rendering a template called index.pug inside of the views folder. We are also passing a value to the template called "name". Inside of the template, the value of that variable will be the value of the firstName variable, which is "Elie"
})

app.get('/colors', function(request, response){
    // {colors} is ES2015 object shorthand notation for {colors: colors}
    response.render('data', {colors}) // here we are rendering a template called data.pug inside of the views folder. We are also passing a value to the template called "colors". Inside of the template, the value of that variable will be the value of the colors variables, which is an array.
})

app.listen(3000, function(){
    console.log("The server has started on port 3000. Head to localhost:3000 in the browser and see what's there!")
})
```
To evaluate variables inside of our templates, we will use the Jade `#{variable}` syntax. When you do string concatenation with Jade - use the ES2015 string templates with the `${javascript to interpolate}` syntax. Here is what our `index.pug` file might look like:
```
h1 Hello #{name}
```
And our `colors.pug` - notice we are using the Jade syntax `#{}` for variables and ES2015 `${}` for string concatentation/interpolation.
```
h1 Here are the colors!
each color in colors
    p #{color}
    a(href=`/colors/${color}`) Learn more about this color
```
#### Express and pug
Now in our `views/index.pug` file let's place the following:
```html
<!DOCTYPE html>
html(lang="en")
head
    meta(charset="UTF-8")
    title Document
body
    h1 "Hello World!"
    p Outer Paragraph
        .foo A div with a class of foo
    h2 Let's do some math! #{5+5}
```
#### Template inheritance with pug
One very useful feature that `pug` gives us is called *template inheritance*. What this means is that we can use the same template in all of our files and only have to define it once! We can include templates using the `extends` keyword. We also need to tell our template where to inject HTML from other files; we do this with the `block` keyword.

Let's look at an example. In our views folder, let's create a file called `base.pug`.
```html
<!DOCTYPE html>
html(lang="en")
head
    meta(charset="UTF-8")
    title Document
body
    block content
```
Next, let's create a `hello.pug`, extend our `base.pug` file, and include our own content! We just have to make sure that the content specific to `hello.pug` is indented inside of `block content`
```
extends base.pug

block content
    h1 Hello World!
```
For more examples of the features available to you in pug, take a look at the [documentation](https://pugjs.org/).

#### css/js using `express.static`
Before we continue with routing, let's examine how to tell our server how to serve static assets (css/javascript/images). To do this we create a folder called `public` in the root directory. We can then add the following in our `app.js`:
```js
var express = require("express");
var app = express();

app.set("view engine", "pug");
app.use(express.static(__dirname + "/public")); // we are using the express.static middleware and specifying a path for static files to be found (__dirname is a variable that we can use to refer to the directory name of the current module)

// even though we were previously using request and response as names of parameters, you will very commonly see these as req and res.
app.get("/", function(req, res, next){
  res.render("index");
});

app.listen(3000, function(){
  console.log("Server is listening on port 3000");
});
```
#### Example App
You can see an example of this [here](https://github.com/rithmschool/node_curriculum_examples/tree/master/pug_intro).





### Routing and CRUD

Routing refers to determining how an application responds to a client request to a particular endpoint, which is a URI (or path) and a specific HTTP request method (GET, POST, and so on).

Each route can have one or more handler functions, which are executed when the route is matched.

Route definition takes the following structure:
```javascript
app.METHOD(PATH, HANDLER)
```
Where:

- app is an instance of express.
- METHOD is an HTTP request method, in lowercase.
- PATH is a path on the server.
- HANDLER is the function executed when the route is matched.

This chapter assumes that an instance of express named app is created and the server is running. If you are not familiar with creating an app and starting it, see the Hello world example.
The following examples illustrate defining simple routes.

Respond with Hello World! on the homepage:
```javascript
app.get('/', function (req, res) {
  res.send('Hello World!')
})
```
Respond to POST request on the root route (/), the application’s home page:
```javascript
app.post('/', function (req, res) {
  res.send('Got a POST request')
})
```
Respond to a PUT request to the /user route:
```javascript
app.put('/user', function (req, res) {
  res.send('Got a PUT request at /user')
})
```
Respond to a DELETE request to the /user route:
```javascript
app.delete('/user', function (req, res) {
  res.send('Got a DELETE request at /user')
})
```
#### what wikipedia says??
Routing, in a narrower sense of the term, is often contrasted with bridging in its assumption that network addresses are structured and that similar addresses imply proximity within the network. Structured addresses allow a single routing table entry to represent the route to a group of devices. In large networks, structured addressing (routing, in the narrow sense) outperforms unstructured addressing (bridging). Routing has become the dominant form of addressing on the Internet. Bridging is still widely used within localized environments.

### why's and how's??
#### CRUD
Now that we have an idea of how to collect data from our forms we can start to build some applications! We'll start by building an application that does full CRUD on a resource. Let's first define what we mean by CRUD and resource.

**CRUD** - An *acronym* for *Create*, *Read*, *Update*, and *Delete*. When we talk about building an application with "full CRUD", that means we can perform all of those operations on whatever resources we are working on.

**Resource** - A resource refers to a noun that we are operating on. For example if our resource was "users" and we wanted full CRUD on the "users" resources, we would have routes for the following (these routes are using a convention called RESTful routing):

|   HTTP Verb   | Path    | Description    |
|---------------|:----------|------------------|
|GET            |`/users`     | Show all users |
|GET            |`/users/new` |Show a form for creating a new user|
|GET            |`/users/:id` | Show a single user |
|GET            |`/users/:id/edit`|Show a form for editing a user|
|POST         |`/users`     |Create a user when a form is submitted|
|PATCH          |`/users/:id` |Edit a user when a form is submitted|
|DELETE         |`/users/:id` |Delete a user when a form is submitted|

#### `methodOverride`
You may have noticed that in the last two routes in the above table, we are using different HTTP verbs! These verbs are `PATCH` and `DELETE` and serve the purpose of updating a resource and deleting a resource.

The problem is that HTML forms only know the GET and POST methods, so we need to override these methods and change them to be PATCH and DELETE for certain requests. This can either be done using a header in the request, or through the query string. To make this process easier, we will use a module called `method-override`.
```
npm install --save method-override
```
```js
// include the method override module
var methodOverride = require("method-override");
// use the method override module and specify the name of the key in the query string which we will assing a value of an HTTP verb to override the POST method (_method=DELETE or _method=PATCH)
app.use(methodOverride("_method"));
```
To use method override, we simply attach the key of `_method` and value of an HTTP verb like `PATCH` or `DELETE` as the value into the query string. You **must** make sure that your method on the form is `POST` as well, even though we will be overriding it. Here is what a form might look like.
```
form(action=`/create-new-user/${user.id}?_method=PATCH` method="POST")
    input(type="text" name="first" value=`${first}`)
    input(type="text" name="last" value=`${last}`)
```
#### CRUD on users
Now that we have an idea of how to build our routes, let's create a simple CRUD application for our users resource. Let's get started with a simple express app in the terminal.
```
mkdir users_crud && cd users_crud
touch app.js
npm init -y
npm install --save express pug body-parser method-override
mkdir views
touch views/{base,index,new,edit,show}.pug # create 5 pug files in the views folder
echo node_modules > .gitignore
git init
git add .
git commit -m "initial commit"
```
Now that we have that folder structure, let's add the following to our app.js
```js
var express = require("express");
var app = express();
var methodOverride = require("method-override");
var bodyParser = require("body-parser");

app.set("view engine", "pug");
app.use(express.static(__dirname + "/public"));
app.use(bodyParser.urlencoded({extended:true}));
app.use(methodOverride("_method"));

var users = []; // this would ideally be a database, but we'll start with something simple to store our users
var id = 1; // this will help us identify unique users for finding, editing, and deleting

app.get("/", function(req, res, next){
  res.redirect("/users");
});

app.get('/users', function(req, res, next){
  res.render('index', {users}); // {users:} is ES2015 object shorthand for {users:users}
});

app.get('/users/new', function(req, res, next){
  // render a page called new.pug inside the views folder. In this file there will be a form that when submitted will POST to /users
  res.render('new');
});

app.get('/users/:id', function(req, res, next){
  // find a user by their id using the find method on arrays. We also have to make sure that we convert req.params.id into a number from a string so we can safely compare
  var user = users.find(val => val.id === Number(req.params.id));
  // render the show page passing in the user we found
  res.render('edit', {user});
});

app.get('/users/:id/edit', function(req, res, next){
  // find a user by their id using the find method on arrays. We also have to make sure that we convert req.params.id into a number from a string so we can safely compare
  var user = users.find(val => val.id === Number(req.params.id));
  // render the edit page passing in the user we found
  res.render('edit', {user});
});

// this route will be requested when a form for creating a user is submitted
app.post('/users', function(req, res, next){
  // add an object to the users array
  users.push({
    // with the key of name and the value of the data from the form
    name: req.body.name
    // and a key of id and a value of id (the variable we are using as a counter)
    id,
  });
  // increment the id for the next user to be added
  id++;
  res.redirect('/users')
});

// this route will be requested when a form updating a user is submitted
app.patch('/users/:id', function(req, res, next){
  // find the user
  var user = users.find(val => val.id === Number(req.params.id));
  // update their name with the data from the form
  user.name = req.body.name;
  res.redirect('/users');
});

// this route will be requested when a form for deleting a user is submitted
app.delete('/users/:id', function(req, res, next){
  // instead of finding the entire user, let's just see what index they are at in our users array
  var userIndex = users.findIndex(val => val.id === Number(req.params.id));
  // then we can remove one item from the users array starting at the correct index we found above
  users.splice(userIndex,1);
  res.redirect('/users');
});

app.listen(3000, function(){
  console.log("Server is listening on port 3000");
});
```
You may also notice that we are starting to place a `next` parameter in our callback functions. We will see in a later section why this is so helpful, but in a nutshell it is useful for handling errors and making sure they are propagated to the top.

#### Sample App
You can see an example of RESTful routing [here.](https://github.com/rithmschool/node_curriculum_examples/tree/master/express_restful_routing)



### Midddlewares
Middleware functions are functions that have access to the request object (req), the response object (res), and the next middleware function in the application’s request-response cycle. The next middleware function is commonly denoted by a variable named next.


Middleware functions can perform the following tasks:

- Execute any code.
- Make changes to the request and the response objects.
- End the request-response cycle.
- Call the next middleware function in the stack.

If the current middleware function does not end the request-response cycle, it must call next() to pass control to the next middleware function. Otherwise, the request will be left hanging.


### why's and how's??
#### Middleware
Middleware is code that gets run in between the request and the response. We've seen some examples of middleware already; ``body-parser`` and ``method-override`` are both examples of middleware. But we can write our own middleware as well! We will also be using our own custom middleware to configure the express router and handle errors.

To include middleware we use the app.use function. Here are two examples:
```javascript
// on every single request, run the following
app.use(function(req, res, next){
    console.log('Middleware just ran!')
    next()
})
// on every single request to anything that starts with /users, run the following
app.use('/users', function(req, res, next){
    console.log('Middleware just ran on a user route!')
    next()
})
```
#### Error Handling
Another tool that we can add to our applications is an easier way to manage errors (especially in production). This middleware allows errors to be built up and finally sent to an errors page. This pattern is also quite useful when you have lots of different levels of nesting and there's potential for errors to be uncaught.
```javascript
// catch 404 and forward to error handler
app.use(function(req, res, next) {
  var err = new Error('Not Found');
  err.status = 404;
  next(err);
});

// error handlers

// development error handler
// will print stacktrace - make sure you have a file called views/error.pug to render this error message!
if (app.get('env') === 'development') {
  app.use(function(err, req, res, next) {
    res.status(err.status || 500);
    res.render('error', {
      message: err.message,
      error: err
    });
  });
}

// production error handler
// no stacktraces leaked to user
app.use(function(err, req, res, next) {
  res.status(err.status || 500);
  res.render('error', {
    message: err.message,
    error: {}
  });
});
```
You can read more about why it is so valuable to always have the next parameter and use it [here](https://derickbailey.com/2014/09/06/proper-error-handling-in-expressjs-route-handlers/) and [here](https://expressjs.com/en/guide/error-handling.html).

#### ``morgan``
Before we continue on, let's examine a useful module that will help log out requests and responses to the terminal. This module is called morgan, and can be very helpful when you're trying to debug your application.
```
mkdir another_express_app && cd another_express_app
touch app.js
npm init -y
npm install --save express pug body-parser morgan
```
Here is what our app.js might look like with all our middleware set up!
```javascript
var express = require("express");
var app = express();
var morgan = require("morgan");
var bodyParser = require("body-parser");

app.set("view engine", "pug");
app.use(express.static(__dirname + "/public"));
app.use(morgan("tiny"))
app.use(bodyParser.urlencoded({extended:true}));

app.get("/", function(req, res, next){
  res.render("index");
});

// catch 404 and forward to error handler
app.use(function(req, res, next) {
  var err = new Error('Not Found');
  err.status = 404;
  next(err);
});

// error handlers

// development error handler
// will print stacktrace
if (app.get('env') === 'development') {
  app.use(function(err, req, res, next) {
    res.status(err.status || 500);
    res.render('error', {
      message: err.message,
      error: err
    });
  });
}

// production error handler
// no stacktraces leaked to user
app.use(function(err, req, res, next) {
  res.status(err.status || 500);
  res.render('error', {
    message: err.message,
    error: {}
  });
});


app.listen(3000, function(){
  console.log("Server is listening on port 3000");
});
```
#### Generator
So far we have been generating our express applications starting with a brand new folder, installing packages and building applications from scratch. There is a faster tool for the job called express-generator, which you can install using :

    npm install -g express-generator.
 We will not be making use of this module since it's valuable to learn how to create these applications from scratch, but it is a commonly used tool for generating applications quickly. You can read more about it [here](https://expressjs.com/en/starter/generator.html).

#### Debugging Node
One easy way to debug node application is to simply ``console.log`` values and see what the error could be. Unfortunately, this is quite time consuming so there is a better tool called ``locus``, which you can install using

      npm install --save locus
 It helps to pause your code at a certain line so that you can then go to the terminal and examine variables and write JavaScript. Once you have installed the module, place the line eval(``require("locus")``) in your application and when your code reaches that point, you can debug in the terminal.



### Forms
An HTML Form is a group of one or more fields/widgets on a web page that can be used to collect information from users for submission to a server. Forms are a flexible mechanism for collecting user input because there are suitable form inputs available for entering many different types of data—text boxes, checkboxes, radio buttons, date pickers, etc. Forms are also a relatively secure way of sharing data with the server, as they allow us to send data in POST requests with cross-site request forgery protection.

#### why's and how's??

The form is defined in HTML as a collection of elements inside \<form>...\</form> tags, containing at least one input element of type="submit".
```html
<form action="/team_name_url/" method="post">
    <label for="team_name">Enter name: </label>
    <input id="team_name" type="text" name="name_field" value="Default name for team.">
    <input type="submit" value="OK">
</form>
```
 The ``field's type`` attribute defines what sort of widget will be displayed. The name and id of the field are used to identify the field in JavaScript/CSS/HTML, while value defines the initial value for the field when it is first displayed. The matching team label is specified using the label tag (see "Enter name" above), with a for field containing the id value of the associated input.

The submit input will be displayed as a button (by default)—this can be pressed by the user to upload the data contained by the other input elements to the server (in this case, just the team_name). The form attributes define the HTTP method used to send the data and the destination of the data on the server (action):

- action: The resource/URL where data is to be sent for processing when the form is submitted. If this is not set (or set to an empty string), then the form will be submitted back to the current page URL.
- method: The HTTP method used to send the data: POST or GET.
    - The POST method should always be used if the data is going to result in a change to the server's database, because this can be made more resistant to cross-site forgery request attacks.
    - The GET method should only be used for forms that don't change user data (e.g. a search form). It is recommended for when you want to be able to bookmark or share the URL.
## Form handling process

Form handling uses all of the same techniques that we learned for displaying information about our models: 
 the route sends our request to a controller function which performs any database actions required, including reading data from the models, then generates and returns an HTML page. What makes things more complicated is that the server also needs to be able to process data provided by the user, and redisplay the form with error information if there are any problems.

The main things that form handling code needs to do are are:

- Display the default form the first time it is requested by the user.
    - The form may contain blank fields (e.g. if you're creating a new record), or it may be pre-populated with initial values (e.g. if you are changing a record, or have useful default initial values).
- Receive data submitted by the user, usually in an HTTP POST request.
- Validate and sanitize the data.
- If any data is invalid, re-display the form—this time with any user populated values and error messages for the problem fields.
- If all data is valid, perform required actions (e.g. save the data in the database, send a notification email, return the result of a search, upload a file, etc.)
- Once all actions are complete, redirect the user to another page.

Often form handling code is implemented using a GET route for the initial display of the form and a POST route to the same path for handling validation and processing of form data. This is the approach that will be used in this tutorial!

Express itself doesn't provide any specific support for form handling operations, but it can use middleware to process POST and GET parameters from the form, and to validate/sanitize their values.

#### Validation and sanitization

Before the data from a form is stored it must be validated and sanitized:

- Validation checks that entered values are appropriate for each field (are in the right range, format, etc.) and that values have been supplied for all required fields.
- Sanitization removes/replaces characters in the data that might potentially be used to send malicious content to the server.
For this Chapter we'll be using the popular ``express-validator`` module to perform both validation and sanitization of our form data.

#### Installation

Install the module by running the following command in the root of the project.

    npm install express-validator --save
#### Add the validator to the app middleware

Open ./app.js and import the express-validator module after the other modules (near the top of the file, as shown).
```js
...
var cookieParser = require('cookie-parser');
var bodyParser = require('body-parser');
var expressValidator = require('express-validator');
```
Further down in the file call app.use() to add the validator to the middleware stack. This should be done after the code that adds the bodyParser to the middleware stack (express-validator uses the body-parser to access parameters).
```js
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: false }));
app.use(expressValidator()); // Add this after the bodyParser middlewares!
```
#### Using express-validator

For this chapter, we'll primarily be using the following APIs:

- **checkBody(parameter, message)**: Specifies a body (POST) parameter to validate along with a message to be displayed if it fails the tests. The validation criteria are daisy chained to the checkBody() method. For example, the first check below will test that the "name" parameter is alphanumeric and set an error message "Invalid name" if it is not. The second test checks that the age parameter has an integer value.
```js
req.checkBody('name', 'Invalid name').isAlpha();
req.checkBody('age', 'Invalid age').notEmpty().isInt();
```
- sanitizeBody(parameter): Specifies a body parameter to sanitize. The sanitization operations are then daisy-chained to this method. For example, the escape() sanitization operation below removes HTML characters from the name variable that might be used in JavaScript cross-site scripting attacks.
```js
req.sanitizeBody('name').escape();
```
To run the validation we call req.validationErrors(). This returns an array of error objects (or false if there are no errors) and is typically used like this
```js
var errors = req.validationErrors();
if (errors) {
    // Render the form using error information
}
else {
   // There are no errors so perform action with valid data (e.g. save record).
}
```
Each error object in the array has values for the parameter, message and value.
```js
{param: 'name', msg: 'Invalid name', value: '<received input>'}
```
Note: The API also has methods to check and sanitize query and url parameters (not just body parameters as shown). For more information see: express-validator (npm).
We'll cover more examples when we implement the LocalLibrary forms below.

#### Form design

Many of the models in the library are related/dependent—for example, a Book requires an Author, and may also have one or more Genres. This raises the question of how we should handle the case where a user wishes to:

- Create an object when its related objects do not yet exist (for example, a book where the author object hasn't been defined).
- Delete an object that is still being used by another object (so for example, deleting a Genre that is still being used by a Book).
For this project we will simplify the implementation by stating that a form can only:

- Create an object using objects that already exist (so users will have to create any required Author and Genre instances before attempting to create any Book objects).
- Delete an object if it is not referenced by other objects (so for example, you won't be able to delete a Book until all associated BookInstance objects have been deleted).

#### Routes

In order to implement our form handling code we will need two routes that have the same URL pattern. The first (GET) route is used to display a new empty form for creating the object. The second route (POST) is used for validating data entered by the user, and then saving the information and redirecting to the detail page (if the data is valid) or redisplaying the form with errors (if the data is invalid).

We have already created the routes for all our model's create pages in /routes/catalog.js (in a previous tutorial). For example, the genre routes are shown below:
```js
/* GET request for creating a Genre. NOTE This must come before route that displays Genre (uses id) */
router.get('/genre/create', genre_controller.genre_create_get);

/* POST request for creating Genre. */
router.post('/genre/create', genre_controller.genre_create_post);
```
#### Create genre form
This section shows how we define our page to create Genre objects (this is a good place to start because the Genre has only one field, its name, and no dependencies). Like any other pages, we need to set up routes, controllers, and views.

##### Controller—get route

Open /controllers/genreController.js. Find the exported genre_create_get() controller method and replace it with the following code. This simply renders the genre_form.pug view, passing a title variable.
```js
// Display Genre create form on GET
exports.genre_create_get = function(req, res, next) {       
    res.render('genre_form', { title: 'Create Genre' });
};
```
##### Controller—post route

Find the exported genre_create_post() controller method and replace it with the following code.
```js
// Handle Genre create on POST 
exports.genre_create_post = function(req, res, next) {
    
    //Check that the name field is not empty
    req.checkBody('name', 'Genre name required').notEmpty(); 
    
    //Trim and escape the name field. 
    req.sanitize('name').escape();
    req.sanitize('name').trim();
    
    //Run the validators
    var errors = req.validationErrors();

    //Create a genre object with escaped and trimmed data.
    var genre = new Genre(
      { name: req.body.name }
    );
    
    if (errors) {
        //If there are errors render the form again, passing the previously entered values and errors
        res.render('genre_form', { title: 'Create Genre', genre: genre, errors: errors});
    return;
    } 
    else {
        // Data from form is valid.
        //Check if Genre with same name already exists
        Genre.findOne({ 'name': req.body.name })
            .exec( function(err, found_genre) {
                 console.log('found_genre: ' + found_genre);
                 if (err) { return next(err); }
                 
                 if (found_genre) { 
                     //Genre exists, redirect to its detail page
                     res.redirect(found_genre.url);
                 }
                 else {
                     
                     genre.save(function (err) {
                       if (err) { return next(err); }
                       //Genre saved. Redirect to genre detail page
                       res.redirect(genre.url);
                     });
                     
                 }
                 
             });
    }

};
```
The first part of this code defines a validator to check that the name field is not empty (and an error message if it is empty), sanitizes and trims (removes whitespace at either end of the string) the value, and then runs the validator.
```js
//Check that the name field is not empty
req.checkBody('name', 'Genre name required').notEmpty(); 
    
//Trim and escape the name field. 
req.sanitize('name').escape();
req.sanitize('name').trim();
    
//Run the validators
var errors = req.validationErrors();
If there are errors we render the template with the form again, this time additionally passing a variable with any (sanitized) values passed by the user, and an object containing the error information.

//Create a genre object with escaped and trimmed data.
var genre = new Genre({ name: req.body.name });

if (errors) {
    res.render('genre_form', { title: 'Create Genre', genre: genre, errors: errors});
    return;
} 
```
If the data is valid then we check if a Genre with the same name already exists (we don't want to create duplicates). If it does we redirect to the existing genre's detail page. If not, we save the new Genre and redirect to its detail page.
```js
//Check if Genre with same name already exists
Genre.findOne({ 'name': req.body.name })
    .exec( function(err, found_genre) {
    console.log('found_genre: '+found_genre)
    if (err) { return next(err); }
                 
        if (found_genre) { 
            //Genre exists, redirect to its detail page
            res.redirect(found_genre.url);
            }
        else {
            genre.save(function (err) {
                if (err) { return next(err); }
                    //Genre saved. Redirect to genre detail page
                    res.redirect(genre.url);
                });
        }
                 
});
```
##### View

The same view is rendered in both the GET and POST controllers (routes). In the GET case the form is empty and we just pass a title variable. In the POST case the user has previously entered invalid data—in the genre variable we pass back the entered data (sanitized) and in the errors variable we pass back error messages.
```js
res.render('genre_form', { title: 'Create Genre'});
res.render('genre_form', { title: 'Create Genre', genre: genre, errors: errors});
Create /views/genre_form.pug and copy in the text below.

extends layout

block content
  h1 #{title}

  form(method='POST' action='')
    div.form-group
      label(for='name') Genre:
      input#name.form-control(type='text', placeholder='Fantasy, Poetry etc.' name='name' value=(undefined===genre ? '' : genre.name))
    button.btn.btn-primary(type='submit') Submit

  if errors 
    ul
      for error in errors
        li!= error.msg
```
Much of this template will be familiar from our previous tutorials. First we extend the layout.pug base template and override the block named 'content'. We then have a heading with the title we passed in from the controller (via the render() method).

Next we have the pug code for our HTML form that uses the POST method to send the data to the server, and because the action is an empty string, will send the data to the same URL as the page.

The form defines a single required field of type "text" called "name". The default value of the field depends on whether the genre variable is defined. If called from the GET route it will be empty as this is a new form. If called from a POST route it will contain the (invalid) value originally entered by the user.

The last part of the page is the error code. This simply prints a list of errors, if the error variable has been defined (in other words, this section will not appear when the template is rendered on the GET route).

#### Create author form
This section shows how to define a page for creating Author objects.

##### Controller—get route

Open /controllers/authorController.js. Find the exported author_create_get() controller method and replace it with the following code. This simply renders the author_form.pug view, passing a title variable.
```js
// Display Author create form on GET
exports.author_create_get = function(req, res, next) {       
    res.render('author_form', { title: 'Create Author'});
};
```
##### Controller—post route

Find the exported author_create_post() controller method, and replace it with the following code.
```js
// Handle Author create on POST 
exports.author_create_post = function(req, res, next) {
   
    req.checkBody('first_name', 'First name must be specified.').notEmpty(); //We won't force Alphanumeric, because people might have spaces.
    req.checkBody('family_name', 'Family name must be specified.').notEmpty();
    req.checkBody('family_name', 'Family name must be alphanumeric text.').isAlpha();
    req.checkBody('date_of_birth', 'Invalid date').optional({ checkFalsy: true }).isDate();
    req.checkBody('date_of_death', 'Invalid date').optional({ checkFalsy: true }).isDate();
    
    req.sanitize('first_name').escape();
    req.sanitize('family_name').escape();
    req.sanitize('first_name').trim();     
    req.sanitize('family_name').trim();
    req.sanitize('date_of_birth').toDate();
    req.sanitize('date_of_death').toDate();

    var errors = req.validationErrors();
    
    var author = new Author(
      { first_name: req.body.first_name, 
        family_name: req.body.family_name, 
        date_of_birth: req.body.date_of_birth,
        date_of_death: req.body.date_of_death
       });
       
    if (errors) {
        res.render('author_form', { title: 'Create Author', author: author, errors: errors});
    return;
    } 
    else {
    // Data from form is valid
    
        author.save(function (err) {
            if (err) { return next(err); }
               //successful - redirect to new author record.
               res.redirect(author.url);
            });
    }

};
```
The structure and behaviour of this code is almost exactly the same as for creating a Genre object. First we validate and sanitize the data. If the data is invalid then we re-display the form along with the data that was originally entered by the user and a list of error messages. If the data is valid then we save the new author record and redirect the user to the author detail page.

The validation code demonstrates two new features:

- We can use the optional() function to run a subsequent validation only if a field has been entered (this allows us to validate optional fields). For example, below we check that the optional date of birth is a date (the checkFalsy flag means that we'll accept either an empty string or null as an empty value).
```js
req.checkBody('date_of_birth', 'Invalid date').optional({ checkFalsy: true }).isDate();
```
- Parameters are recieved from the request as strings. We can use toDate() (or toBoolean(), etc.) to cast these to the proper JavaScript types.
```js
req.sanitize('date_of_birth').toDate()
```
### View

Create /views/author_form.pug and copy in the text below.
```html
extends layout

block content
  h1=title

  form(method='POST' action='')
    div.form-group
      label(for='first_name') First Name:
      input#first_name.form-control(type='text', placeholder='First name (Christian) last' name='first_name' required='true' value=(undefined===author ? '' : author.first_name) )
      label(for='family_name') Family Name:
      input#family_name.form-control(type='text', placeholder='Family name (surname)' name='family_name' required='true' value=(undefined===author ? '' : author.family_name))
    div.form-group
      label(for='date_of_birth') Date of birth:
      input#date_of_birth.form-control(type='date', name='date_of_birth', value=(undefined===author ? '' : author.date_of_birth) )
    button.btn.btn-primary(type='submit') Submit
  if errors 
    ul
      for error in errors
        li!= error.msg
```
The structure and behaviour for this view is exactly the same as for the genre_form.pug template, so we won't describe it again.

Note: Some browsers don’t support the input type=“date”, so you won’t get the datepicker widget or the default dd/mm/yyyy placeholder, but will instead get an empty plain text field. One workaround is to explicitly add the attribute placeholder='dd/mm/yyyy' so that on less capable browsers you will still get information about the desired text format.
Challenge: The template above is missing a field for entering the date_of_death. Create the field following the same pattern as the date of birth form group!
#### Create book form
This section shows how to define a page/form to create Book objects. This is a little more complicated than the equivalent Author or Genre pages because we need to get and display available Author and Genre records in our Book form.

##### Controller—get route

Open /controllers/bookController.js. Find the exported book_create_get() controller method and replace it with the following code.
```js
// Display book create form on GET
exports.book_create_get = function(req, res, next) { 
      
    //Get all authors and genres, which we can use for adding to our book.
    async.parallel({
        authors: function(callback) {
            Author.find(callback);
        },
        genres: function(callback) {
            Genre.find(callback);
        },
    }, function(err, results) {
        if (err) { return next(err); }
        res.render('book_form', { title: 'Create Book', authors: results.authors, genres: results.genres });
    });
    
};
```
This uses the async module (described in Express Tutorial Part 5: Displaying library data) to get all Author and Genre objects. These are then passed to the view book_form.pug as variables named authors and genres (along with the page title).

##### Controller—post route

Find the exported book_create_post() controller method and replace it with the following code.
```js
// Handle book create on POST 
exports.book_create_post = function(req, res, next) {

    req.checkBody('title', 'Title must not be empty.').notEmpty();
    req.checkBody('author', 'Author must not be empty').notEmpty();
    req.checkBody('summary', 'Summary must not be empty').notEmpty();
    req.checkBody('isbn', 'ISBN must not be empty').notEmpty();
    
    req.sanitize('title').escape();
    req.sanitize('author').escape();
    req.sanitize('summary').escape();
    req.sanitize('isbn').escape();
    req.sanitize('title').trim();     
    req.sanitize('author').trim();
    req.sanitize('summary').trim();
    req.sanitize('isbn').trim();
    req.sanitize('genre').escape();
    
    var book = new Book({
        title: req.body.title, 
        author: req.body.author, 
        summary: req.body.summary,
        isbn: req.body.isbn,
        genre: (typeof req.body.genre==='undefined') ? [] : req.body.genre.split(",")
    });
       
    console.log('BOOK: ' + book);
    
    var errors = req.validationErrors();
    if (errors) {
        // Some problems so we need to re-render our book

        //Get all authors and genres for form
        async.parallel({
            authors: function(callback) {
                Author.find(callback);
            },
            genres: function(callback) {
                Genre.find(callback);
            },
        }, function(err, results) {
            if (err) { return next(err); }
            
            // Mark our selected genres as checked
            for (i = 0; i < results.genres.length; i++) {
                if (book.genre.indexOf(results.genres[i]._id) > -1) {
                    //Current genre is selected. Set "checked" flag.
                    results.genres[i].checked='true';
                }
            }

            res.render('book_form', { title: 'Create Book',authors:results.authors, genres:results.genres, book: book, errors: errors });
        });

    } 
    else {
    // Data from form is valid.
    // We could check if book exists already, but lets just save.
    
        book.save(function (err) {
            if (err) { return next(err); }
            //successful - redirect to new book record.
            res.redirect(book.url);
        });
    }

};
```
The structure and behaviour of this code is almost exactly the same as for creating a Genre object. First we validate and sanitize the data. If the data is invalid then we re-display the form along with the data that was originally entered by the user and a list of error messages. If the data is valid, we then save the new Book record and redirect the user to the book detail page.

Again, the main difference with respect to the other form handling code is that we need to pass in all existing genres and authors to the form. In order to mark the genres that were checked by the user we iterate through all the genres and add the checked='true' parameter to those that were in our post data (as reproduced in the code fragment below).
```js
// Mark our selected genres as checked
for (i = 0; i < results.genres.length; i++) {
    if (book.genre.indexOf(results.genres[i]._id) > -1) {
        //Current genre is selected. Set "checked" flag.
        results.genres[i].checked='true';
    }
}
```
##### View

Create /views/book_form.pug and copy in the text below.
```html
extends layout

block content
  h1= title

  form(method='POST' action='')
    div.form-group
      label(for='title') Title:
      input#title.form-control(type='text', placeholder='Name of book' name='title' required='true' value=(undefined===book ? '' : book.title) )
    div.form-group
      label(for='author') Author:
      select#author.form-control(type='select', placeholder='Select author' name='author' required='true' )
        for author in authors
          if book
            option(value=author._id selected=(author._id.toString()==book.author ? 'selected' : false) ) #{author.name}
          else
            option(value=author._id) #{author.name}
    div.form-group
      label(for='summary') Summary:
      input#summary.form-control(type='textarea', placeholder='Summary' name='summary' value=(undefined===book ? '' : book.summary) required='true')
    div.form-group
      label(for='isbn') ISBN:
      input#isbn.form-control(type='text', placeholder='ISBN13' name='isbn' value=(undefined===book ? '' : book.isbn) required='true') 
    div.form-group
      label Genre:
      div
        for genre in genres
          div(style='display: inline; padding-right:10px;')
            input.checkbox-input(type='checkbox', name='genre', id=genre._id, value=genre._id, checked=genre.checked )
            label(for=genre._id) #{genre.name}
    button.btn.btn-primary(type='submit') Submit

  if errors 
    ul
      for error in errors
        li!= error.msg
```
The view structure and behaviour is almost the same as for the genre_form.pug template.

The main differences are in how we implement the selection-type fields: Author and Genre.

- The set of genres are displayed as checkboxes, using the checked value we set in the controller to determine whether or not the box should be selected.
- The set of authors are displayed as a single-selection drop-down list. In this case we determine what author to display by comparing the id of the current author option with the value previously entered by the user (passed in as the book variable). This is highlighted above!
## Create BookInstance form
This section shows how to define a page/form to create BookInstance objects. This is very much like the form we used to create Book objects.

##### Controller—get route

Open /controllers/bookinstanceController.js.

At the top of the file, require the Book module (needed because each BookInstance is associated with a particular Book).
```js
var Book = require('../models/book');
```
Find the exported bookinstance_create_get() controller method and replace it with the following code.
```js
// Display BookInstance create form on GET
exports.bookinstance_create_get = function(req, res, next) {       

    Book.find({},'title')
    .exec(function (err, books) {
      if (err) { return next(err); }
      //Successful, so render
      res.render('bookinstance_form', {title: 'Create BookInstance', book_list:books});
    });
    
};
```
The controller gets a list of all books (book_list) and passes it to the view bookinstance_form.pug (along with the title)

##### Controller—post route

Find the exported bookinstance_create_post() controller method and replace it with the following code.
```js
// Handle BookInstance create on POST 
exports.bookinstance_create_post = function(req, res, next) {

    req.checkBody('book', 'Book must be specified').notEmpty(); //We won't force Alphanumeric, because book titles might have spaces.
    req.checkBody('imprint', 'Imprint must be specified').notEmpty();
    req.checkBody('due_back', 'Invalid date').optional({ checkFalsy: true }).isDate();
    
    req.sanitize('book').escape();
    req.sanitize('imprint').escape();
    req.sanitize('status').escape();
    req.sanitize('book').trim();
    req.sanitize('imprint').trim();   
    req.sanitize('status').trim();
    req.sanitize('due_back').toDate();
    
    var bookinstance = new BookInstance({
        book: req.body.book,
        imprint: req.body.imprint, 
        status: req.body.status,
        due_back: req.body.due_back
    });

    var errors = req.validationErrors();
    if (errors) {
        
        Book.find({},'title')
        .exec(function (err, books) {
          if (err) { return next(err); }
          //Successful, so render
          res.render('bookinstance_form', { title: 'Create BookInstance', book_list : books, selected_book : bookinstance.book._id , errors: errors, bookinstance:bookinstance });
        });
        return;
    } 
    else {
    // Data from form is valid
    
        bookinstance.save(function (err) {
            if (err) { return next(err); }
               //successful - redirect to new book-instance record.
               res.redirect(bookinstance.url);
            }); 
    }

};
```
The structure and behaviour of this code is the same as for creating our other objects. First we validate and sanitize the data. If the data is invalid, we then re-display the form along with the data that was originally entered by the user and a list of error messages. If the data is valid, we save the new BookInstance record and redirect the user to the detail page.

##### View

Create /views/bookinstance_form.pug and copy in the text below.
```html
extends layout

block content
  h1=title

  form(method='POST' action='')
    div.form-group
      label(for='book') Book:
      select#book.form-control(type='select' placeholder='Select book' name='book' required='true')
        for book in book_list
          if bookinstance
            option(value=book._id selected=(bookinstance.book.toString()==book._id.toString() ? 'selected' : false)) #{book.title}
          else
            option(value=book._id) #{book.title}
        
    div.form-group
      label(for='imprint') Imprint:
      input#imprint.form-control(type='text' placeholder='Publisher and date information' name='imprint' required='true' value=(undefined===bookinstance ? '' : bookinstance.imprint))
    div.form-group
      label(for='due_back') Date when book available:
      input#due_back.form-control(type='date' name='due_back' value=(undefined===bookinstance ? '' : bookinstance.due_back))
            
    div.form-group
      label(for='status') Status:
      select#status.form-control(type='select' placeholder='Select status' name='status' required='true')
        option(value='Maintenance') Maintenance
        option(value='Available') Available
        option(value='Loaned') Loaned
        option(value='Reserved') Reserved

    button.btn.btn-primary(type='submit') Submit

  if errors 
    ul
      for error in errors
        li!= error.msg
```
The view structure and behaviour is almost the same as for the book_form.pug template, so we won't go over it again.


#### Delete author form
This section shows how to define a page to delete Author objects.

As discussed in the form design section, our strategy will be to only allow deletion of objects that are not referenced by other objects (in this case that means we won't allow an Author to be deleted if it is referenced by a Book). In terms of implementation this means that the form needs to confirm that there are no associated books before the author is deleted. If there are associated books, it should display them, and state that they must be deleted before the Author object can be deleted.

##### Controller—get route

Open /controllers/authorController.js. Find the exported author_delete_get() controller method and replace it with the following code.
```js
// Display Author delete form on GET
exports.author_delete_get = function(req, res, next) {       

    async.parallel({
        author: function(callback) {     
            Author.findById(req.params.id).exec(callback);
        },
        authors_books: function(callback) {
          Book.find({ 'author': req.params.id }).exec(callback);
        },
    }, function(err, results) {
        if (err) { return next(err); }
        //Successful, so render
        res.render('author_delete', { title: 'Delete Author', author: results.author, author_books: results.authors_books } );
    });
    
};
```
The controller gets the id of the Author instance to be deleted from the URL parameter (req.params.id). It uses the async.parallel() method to get the author record and all associated books in parallel. When both operations have completed it renders the author_delete.pug view, passing variables for the title, author, and author_books.

##### Controller—post route

Find the exported author_delete_post() controller method, and replace it with the following code.
```js
// Handle Author delete on POST 
exports.author_delete_post = function(req, res, next) {

    req.checkBody('authorid', 'Author id must exist').notEmpty();  
    
    async.parallel({
        author: function(callback) {     
            Author.findById(req.body.authorid).exec(callback);
        },
        authors_books: function(callback) {
          Book.find({ 'author': req.body.authorid },'title summary').exec(callback);
        },
    }, function(err, results) {
        if (err) { return next(err); }
        //Success
        if (results.authors_books>0) {
            //Author has books. Render in same way as for GET route.
            res.render('author_delete', { title: 'Delete Author', author: results.author, author_books: results.authors_books } );
            return;
        }
        else {
            //Author has no books. Delete object and redirect to the list of authors.
            Author.findByIdAndRemove(req.body.authorid, function deleteAuthor(err) {
                if (err) { return next(err); }
                //Success - got to author list
                res.redirect('/catalog/authors');
            });

        }
    });

};
```
First we validate that an id has been provided (this is sent via the form body parameters, rather than using the version in the URL). Then we get the author and their associated books in the same way as for the GET route. If there are no books then we delete the author object and redirect to the list of all authors. If there are still books then we just re-render the form, passing in the author and list of books to be deleted.

##### View

Create /views/author_delete.pug and copy in the text below.
```html
extends layout

block content
  h1 #{title}: #{author.name}
  p= author.lifespan
  
  if author_books.length
  
    p #[strong Delete the following books before attempting to delete this author.]
  
    div(style='margin-left:20px;margin-top:20px')

      h4 Books
    
      dl
      each book in author_books
        dt 
          a(href=book.url) #{book.title}
        dd #{book.summary}

  else
    p Do you really want to delete this Author?
    
    form(method='POST' action='')
      div.form-group
        input#authorid.form-control(type='hidden',name='authorid', required='true', value=author._id )

      button.btn.btn-primary(type='submit') Delete
```
The view extends the layout template, overriding the block named content. At the top it displays the author details. It then includes a conditional statement based on the number of author_books (the if and else clauses).

If there are books associated with the author then the page lists the books and states that these must be deleted before this Author may be deleted.
If there are no books then the page displays a confirmation prompt. If the Delete button is clicked then the author id is sent to the server in a POST request and that author's record will be deleted.
##### Add a delete control

Next we will add a Delete control to the Author detail view (the detail page is a good place from which to delete a record).

Note: In a full implementation the control would be made visible only to authorised users. However at this point we haven't got an authorisation system in place!
Open the author_detail.pug view and add the following lines at the bottom.
```
hr
p
  a(href=author.url+'/delete') Delete author
```
#### Update Book form
This section shows how to define a page to update Book objects. Form handling when updating a book is much like that for creating a book, except that you must populate the form in the GET route with values from the database.

##### Controller—get route

Open /controllers/bookController.js. Find the exported book_update_get() controller method and replace it with the following code.
```js
// Display book update form on GET
exports.book_update_get = function(req, res, next) {

    req.sanitize('id').escape();
    req.sanitize('id').trim();

    //Get book, authors and genres for form
    async.parallel({
        book: function(callback) {
            Book.findById(req.params.id).populate('author').populate('genre').exec(callback);
        },
        authors: function(callback) {
            Author.find(callback);
        },
        genres: function(callback) {
            Genre.find(callback);
        },
    }, function(err, results) {
        if (err) { return next(err); }
            
        // Mark our selected genres as checked
        for (var all_g_iter = 0; all_g_iter < results.genres.length; all_g_iter++) {
            for (var book_g_iter = 0; book_g_iter < results.book.genre.length; book_g_iter++) {
                if (results.genres[all_g_iter]._id.toString()==results.book.genre[book_g_iter]._id.toString()) {
                    results.genres[all_g_iter].checked='true';
                }
            }
        }
        res.render('book_form', { title: 'Update Book', authors:results.authors, genres:results.genres, book: results.book });
    });
    
};
```
The controller gets the id of the Book to be updated from the URL parameter (req.params.id). It uses the async.parallel() method to get the specified Book record (populating its genre and author fields) and lists of all the Author and Genre objects. When all operations have completed it marks the currently selected genres as checked and then renders the book_form.pug view, passing variables for title, book, all authors, and all genres.

##### Controller—post route
Find the exported book_update_post() controller method, and replace it with the following code.

```js
// Handle book update on POST 
exports.book_update_post = function(req, res, next) {
    
    //Sanitize id passed in. 
    req.sanitize('id').escape();
    req.sanitize('id').trim();
    
    //Check other data
    req.checkBody('title', 'Title must not be empty.').notEmpty();
    req.checkBody('author', 'Author must not be empty').notEmpty();
    req.checkBody('summary', 'Summary must not be empty').notEmpty();
    req.checkBody('isbn', 'ISBN must not be empty').notEmpty();
    
    req.sanitize('title').escape();
    req.sanitize('author').escape();
    req.sanitize('summary').escape();
    req.sanitize('isbn').escape();
    req.sanitize('title').trim();
    req.sanitize('author').trim(); 
    req.sanitize('summary').trim();
    req.sanitize('isbn').trim();
    req.sanitize('genre').escape();
    
    var book = new Book(
      { title: req.body.title, 
        author: req.body.author, 
        summary: req.body.summary,
        isbn: req.body.isbn,
        genre: (typeof req.body.genre==='undefined') ? [] : req.body.genre.split(","),
        _id:req.params.id //This is required, or a new ID will be assigned!
       });
    
    var errors = req.validationErrors();
    if (errors) {
        // Re-render book with error information
        // Get all authors and genres for form
        async.parallel({
            authors: function(callback) {
                Author.find(callback);
            },
            genres: function(callback) {
                Genre.find(callback);
            },
        }, function(err, results) {
            if (err) { return next(err); }
            
            // Mark our selected genres as checked
            for (i = 0; i < results.genres.length; i++) {
                if (book.genre.indexOf(results.genres[i]._id) > -1) {
                    results.genres[i].checked='true';
                }
            }
            res.render('book_form', { title: 'Update Book',authors:results.authors, genres:results.genres, book: book, errors: errors });
        });

    } 
    else {
        // Data from form is valid. Update the record.
        Book.findByIdAndUpdate(req.params.id, book, {}, function (err,thebook) {
            if (err) { return next(err); }
            //successful - redirect to book detail page.
            res.redirect(thebook.url);
        });
    }

};```


 This is very similar to the post route used when creating a Book. First we validate and sanitize the book data from the form and use it to create a new Book object (setting its _id value to the id of the object to update). If there are errors when we validate the data then we re-render the form, additionally displaying the data entered by the user, the errors, and lists of genres and authors. If there are no errors then we call Book.findByIdAndUpdate() to update the Book document, and then redirect to its detail page.



##### View

Open /views/book_form.pug and update the section where the author form control is set to have the conditional code shown below.
```js
    div.form-group
      label(for='author') Author:
      select#author.form-control(type='select' placeholder='Select author' name='author' required='true' )
        for author in authors
          if book
            //- Handle GET form, where book.author is an object, and POST form, where it is a string.
            option(
              value=author._id
              selected=(
                author._id.toString()==book.author._id
                || author._id.toString()==book.author
              ) ? 'selected' : false
            ) #{author.name}
          else
            option(value=author._id) #{author.name}
```
Note: This code change is required so that the book_form can be used for both creating and updating book objects (without this, there is an error on the GET route when creating a form).
Add an update button

Open the book_detail.pug view and make sure there are links for both deleting and updating books at the bottom of the page, as shown below.

    hr
    p
        a(href=book.url+'/delete') Delete Book
    p
        a(href=book.url+'/update') Update Book
You should now be able to update books from the Book detail page.


### Routing
Use the express.Router class to create modular, mountable route handlers. A Router instance is a complete middleware and routing system; for this reason, it is often referred to as a “mini-app”.

The following example creates a router as a module, loads a middleware function in it, defines some routes, and mounts the router module on a path in the main app.

Create a router file named birds.js in the app directory, with the following content:
```javascript
var express = require('express')
var router = express.Router()

// middleware that is specific to this router
router.use(function timeLog (req, res, next) {
  console.log('Time: ', Date.now())
  next()
})
// define the home page route
router.get('/', function (req, res) {
  res.send('Birds home page')
})
// define the about route
router.get('/about', function (req, res) {
  res.send('About birds')
})

module.exports = router
```
Then, load the router module in the app:
```javascript
var birds = require('./birds')

// ...

app.use('/birds', birds)
```
The app will now be able to handle requests to /birds and /birds/about, as well as call the timeLog middleware function that is specific to the rout

#### Router
So far we have seen how to create routes in express and render templates and even respond with redirects! As we start building more and more routes, our app.js can get very messy quickly. In order to clean this up, we will move our routes to a file in a folder called ``routes``. The files in this folder will contain all of our routes and we will export them using the express router. Let's get started with a simple application.

#### RESTful Review
We saw earlier that there is an idea of how to standardize our routes using something called RESTful routing. REST or REpresentational State Transfer is a standard for designing web applications, but the important takeaway here is that standardizing our routes is very important and allows us to work with other developers and across a vareity of projects using the same structure. You can have a look of what RESTful routes for a users resource look like in 3rd chapter.

#### Getting started
Now that we have a good structure for our RESTful routes, let's use the express router to re-create our users application. Let's get started with a new express app. In the terminal, let's run the following

    mkdir express-router && cd express-router
    touch app.js
    npm init -y
    npm install express pug body-parser method-override
    mkdir views routes
    touch routes/users.js
    touch views/{base,index,new,edit,show}.pug

Now let's add the following to our routes/users.js
```javascript
// we need to bring in express
var express = require("express");
// so that we can create a router object. This router object will have similar methods (.get, .post, .patch, .delete) to the app object we have previously been using.
var router = express.Router();

var users = []; // this would ideally be a database, but we'll start with something simple
var id = 1; // this will help us identify unique users

// instead of app.get...
router.get('/users', function(req, res, next){
  res.render('index', {users}); // {users:} is ES2015 object shorthand for {users:users}
});

router.get('/users/new', function(req, res, next){
  res.render('new');
});

router.get('/users/:id', function(req, res, next){
  var user = users.find(val => val.id === Number(req.params.id));
  res.render('show', {user});
});

router.get('/users/:id/edit', function(req, res, next){
  var user = users.find(val => val.id === Number(req.params.id));
  res.render('edit', {user});
});

// instead of app.post...
router.post('/users', function(req, res, next){
  users.push({
    name: req.body.name
    id: ++id
  });
  res.redirect('/users');
});

// instead of app.patch...
router.patch('/users/:id', function(req, res, next){
  var user = users.find(val => val.id === Number(req.params.id));
  user.name = req.body.name;
  res.redirect('/users');
});

// instead of app.delete...
router.delete('/users/:id', function(req, res, next){
  var userIndex = users.findIndex(val => val.id === Number(req.params.id));
  users.splice(userIndex,1);
  res.redirect('/users');
});

// Now that we have built up all these routes - let's export this module for use in our app.js!
module.exports = router;
```
Now, how do we actually use these routes? Let's add the following in our app.js
```javascript
var express = require("express");
var app = express();
var methodOverride = require("method-override");
var morgan = require("morgan");
var bodyParser = require("body-parser");

// require our routes/users.js file
var userRoutes = require("./routes/users");

app.set("view engine", "pug");
app.use(express.static(__dirname + "/public"));
app.use(morgan("tiny"));
app.use(bodyParser.urlencoded({extended:true}));
app.use(methodOverride("_method"));

// Now let's tell our app about those routes we made!
app.use(userRoutes);

app.get("/", function(req, res, next) {
  res.redirect('/users');
});

app.listen(3000, function() {
  console.log("Server is listening on port 3000");
});
We could also write these routes using an alternate syntax for express routing.

var express = require("express");
var router = express.Router();

var users = []; // this would ideally be a database, but we'll start with something simple
var id = 1; // this will help us identify unique users

router.route('/users')
    .get(function(){
      res.render('index', {users});
    })
    .post(function(){
      users.push({
        name: req.body.name
        id: ++id
      });
      res.redirect('/users');
    });

router.route('/users/new').get(function(req, res, next) {
  res.render('new');
});

router.route('/users/:id/edit').get(function(req, res, next) {
  var user = users.find(val => val.id === Number(req.params.id));
  res.render('edit', {user});
});

router.route('/users/:id')
    var user = users.find(val => val.id === Number(req.params.id));
    .get(function(req, res, next){
        res.render('show', {user});
    });
    .patch(function(req, res, next){
      user.name = req.body.name;
      res.redirect('/users')
    });
    .delete(function(req, res, next){
      users.splice(user.id,1)
      res.redirect('/users')
    });

module.exports = router;
```
Even better - we can prefix all of our routes to start with /users if we would like. Here is what that looks like in our routes/users.js
```javascript
// we need to bring in express
var express = require("express");
// so that we can create a router object. This router object will have similar methods (.get, .post, .patch, .delete) to the app object we have previously been using.
var router = express.Router();

var users = []; // this would ideally be a database, but we'll start with something simple
var id = 1; // this will help us identify unique users

// no need to start with /users since we will prefix all routes in this file with /users
router.get('/', function(req, res, next) {
  res.render('index', {users}); // {users:} is ES2015 object shorthand for {users:users}
});

// no need to start with /users since we will prefix all routes in this file with /users
router.get('/new', function(req, res, next) {
  res.render('new');
});

// no need to start with /users since we will prefix all routes in this file with /users
router.get('/:id', function(req, res, next) {
  var user = users.find(val => val.id === Number(req.params.id));
  res.render('show', {user});
});

// no need to start with /users since we will prefix all routes in this file with /users
router.get('/:id/edit', function(req, res, next) {
  var user = users.find(val => val.id === Number(req.params.id));
  res.render('edit', {user});
});

// no need to start with /users since we will prefix all routes in this file with /users
router.post('/', function(req, res, next) {
  users.push({
    name: req.body.name
    id: ++id
  });
  res.redirect('/users');
});

// no need to start with /users since we will prefix all routes in this file with /users
router.patch('/:id', function(req, res, next) {
  var user = users.find(val => val.id === Number(req.params.id));
  user.name = req.body.name;
  res.redirect('/users');
});

// no need to start with /users since we will prefix all routes in this file with /users
router.delete('/:id', function(req, res, next) {
  var userIndex = users.findIndex(val => val.id === Number(req.params.id));
  users.splice(userIndex,1);
  res.redirect('/users');
});

// Now that we have built up all these routes - let's export this module for use in our app.js!
module.exports = router;
And here in our app.js

// prefix every single route in here with /users
app.use('/users', userRoutes);
```

#### Sample App
You can see an example of the express router [here](https://github.com/rithmschool/node_curriculum_examples/tree/master/express_router)


#### What else?
- Now you have everything you need in order to create your edit/update view. Here's what you have to do:

  - Create an edit route
  - Create a form (see the edit wireframe below) that has the fields from the album pre-populated
  - The form should post to /albums/:id/update
  - The update route should update that record, and redirect to the show page

You can put almost all of this together yourself:

- You can see how to define dynamic routes from the show route
- You can see how to find the individual record from the show route
- You know how to redirect from the create route
- Do these using ``express.Route()``

#
- We should be using an array to store our items as well as assigning a unique id to each item so that we can find them easily. Our application should have the following routes:

    - GET /items - to show all items in the shopping list
    - GET /items/new - to show a form for creating a new item
    - GET /items/:id - to show a single item
    - GET /items/:id/edit - to show a form for editing a item
    - POST /items - to create an item when a form is submitted
    - PATCH /items/:id - to edit an item when a form is submitted
    - DELETE /items/:id - to delete an item when a form is submitted

#

Implement the delete pages for the Book, BookInstance, and Genre models, linking them from the associated detail pages in the same way as our Author delete page. The pages should follow the same design approach:

- If there are references to the object from other objects, then these other objects should be displayed along with a note that this record can't be deleted until the listed objects have been deleted.
- If there are no other references to the object then the view should prompt to delete it. If the user presses the Delete button, the record should then be deleted.
A few tips:

- Deleting a Genre is just like deleting an Author as both objects are dependencies of Books (so in both cases you can delete the object only when the associated books are deleted.
- Deleting a Book is also similar, but you need to check that there are no associated BookInstances.
- Deleting a BookInstance is the easiest of all, because there are no dependent objects. In this case you can just find the associated record and delete it.
Implement the update pages for the BookInstance, Author, and Genre models, linking them from the associated detail pages in the same way as our Book update page.

A few tips:

- The Book update page we just implemented is the hardest! The same patterns can be used for the update pages for the other objects.
- The Author date of death and date of birth fields, and the BookInstance due_date field are the wrong format to input into the date input field on the form (it requires data in form "YYYY-MM-DD"). The easiest way to get around this is to define a new virtual property for the dates that formats the dates appropriately, and then use this field in the associated view templates.
- If you get stuck, there are examples of the update pages in the example [here](https://github.com/mdn/express-locallibrary-tutorial).

## Exploration
- [ExpressJS](https://developer.mozilla.org/en-US/docs/Learn/Server-side/Express_Nodejs) on MDN
- Web application frameworks [comparison](https://www.airpair.com/node.js/posts/nodejs-framework-comparison-express-koa-hapi)
- [Best practices](https://expressjs.com/en/advanced/best-practice-performance.html)
- Express 4.0: [What's new](https://scotch.io/bar-talk/expressjs-4-0-new-features-and-upgrading-from-3-0)
- ExpressJS [Design patterns](https://canho.me/category/expressjs/)



