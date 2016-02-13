# AngularJS
####[Documentation](https://docs.angularjs.org/api)

### live.html
In our <head> tag we load angular, angular-route, and the javascript file we are going to write (app.js). Angular-route allows us to use routing with angular apps. We could essentially have our whole app live at one url but that is not what we want here. The only other new concepts in this file are the ng-app="MyApp" and the  ng-view that we have in our <div> tag. All ng-app="MyApp" is saying is that this is an angular application and it is called "MyApp". Then we have the ng-view. This is where our views get loaded into. More on this when we look at the actual routing.
```
<!-- views/live.html or perhaps templates/live.html -->
<!doctype html>
<html ng-app="MyApp">
    <head>
        <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.4.5/angular.min.js"></script>
        <script src="//ajax.googleapis.com/ajax/libs/angularjs/1.2.16/angular-route.min.js"></script>
        <script type="text/javascript" src="/static/js/app/app.js"></script>
        <title>EECS485</title>
    </head>
    <body>
        <div ng-view></div>
    </body>
</html>
```


### app.js
We will have only 1 module in this app becuase it is small. We will call it "MyApp". This corresponds to the "MyApp" you had in live.html. When we instantiate our module, we have to list its dependencies. We only have 1 and it is "ngRoute". This is the angular-route that we loaded in our live.html.

The config method in MyApp is ran before anything else. That is where we will declare our routes. We have 1 route and when someone goes to that route we want to load the pic.html template. Everything in '/static/pa3/pic.html' will get loaded into that <div ng-view> tag in live.html. Note that this pic.html is different than any ones in previous projects. We also state that if we go to any other route that is not '/pic/:picid', then redirect them to '/pic/sports_s1'. You can redirect them to any picture, or not redirect them at all.

The controller is where the data-binding happens. We list our dependencies in the function that we pass to MyApp.conroller. We use $scope to bind data to our html, we use $routeParams so that we can access the picid in the route '/pic/:picid', and we use AppService to make our network requests (XHR requests)(AJAX requests)(requests to our server). Anything we add to the $scope variable will be available in our view (our html). Last thing to note is the init() function. This is not an AngularJS thing. We create this function and call it at the bottom of the function we pass to MyApp.controller becuase we want that function to run when the controller is instantiated. The controller is instantiated when the view is loaded.

We declare a sevice called 'AppService' that we use in our controller. AppService is an interface we create to make XHR requests. We could just make all of the requests in our controller but that would get messy. Notice that we provide a public  api to whoever wants to use this service (in this case our controller). This api currently has 3 methods. We call these methods in our controller.
```
// app.js

var MyApp = angular.module('MyApp', ['ngRoute']);

MyApp.config(function($routeProvider) {
    $routeProvider.when('/pic/:picid', {templateUrl: '/static/pa3/pic.html', controller: 'AppController'})
                  .otherwise({redirectTo: '/pic/sports_s1'});

});

MyApp.controller('AppController', function($scope, $routeParams, AppService) {
    var init = function() {
        var promise = AppService.getPic($routeParams.picid);

        promise.then(function(data) {
            var favorites = data.relationships.favorites.data;
            getFavorites(favorites);
            $scope.nextpicid = data.attributes.nextpicid;
            $scope.prevpicid = data.attributes.prevpicid;
            $scope.picid = $routeParams.picid;
            $scope.picurl = data.attributes.picurl;
            $scope.caption = data.attributes.caption;
        }, function(error) {
            console.error(error);
        });

    };

    function getFavorites(favorites) {
        $scope.favorites = [];
        for (fav in favorites) {
            AppService.getFavorite(fav.id).then(function(data) {
                $scope.favorites.push(data.attributes);
            }, function(error) {
                console.error(error);
            });
        }
    };

    $scope.submitCaption = function(e, picid, caption) {
        if (e.keyCode === 13) {
            AppService.setCaption(picid, caption);
        }
    };

    init();
});

MyApp.service('AppService', function($http) {
    function setCaption(picid, caption) {
        var promise = $http({
            url: 'http://eecs485-10.eecs.umich.edu:5990/lcahwk/pa3/api/v1/caption',
            method: 'POST',
            data: {id: picid, caption: caption}
        }).then(function (response) {
            return response.data;
        });
        return promise;
    };

    function getPic(picid) {
        var promise = $http({
            url: 'http://eecs485-10.eecs.umich.edu:5990/lcahwk/pa3/api/v1/pics/' + picid,
            method: 'GET'
        }).then(function (response) {
            return response.data.data;
        });
        return promise;
    };

    function getFavorite(favid) {
        var promise = $http({
            url: 'http://eecs485-10.eecs.umich.edu:5990/lcahwk/pa3/api/v1/favorites/' + favid,
            method: 'GET'
        }).then(function (response) {
            return response.data.data;
        });
        return promise;
    };

    var publicApi = {
        setCaption:     setCaption,
        getPic:         getPic,
        getFavorite:    getFavorite
    };
    return publicApi;
});
```

### pic.html
This is a template file that we created. It gets loaded into that <div> tag that we had in live.html when we go to the route '/pic/:picid' where picid is any ':picid'. Remember how we added things to our $scope variable in our controller? This is where we use those variables. We can access those variables using {{variable}}.

Notice how there are attributes in our html that start with 'ng-'. These are attributes only angular understands. You can think of them as decorators. They provide extra functionality to our HTML elements. Like for example, 'ng-keydown' will execute "submitCaption($event, picid, caption)" whenever you type anything into that input field. You can google any of these attributes (actually called directives) and find documentation on them.

Also notice ng-repeat. It loops through the favorites array and prints all of the usernames.
```
<!--   /static/pa3/pic.html -->
<h1>Picture {{picid}}</h1>
<img src="{{picurl}}">

<br>

<a ng-href="#/pic/{{prevpicid}}">Previous</a>
<a ng-href="#/pic/{{nextpicid}}">Next</a>

<br>

<b>Caption: </b><input type="text" ng-model="caption" ng-keydown="submitCaption($event, picid, caption)">

<h3>{{favorites.length}} favorites</h3>
<div class="favorite" ng-repeat="favorite in favorites">
    {{favorite.username}}
</div>
```
