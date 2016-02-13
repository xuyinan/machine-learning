# Assignment 3: Javascript and the Frontend Web
### Due Tuesday, October 27th @ 11:55:00 PM EDT

## Introduction
​
Within the last few years, there have been dramatic shifts in the way web applications are built. Starting around 2011 with the rise of Backbone, to the present-day situation where React, Ember, Angular and other front end frameworks are seeing widespread industry use, the front-end web is constantly evolving. All of these frameworks focus on a type of website known as a single page application (SPA), which loads views/templates asynchronously on the client-side instead of being rendered server-side. Ever go on Facebook and notice the activity feed live updating without clicking a button or reloading the page? That is an example of a component of a SPA. This shift has not only had implications on the frontend (that is, the HTML/CSS/JS), but also backend systems have shifted to focusing on REST API-first principles. Creating a common interface (the API) has been critical in an era where its clients can take any form (mobile applications, frontend web frameworks, IoT devices, etc).

## Objectives

The goal of this project is to introduce you to the following:
*Help you develop a deeper understanding of client-side Javascript in modern web applications
*Understand why we have seen a rise in frontend frameworks and what problems they help solve
*Gain experience with the asynchronous programming model which is core to JavaScript

For many students, this may be their first in-depth experience with JavaScript. Do not worry if you are in that boat. Javascript is currently in a period of rapid change. The JavaScript standards committee (TC-39) is currently aiming for yearly updates to the language. The important aspect of this project is not that you understand Javascript's syntax perfectly, but instead that you understand its asynchronous programming model. Pay attention to callbacks, and take a look at Promises (used in parts 2 an 3 of this project) which are currently being implemented by all major browser vendors.

There are three main parts to this project:  
* Part 1 will use the traditional, server-side rendered page
* Part 2 will involve writing your own Javascript to perform HTML/JS data binding
* Part 3 will introduce a client-side JS framework aimed to make the process easier (optional extra credit)

For parts 1 and 2, **jQuery is not allowed.** However, in part 3 (extra credit) Ember makes use of jQuery, so feel free to also make use of it. 

## Part 1: The Server Side Method

For parts 1 and 2, we will be allowing users to edit the caption on the photo. The first method will be using the server side method that you're already used to.

Currently, when a user goes to `/pic?id=picid` it will display the image with the picid, previous and next buttons to move between photos within the album and the caption. In Project 2, you were responsible for having the ability to edit the picture capture on the `/pic?id=picid` page. Now, we'll be allowing any user that can view the page the ability to edit the caption in "real" time at the `/pic?id=picid` route. Your code should have looked similar to the following: 
```
<form name="input" action="{{ url_for('pic.pic_route') }}" method="post">
	<input type="hidden" name="id" value={{ picid }} />
	<input type="text" name="caption" value={{ caption }}/>
	<button type="submit">Edit</button><br>
</form>
``` 
Play around with your version of this form. Notice how every time you click the "Edit"/"Submit" button it does a complete page reload? What if you could eliminate that delay to make for a smoother user experience? What if every time a user typed into the input field the data was automatically persisted to the server without the user clicking the edit button? How about making it possible to automatically update the caption if another user edits it? That is what we'll cover in part 2.

Please note there is nothing to submit for part one.

## Part 2: HTML Data Binding

In part 2, we will learn about HTML/JS data binding. Data binding involves binding an html attribute (in our case, it will be the caption input) to its data source. If the data source is updated, then the view (your HTML) should reflect this change. If the input form is updated, then the data source should also be updated (and persisted to the server). This is commonly referred to as two-way data binding.

### Part 2a: Data Binding Captions

1. Modify our base.html template such that it loads our JS scripts that we will be executing. When your page is rendered, the web browser will first retrieve and execute all of the scripts in the head tags before rendering the body. Therefore, we will be loading all of our JS files in the head, and after they are loaded, executing them with function calls in our body (specifically in the "extrascripts" block). Your base.html should look similar to the following:
	```
	<head>
		<script type="text/javascript" src="/static/js/qwest.min.js"></script>
	   	<script type="text/javascript" src="/static/js/caption.js"></script>
	    	<!-- ... other head content here ... -->
	</head>
	<body>
		<!-- ... body content here ... -->
		{% block extrascripts %} <!-- This is where we'll load inline scripts specific for a given template -->
		{% endblock %}
	</body>
	```

2. [Download a version of qwest](https://raw.githubusercontent.com/pyrsmk/qwest/master/qwest.min.js), a small AJAX request library we will be using for this part. Place it in whatever directory linked to above (ex. /static/js/qwest.min.js).

3. Modify our HTML where the pic?id=picid is rendered such that it has an input element with an id of "caption". Having this id or "identifier" allows us to uniquely identify our input element from our Javascript. Remember, Javascript will have access to the DOM, but it still needs to be able to find our input caption element. It does so using this id. We also need to add the Javascript necessary to initialize our data binding For example:
```
<!-- pic.html -->
<div>
	<b>Caption: </b><input id="caption" />
</div>
{% block extrascripts %}
	<script type="text/javascript">
		initCaption("{{ picid }}"); <! -- assuming picid is passed to pic.html from your controller -->
	</script>
{% endblock %} 
```

4. Create a file called caption.js. A good location would be in a directory such as /static/js/caption.js. Now, let's walk through what is in caption.js. **This function should be located at the bottom of your caption.js file** (we will be adding more code above it in the next steps).
```
// caption.js
function initCaption(picid) {
  var caption = document.getElementById("caption");
  var captionBinding = new Caption(caption, picid);
  
  makeCaptionRequest(picid, function(resp) {
    captionBinding.change(resp['caption']);
  });

  setInterval(function() {
   makeCaptionRequest(picid, function(resp) {
      captionBinding.change(resp['caption']);
    }); 
  }, 7000);
}
```
This function takes the picid of the current picture and creates the binding between the caption input and the data. The first two lines locate the caption in the HTML DOM and initialize a binding. The Caption object will be covered below. The next block of code queries our data store (the Flask server which in turn queries MySQL) for the initial caption data. Finally, it sets up a timer such that every 7 seconds, it will query the data store and update the caption with the latest fetched data. In an ideal scenario, you'd have a better system set-up for "real-time" updates by either shortening the timer or using something such as Websockets to push changes to the client. However, for our purposes, this will be fine.

Next, let's write the function that handles querying our server.
```
// caption.js
function makeCaptionRequest(picid, cb) {
  qwest.get('/secretkey/pa3/pic/caption?id=' + picid)
    .then(function(xhr, resp) {
      cb(resp);
    });
}
```
This function utilizes the qwest library to abstract away a lot of the boilerplate involved in making an [XMLHttpRequest/AJAX](http://www.w3schools.com/ajax/ajax_xmlhttprequest_send.asp). Remember, a core part of Javascript is its asynchronous nature. AJAX requests are asynchronous by default. What happens is this function runs, it issues a get request to the server at the path identified (refer to pic.py below), and moves on to running the other code. It is non-blocking, which means other code can run while the qwest request waits in the background for the server to send a response. Qwest creates a [promise](https://www.promisejs.org/). Once the response arrives, qwest calls our callback function that we inlined (function(xhr, resp)...). Within that callback, we issue a function call to the callback that we passed in and pass the response variable. This response variable should contain our data.

In order for this request to work properly, we need to create server-side routes.

#### GET `/pic/caption?id=picid`
This route queries for the caption for the given picid. The following scenarios should be handled:
* If the caption is retrieved properly, return a JSON object of the following and set the response HTTP status code to 200.
	    
	```json
	{ 
		"caption": "caption here"
	}
	```
	    
* If there is no caption currently set for the picid (null), follow the above response format but set the caption value to be a blank string.
* If the parameter of `id` is not provided, return the following and set the response HTTP status code to 404:
	
	```json
	{
		"error": "You did not provide an id parameter.",
		"status": 404
	}
	```
	
* If the parameter of `id` is invalid (as in the picid does not exist), return the following and set the response HTTP status code to 422:

	```json
	{
		"error": "Invalid id parameter. The picid does not exist.",
		"status": 422
	}
	```
	
* For all other error scenarios, return an error JSON object as formatted above and set the response HTTP status code to 400.

#### POST `/pic/caption`
This route handles submission of a caption for a given picid. The following scenarios should be handled:
* If the caption is inserted properly, return a JSON object of the following and set the response HTTP status code to 201.
	    
	```json
	{ 
		"caption": "caption here",
		"status": 201
	}
	```
	  
* If the JSON POST request does not provide either an id or caption (or both), return a JSON object the following that specifies which fields were not included and set the response HTTP status code to 404:
	
	```json
	{
		"error": "You did not provide an id and caption parameter.", // or "You did not provide an id parameter." or "You did not provide a caption parameter."
		"status": 404
	}
	```
	
* If the JSON POST request provides an invalid picid, return the following and set the response HTTP status code to 422:

	```json
	{
		"error": "Invalid id. The picid does not exist.",
		"status": 422
	}
	```
	
* For all other error scenarios, return an error JSON object as formatted above and set the response HTTP status code to 400.

Here is some example code for server-side routes handling both the GET and POST requests and returning a JSON document with the data. Note that your route does not need to look exactly like this (for example, InvalidPicIDError is a custom exception defined elsewhere).
```
# pic.py where pic is a [Blueprint](http://flask.pocoo.org/docs/0.10/blueprints/) with a url_prefix of /secretkey/pa3/pic
@pic.route('/caption', methods=['GET'])
def pic_caption_get():
	'''
	Expects URL query parameter with picid.
	Returns JSON with the picture's current caption or error.
	{
		"caption": "current caption"
	}
	{
		"error": "error message",
		"status": 422
	}
	'''	
	try:
		picid = get_picid(request)
	except InvalidPicIDError as err:
		response = json.jsonify(error='Could not retrieve caption. You did not provide a picture id.', status=404)
		response.status_code = 404
		return response


	query = "SELECT caption FROM Contain WHERE picid='%s';" % (picid)
	results = application.execute(query)
	caption = None
	if len(results) > 0:
		caption = results[0][0]
	else:
		response = json.jsonify(error='Could not retrieve caption. You did not provide a valid picture id.', status=422)
		response.status_code = 422
		return response
	return json.jsonify(caption=caption)

@pic.route('/caption', methods=['POST'])
def pic_caption_post():
	'''
	Expects JSON POST of the format:
	{
		"caption": "this is the new caption",
		"id": "picid"
	}
	Updates the caption and sends a response of the format
	{
		"caption": "caption",
		"status": 201
	}
	Or if an error occurs:
	{
		"error": "error message",
		"status": 422
	}
	''' 
	req_json = request.get_json()
	
	picid = req_json.get('id')
	caption = req_json.get('caption')
	if picid is None and caption is None:
		response.json.jsonify(error='Could not update caption. You did not provide a valid picture id or caption.', status=404)
		response.status_code = 404
		return response
	if picid is None:
		response = json.jsonify(error='Could not update caption. You did not provide a valid picture id.', status=404)
		response.status_code = 404
		return 
	if caption is None:
		response = json.jsonify(error='Could not update caption. You did not provide a valid caption.', status=404)
		response.status_code = 404
		return response

	try:
		query = "UPDATE Contain SET caption='%s' WHERE picid='%s';" % (caption, picid)
		application.update(query)
	except InvalidPicIDError as e:
		response = json.jsonify(error='Could not update caption. The picture id was not valid.', status=422)
		response.status_code = 422
		return response

	response = json.jsonify(id=picid, status=201)
	response.status_code = 201
	return response
```

Congratulations, with those two functions of code, you've created a very simple REST API for handling captions. Note that we do not check user permissions for these routes as it is not required for this project. If you were to ever write an API for production-use, you would definitely want to include an authentication system such as one that uses [OAuth](https://en.wikipedia.org/wiki/OAuth) or [JSON Web Tokens](http://jwt.io/).

Now, to jump back to our frontend code. Here is a function that can handle POSTing to our REST API.
```
// caption.js
function makeCaptionPostRequest(picid, caption, cb) {
  var data = {
    'id': picid,
    'caption': caption
  };

  qwest.post('/secretkey/pa3/pic/caption', data, {
    dataType: 'json',
    responseType: 'json'
  }).then(function(xhr, resp) {
    cb(resp);
  });
}
```

Lastly, and most importantly, we need to handle changes on the data from either the store or the client. **This code should be put first in caption.js so that your initCaption function can reference it.**
```
// caption.js
function Caption(element, picid, caption) {
  this.element = element;
  this.picid = picid;
  element.value = caption; // objects in Javascript are assigned by reference, so this works
  element.addEventListener("change", this, false); 
}

Caption.prototype.handleEvent = function(e) {
  if (e.type === "change") {
    this.update(this.element.value);
  }
}

Caption.prototype.change = function(value) {
  this.data = value;
  this.element.value = value;
}

Caption.prototype.update = function(caption) {
  makeCaptionPostRequest(this.picid, caption, function() {
    console.log('POST successful.');
  });
}
```
First, we create a constructor for our Caption object (don't worry, [JS Classes](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes) are coming!) It is JS best practice to name constructor functions with the first letter being capitilized. In this constructor, we initialize the element attribute, store the picid and assign the value to the caption. Next, we add an event listener. Whenever a browser event fires on our element by the name of ["change"](https://developer.mozilla.org/en-US/docs/Web/Events/change), we will be notified because the handleEvent method will be called. We override this method so that if it's a change event, we call our update function which persists the changes to the server.

Your caption input should always be editable when a user goes to `/pic?id=picid`. If there is already a caption for the picture, then it should be preloaded with the caption. 

#### Deliverables
For part 2a, you are responsible for the following deliverables:
* Implementing the two-way data binding described above
* Creating the API routes specified

#### Testing
To test your website's caption editing, have a teammate open up the site and modify the caption or open up an incognito window/different browser and sign-in as another user. You should see the changes persist and update every seven seconds. To test your API routes are working correctly, install the (Postman extension)[https://chrome.google.com/webstore/detail/postman/fhbjgbiflinjbdggehcddcbncdddomop?hl=en]. You can manually create HTTP requests to your server. It is also a good idea to open up your browser's Javascript console to debug issues. Network requests can be monitored (see what your GET/POST requests look like), and code can even be ran in the console.

### Part 2b: Data Binding Favorites

Now that you have an idea on how two-way data binding works, it will be your job to add a data binded favorites system to our pictures.

To begin, we need to update our database (for this project, we will be using group##pa3) with the following table:
```
create table Favorite (
	favoriteid integer primary key not null auto_increment,
	picid varchar(40) not null,
	username varchar(20) not null,
	date timestamp default current_timestamp,

	foreign key (picid) references Photo (picid),
	foreign key (username) references User (username)
);
```
Refer to the file `pa3_sql.sql` for the sql necessary for pa3, including sample data that needs to be loaded onto your server.

Note that you may add any attributes, triggers, helper SQL, and change the way this relationship is represented if it helps you. The SQL code is not a focus for this project.

Every photo can have multiple favorites, however a user can only like the same photo once.

When navigating to `/pic?id=picid`, the user should see all of the information stated above and the data binded caption input created in part 2a. Directly below the photo, they should also see text of the format "## favorites, most recently favorited by username". The most recently favorited and number of favorites should update immediately after a user submits their own favorite. The favorites and latest username should be polled for updates every 10 seconds.

Your group will be required to create the following server-side API routes.
#### GET `/pic/favorites?id=picid`
This route queries for the favorites for the given picid. The following scenarios should be handled:
* If the favorites are retrieved properly, return a JSON object of the following and set the response HTTP status code to 200.
	    
	```json
	{ 
		"id": "picid",
		"num_favorites": ###,
		"latest_favorite": "username"
	}
	```
	    
* If there are no favorites currently set for the picid (null), follow the above response format but set the num_favorites to 0 and the latest_favorite value to be a blank string.
* If the parameter of `id` is not provided, return the following and set the response HTTP status code to 404:
	
	```json
	{
		"error": "You did not provide an id parameter.",
		"status": 404
	}
	```
	
* If the parameter of `id` is invalid (as in the picid does not exist), return the following and set the response HTTP status code to 422:

	```json
	{
		"error": "Invalid id parameter. The picid does not exist.",
		"status": 422
	}
	```
	
* For all other error scenarios, return an error JSON object as formatted above and set the response HTTP status code to 400.

#### POST `/pic/favorites`
This route handles submission of a caption for a given picid. It should accept a JSON object of the format:
```json
{
	"id": "picid",
	"username": "usernameofpersonwhofavorited"
}
```

The following scenarios should be handled:
* If the favorite is inserted properly, return a JSON object of the following and set the response HTTP status code to 201.
	    
	```json
	{ 
		"id": "picid",
		"status": 201
	}
	```
	  
* If the JSON POST request does not provide either an id or username (or both), return a JSON object the following that specifies which fields were not included and set the response HTTP status code to 404:
	
	```json
	{
		"error": "You did not provide an id and username parameter.", // or "You did not provide an id parameter." or "You did not provide a username parameter."
		"status": 404
	}
	```
Note that previous versions of the spec had an error message of "You did not provide an id and caption parameter." That will also be accepted.

* If the JSON POST request provides an invalid picid, return the following and set the response HTTP status code to 422:

	```json
	{
		"error": "Invalid id. The picid does not exist.",
		"status": 422
	}
	```

* If the JSON POST request provides an invalid username, return the following and set the response HTTP status code to 422:

	```json
	{
		"error": "Invalid username. The username does not exist.",
		"status": 422
	}
	```

* If the user has already favorited the photo, return the following and set the response HTTP status code to 403:

	```json
	{
		"error": "The user has already favorited this photo.",
		"status": 403
	}
	```

* For all other error scenarios, return an error JSON object as formatted above and set the response HTTP status code to 400.

If your group would like to develop a caching procedure that requires more extensive information, then do so with another route. The format of these JSON objects should not differ (ordering of the attributes does not matter).

How favorites are displayed is a group decision. It can be as simple as a simple button with the name of "favorite" or it can be something that provides a better user experiene such as a heart icon overlayed on the picture. No matter what, keep the feature obvious so that users are able to quickly favorite pictures.   

Once again, security of these routes is not pivotal. It is not your responsibility in this project to guarantee that the routes are accessed by authenticated users. For example, it is okay that any person can send a POST request with a username (even if that user does not have access to view the picture). However, if a user goes to  `/pic?id=picid` page and does not have permission to view the page, they should not be allowed to view it. If it is a public page, the user should be able to view the favorites, but they themselves not favorite the picture since they are not logged in. The same permission checks hold true as in project 2. You should also reject requests with invalid information (such as picid and username).

**Your Javascript code for favorites should live in a file called `/static/js/favorites.js`.**

## Part 3: Here Comes the Framework (optional 10% extra credit)

*Please note that this part of the project is optional. Groups who complete this part will gain extensive knowledge with a client-side framework, and as a bonus, up to 10% extra credit on the project.*

After going through part 2, one can quickly realize the complexity involved with binding your DOM with one's data layer. For large-scale applications that have multiple components (and components more complex than just a simple input and text), the code can quickly become messy. Enter frontend (client-side) MVC frameworks whose aim is to provide a consistent way to interact with your server, provide a consistent source of truth data store and push those updates to all of the components that need the data. This also goes back the other way (from view component to server).

In Part 3, we will use [Ember](http://emberjs.com/]) to provide a unified way to address these problems. Before heading any further, it is suggested you read through Ember's documentation to understand some of the terminology and what is happening. While not necessary, reading through [Ember's guides](http://guides.emberjs.com/v2.0.0/) may prove helpful (~2-3 hrs of reading), but most importantly understanding the [Core Concepts](http://guides.emberjs.com/v2.0.0/getting-started/core-concepts/). There are guides on templates, controllers, components, models, etc.

Use of other frameworks is allowed for part 3, however this guide only covers Ember. If another framework is used, the url routes should still be the same. For a tutorial on completing part 3 with Angular to see how Ember and Angular differ, see [here](https://github.com/EECS485-Fall2015/admin/tree/master/pa3/angular.md). This README should be the final source of truth on what is expected for this project. Part 3 will be using Ember, a frontend Javascript MVC framework for single page web applications. We've chosen Ember instead of other frameworks due to its terminology being easier to understand, good documentation, and its widespread popularity. It will be your group's responsibility to read documentation to meet the project's requirements. The course staff has limited knowledge of Ember. That being said, feel free to contact Matt S via office hours or on Piazza if your group runs into issues with Ember on part 3. Remember, this class, especially this project, are not about learning the framework. It's about learning why the framework exists (what problems does it solve) and how does it solve them.

Part 3's web page will live separately from our traditional application at the url `/secretkey/pa3/live`. Flask need not render a Jinja template, as we will be serving Handlebars templates that are rendered client-side by Ember's Handlebars compiler. In order to accomplish this, you should use Flask's send_file function which skips the Jinja process. **Because of this, there are no necessary session/page access checks to be performed. You also should not need to modify the current user's session when entering this page (including the lastupdated session time). This page is public** Here is an example Flask controller:
```
@main.route('/live')
def live_route():
	return send_file('../views/live.html')
``` 
Please note that the directory where this lives may be different for your project. It is not required to live in the views folder.

Within live.html, we will be building our Ember view. This view will have the following features:
* Display the image for the picid passed in as a query parameter
* Display the caption and the ability to edit that caption using an AJAX request
* Displaying favorites (as seen in 2b)
* Allow for users to submit comments asynchronously and view comments on the picture

#### Set-up

First, create a view/template called `live.html` with the following code:
```
<!-- views/live.html or perhaps templates/live.html -->
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN">
<html lang="en">
<head>

  <link rel="stylesheet" href="/static/css/style.css" />
  <script src="//code.jquery.com/jquery-2.1.4.min.js"></script>
  <script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/handlebars.js/4.0.3/handlebars.js"></script>
  <script type="text/javascript" src="http://builds.emberjs.com/release/ember-template-compiler.js"></script>
  <script type="text/javascript" src="http://builds.emberjs.com/release/ember.debug.js"></script>
  <script type="text/javascript" src="http://builds.emberjs.com/release/ember-data.js"></script>
  <script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/moment.js/2.10.6/moment.min.js"></script>
  <script type="text/javascript" src="/static/js/app/app.js"></script>
  <title>EECS485</title>
</head>
<body bgcolor="#66FFCC">
  <script type="text/x-handlebars" data-template-name="pic">
  </script>
  
  <script type="text/x-handlebars" data-template-name="components/comment-submit">
  </script>

  <script type="text/x-handlebars" data-template-name="components/username-input">
  </script>
</body>
</html>
```
In this file we download the Handlebars templating library, the Ember template compiler module, a Ember version meant to be help with development and the ember data module. Next, we set-up three scripts in our body that will encapsulate the Ember template code for our view. We will have a "pic" template that will render the information about the picture. Within it, we will render the comment-submit component which will handle comment submission. Before the page is rendered (client-side), Handlebars will take the Handlebars template code within those scripts and compile them to native HTML that the browser can render.

The last step that may prove valuable is installing the [Ember Inspector](https://chrome.google.com/webstore/detail/ember-inspector/bmdblncegkenkacieihfhpjfppoconhi?hl=en) for Google Chrome. Utilize this to inspect your routes, data, etc.

#### Creating the Ember Application

Next, let's create the app.js which will contain our Ember application. 

```
// /static/js/app/app.js
window.App = Ember.Application.create();

Ember.Application.initializer({
  name: 'usernameValidator',
  after: 'store',
  initialize: function (app) {
    // Inject the Ember Data Store into our validator service
    app.inject('service:username-validator', 'store', 'store:main');

    // Inject the validator into all controllers and routes
    app.inject('controller', 'usernameValidator', 'service:username-validator');
    app.inject('route', 'usernameValidator', 'service:username-validator');
  }
});

App.Router = Ember.Router.extend({
  rootURL: '/secretkey/pa3/live'
});
```
Here we create an Ember application and attach it to the window object with the name App. This means we can refer to it at any point by using "App". You can even type it into your web browser's developer Javascript console to modify it or inspect it. The next thing we do is run the initializer. Whenever the Ember app is created, it will run this function. What we have specified here will play a role in allowing username validation. We have created an Ember "service" and injected it within our other parts of the codebase (controllers and routes). We've also taken our main data store (see below) and made it available to the username-validator service so that it can query the data store. After creating the Ember app, we set a configuration option on its router called the rootURL, which let's Ember know what page is the root page for our Ember application. 

```
App.Store = DS.Store.extend({});

App.ApplicationAdapter = DS.JSONAPIAdapter.extend({
  namespace: '/secretkey/pa3/api/v1'
})
```
Now we let Ember's Data Store (DS) know where our data will be coming from. Ember has several adapters for how to serialize and deserialize data. By default, it uses the [JSONAPI](http://jsonapi.org/) adapter. It expects data coming from our backend REST API to fit the JSONAPI specification, and our backend REST API will be sent data in the format of the JSONAPI specification. 

```
App.Router.map(function() {
  this.route('pic', { path: '/pic/:pic_id' });
  this.route('favorite', { path: '/favorite/:favorite_id' });
  this.route('comment', { path: '/favorite/:comment_id' }); 
  this.route('user', { path: '/user/:username'}) ;
});
```

[Ember's Router](http://guides.emberjs.com/v2.0.0/routing/) helps determine what page a user is on and what information should be loaded. Earlier client side MVC frameworks did not include routers, so when a user would navigate to view their profile, the url would not change. This meant users could not copy and share links and would loser their browser history. Thanks to the recent introduction of the [HTML5 History API](https://developer.mozilla.org/en-US/docs/Web/API/History_API), this problem is no longer an issue as major frameworks now provide a router that supports the API. Here we see the influence that client side frameworks have had on pushing the modern web browser APIs forward. 

Our routes will be located at a URL similar to the following:
`http://hostname:portno/secretkey/pa3/live#/pic/football_s3`

Here we have defined two routes, one for our picture and another for all of the favorites from that picture.

Let's now create our client side Ember data store for the picture. We want to retrieve the picture's file path (pic), the previous and next pictures in the album based on the sequence number, the picture caption and all associated favorites.

```
// app.js

App.Pic = DS.Model.extend({
  picurl: DS.attr('string'),
  prevpicid: DS.attr('string'),
  nextpicid: DS.attr('string'),
  caption: DS.attr('string'),
  favorites: DS.hasMany('favorite'),
  comments: DS.hasMany('comment')
});

App.PicRoute = Ember.Route.extend({
  model: function(params) {
    var pic = this.store.findRecord('pic', params.pic_id);
    return pic;
  },

  actions: {
    save: function() {
      var pic = this.modelFor('pic');
      var caption = this.modelFor('pic').get('caption');
      this.set('caption', caption);
      this.modelFor('pic').save();
    }
  },

  renderTemplate: function() {
    this.render('pic');
  }
});


App.Favorite = DS.Model.extend({
  username: DS.attr('string'),
  datetime: DS.attr('date'),
  pic: DS.belongsTo('pic')
});

App.FavoriteRoute = Ember.Route.extend({
  model: function() {
    return this.store.findRecord();
  }
});
```

Here we define two models, Pic and Favorite. We specify what data types their attribues are, alongside the relationship between the two. For every pic, it can have multiple favorites and comments. Also specified are the models for the routes (what should happen when a route requests a model). In this case, both pic and favorites will find the records in our data store. The data store will in turn query the server if it does not have the information present. 

For a given model, the data store should be the single source of truth. Why is this? You can imagine we add on another route that will display all the pictures that two friends are tagged in together. Instead of having a third data store that repeats the code for getting both users' pictures, it would be wise to just fetch from the two separate data stores. Any other store that depends on them will have the correct information most recently fetched from the server. There will not be inconcsitencies between your templates.

Notice how we've established an action called save on our "pic" route. This will be called when someone triggers the action from our template (as you'll seen see). 

Now that we have our models and routing set-up, let's modify our templates to render the necessary information from our data stores. 
```
<!-- live.html -->
<script type="text/x-handlebars" data-template-name="pic">
  <h1>Picture</h1>
  <img src="{{model.picurl}}">
  <br>
  {{#link-to "pic" model.prevpicid}}Previous{{/link-to}}
  {{#link-to "pic" model.nextpicid}}Next{{/link-to}}
  <br>
  <b>Caption: </b>{{input type="text" value=model.caption enter="save"}}
  <h3>{{model.favorites.length}} favorites</h3>
  {{#each model.favorites as |favorite|}}
    {{favorite.username}}
  {{/each}}
</script>
```
Here we've created a template where our model information can be rendered. It will display the picture and links to the previous and next picture models wil be dynamically generated. Unlike our previous server-rendered set-up, Ember will load the data models for the previous and next pictures asynchronously. No full-page reload required! 

This template is also responsible for displaying the current caption and allowing it to be modified (or added if there is no current caption set on a pic). The "enter" attribute specifies that on enter our "save" action should be fired off. The save action will then modify our data store which then sends a PATCH or a POST request to the server.

Lastly, our template will display the number of favorites alongside the usernames for those who favorited it. How you choose to display this information is up to you. Currently, it's just a list of the usernames, but feel free to modify it so it displays the most recent favorite username and then a drop down view of the remaining users who favorited it. You are not responsible for handling users submitting favorites for this portion of the project.

Your template does not need the ability to view all of an album's pictures. Visitors will be viewing an album's pictures through the picid and prev/next links.

You should have the URL route specified to query by favoriteid. While it may be tempting to include all favorites when querying for /pic, they should be separated and require separate queries. Avoid the premature optimization and maintain a separation of resources. 

So what does the API backend look like? It will be similar to the set-up from part 2, but this time it must follow the [JSONAPI specification](http://jsonapi.org/format/). Below is an example Flask controller for retrieving a favorite by favoriteid. An example api.py file has been provided for you in the repo to help with implementing the API routes. Please modify as necessary to get working for your group's project.
```
@api.route('/favorites/<int:id>')
def favorites(id):
	
	try:
		favorite = get_favorite_by_id(id)
	except RecordNotFound as e:
		response = json.jsonify(errors=[e.to_json()])
		response.status_code = 404
		return response

	data = {
		"type": "favorites",
		"id": id,
		"attributes": {
			"username": favorite.username,
			"datetime": favorite.date.isoformat()
		}
	}
	return json.jsonify(data=data)
```
Please note that the data for these routes is hardcoded, but you should be fetching and updating this information with your database. Feel free to add additional routes. One such example is a favorites route that returns all of the favorites for a given picture.

#### Comments
Now let's implement a comments feature for our single page app.

You should create the following SQL table, `Comment` which will capture the necessary information. Every comment has an id, picid, message, username who submitted the comment and a timestamp.
```
drop table if exists Comment;
create table Comment (
	commentid integer primary key not null auto_increment,
	picid varchar(40)
	message varchar(140) not null,
	username varchar(20) not null,
	date timestamp default current_timestamp,
	
	foreign key (picid) references Photo (picid)
);
```
We will be supporting the following features:

Every picture should display the comments associated with it with the least recently created at the top and most recently at the bottom. This should be after displaying the favorites information. 

At the bottom of the comments, there should be a submission form where a user can submit their own comments. It should have two input fields: the first for a username and the second for their comment message. It should also have a submit button. When a user tabs or clicks out from the username field to the message, you should check to see if the username exists by performing a request to your backend. You should not hardcode the usernames into your Javascript. If the username exists, do nothing. If the username does not exist, then you should print a message right below the username input field saying, "Sorry, the username does not exist." If the user attempts to submit with an invalid username, display an error message right above the submit button stating, "Your comment cannot be submitted as you did not provide a valid username." If you would like to provide a better user experience through also highlighting the input fields red, please feel free to do so. Always try to provide the best user experience when displaying error messages.

#### Implementation
Let's start by modifying our view (live.html) to showcase what we expect this layout to look like and get an idea for what is happening.
```
<!-- live.html -->
<body bgcolor="#66FFCC">
  <script type="text/x-handlebars" data-template-name="pic">
  <h1>Picture</h1>
  <img src="{{model.picurl}}">
  <br>
  {{#link-to "pic" model.prevpicid}}Previous{{/link-to}}
  {{#link-to "pic" model.nextpicid}}Next{{/link-to}}
  <br>
  <b>Caption: </b>{{input type="text" value=model.caption enter="save"}}
  <h3>{{model.favorites.length}} favorites</h3>
  {{#each model.favorites as |favorite|}}
    {{favorite.username}}
  {{/each}}

  <h3>Comments</h3>
  {{#each model.comments as |comment|}}
    {{comment.username}}
    {{comment.date}}
    {{comment.message}}
    <br/>
  {{/each}}
  
  {{#comment-submit action=(action "commentSubmit")}}
  {{/comment-submit}}
  </script>

  <script type="text/x-handlebars" data-template-name="components/comment-submit">
  <form {{action action username message on="submit"}}>
    {{#username-input}}
    {{/username-input}}
    Message: {{input type="text" value=message}}
    <button type="submit">Submit</button>
  </form>
  </script>

  <script type="text/x-handlebars" data-template-name="components/username-input">
  Username: {{input type="text" value=username}}
  <p id="usernameError"></p>
  </script>
</body>
```

As before, we have the necessary code to handle pictures and favorites. This time, however, we've added displaying of the comments that are attached to the pic model. We've also created a compononent (a way to encapsulate a feature) that will handle submitting comments. Because it is a component, it should be relatively easy to reuse this in another system. One can imagine having a similar feature on another website. Generally, one should avoid tying their components to their data stores directly and should use something such as event bus to pass data from the component to the data stores. However, since this is a small application, we'll violate those constraints in the next sections.

The `comment-submit` component has a form. On "submit", an action will be triggered (it is passed into it with the name action) and it will send that action the username and message variables. The username variable is set within the `username-input` component. The primary responsibility of this component will be handling the specifications for username input (verifying the username on tab or switching to the other field).

Now, let's modify our app with the necessary code for handling username validation.
```
// /static/app/app.js
App.User = DS.Model.extend({
  username: DS.attr('string'),
  firstname: DS.attr('string'),
  lastname: DS.attr('string')
});

App.UserRoute = Ember.Route.extend({
  model: function() {
    return this.store.findRecord();
  }
});

App.UsernameValidatorService = Ember.Service.extend({

  validateUsername: function(username) {
    var store = this.get('store');
    return store.findRecord('user', username)
   }
});

App.UsernameInputComponent = Ember.Component.extend({

  usernameValidator: Ember.inject.service(),

  focusOut: function() {
    var username = this.get('username')
    this.get('usernameValidator').validateUsername(username)
    .then(function(username) {
      console.log('username ', username, ' exists');
      $('#usernameInputError #error').remove();
    })
    .catch(function(error) {
      $('#usernameInputError').append('<div id="error">Sorry, the username does not exist.</div>');
    });
    var parent = this.get('parentView');
    parent.set('username', username);
  }
});
```

First, we create a User data store which will keep track of a user's information. This allows us to query the server for a user. As before, we specify a model on its route so that way Ember-Data knows what to do when its route is hit. 

Next, we create our username validation service. It's job is very simple: look in our user store (which will query our server API if necessary) and see if the user exists. 

Lastly, we have our username-input component. It's `focusOut` method is called when the user loses focus of the input element (for example, they select elsewhere or go to the next form input. When they focusOut, we want to get the username and call our validation service. If there is an error, we append the error. Our final step is setting the username variable for the parent component. Usually a parent component wouldn't have access to the child component's data unless the system implements an event bus or action loop, but we went ahead and bent the rules for this situation.

Now that we have username validation taken care of, let's handle comment submission. Remember, we need to handle checking the username on comment submit also. We'll add the following functions to our app.js:
```
// /static/app/app.js

App.PicController = Ember.Controller.extend({

  actions: {
    commentSubmit: function(username, message) {
      var picid = this.get('model').get('id');
      var store = this.store;
      this.get('usernameValidator').validateUsername(username)
      .then(function(username) {
        $('#usernameInputError #error').remove();
        var comment = store.createRecord('comment', {
          message: message,
          username: username.get('id'),
          date: moment().format("YYYY-MM-DDTHH:mm:ss"),
        });
        var pic = store.findRecord('pic', picid)
          .then(function (pic) {
            var picurl = pic.get('picurl');
            comment.set('pic', pic);
            comment.save();
          });
      })
      .catch(function(error) {
        console.log(error);
        $('#usernameInputError').append('<div id="error">Your comment cannot be submitted as you did not provide a valid username.</div>');
        return;
      });
    }
  }
});
App.Comment = DS.Model.extend({
  pic: DS.belongsTo('pic'),
  message: DS.attr('string'),
  username: DS.attr('string'),
  date: DS.attr()
});

App.CommentRoute = Ember.Route.extend({
  model: function() {
    return this.store.findRecord();
  }
});
```
Here we create our comment data store. It has a reference to the picture, the message, username and date. We also have the ability to find comments using the CommentRoute. 

However, the most important part is the PicController. This controller has an action that we linked to in our template called `commentSubmit`. Here is where we get the necessary information to save the picture (such as querying for the current pic model). Notice the promises used here to guarantee pic will be set before calling save on the comment model.


#### What to submit
If part 3 is completed (for extra credit), your group should specify so in the README.md so that it can be graded. To receive full credit, you should have the features outlined and stepped through implemented (dynamic routing, favorites, comments, caption). 

Please note that unlike part 2, this system is not meant to be "real time". In order for new content to be displayed, the user either needs to reload the page or have the Ember data store refresh via it issuing a request for another page action.

### Wrap-up
As you can hopefully see, Ember provides a nice abstraction for many of the features needed. One can imagine that if we were to expand the scope of our application to include more models, routes and functionality, we could maintain a consistent set of best practices for a large team that minimizes errors and code functionality duplication. While the boilerplate may seem tedious for a small application like this, it is necessary when working on larger proejcts. 


## Deliverables
Make sure that you have all of the following present to receive full credit:
* Dynamic data binding for caption and favorites at the `/pic?id=picid` url, with JS code located in the files `/static/js/caption.js` and `/static/js/favorites.js`, respectively.
* API routes specified in part 2, including error handling
* README.md with specification as to whether the extra credit was complete, your team's URLs and any other notes and how many late days have been used
* Database loaded with given data (from projects 1 and 2), alongside the given SQL load data file for favorites and comments. Extra information should not be present in the database.

Your code should be hosted within the `/var/www/html/group##/pa3` sub-directory. **Commit your code to Github and do not modify your Github repository or server code after the due date. Doing so is considered an Honor Code violation.** Also note that if your server is down when we go to grade it, your group will receive a **10% deduction** on the project. If your server is having issues with staying available (such as those caused by MySQL connection errors), it is your group's responsibility to keep it running.

*Please note that the maximum grade for PA3, including extra credit, will be 100%. The extra credit will benefit your grade if you lost points elsewhere on the project.*

Your team's ports for pa3 should be your ports from pa2+200. You **should not** use your databased from either pa1 or pa2. Instead, use the database `group##pa3`.
