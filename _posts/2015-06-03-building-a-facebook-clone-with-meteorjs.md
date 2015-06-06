---
layout: post
title: Building Facebook with Meteor in an Afternoon
excerpt: "From nothing to facebook in a few hours"
modified: 2013-05-31
tags: [facebook, meteor, tutorial]
---

> *"Nothing is particularly hard if you divide it into small jobs."* – Henry Ford

## Goals

We must first define the features that facebook has that we would like to recreate. 

* A user must be able to sign up and then be able to log in
* A user must have a profile page
* A user must be able to make a post (similar to a blog) and also be flexible enough to post to other peoples "walls"
* A user must be able to make friend requests
* A user must be able to view a "feed" of activity going on with his or her friends.


Things currently not included in this tutorial

* Picture uploader (probably soon to come)
* editing profile details (definitely soon to come)

## Before we begin

I am not a great designer and I don't pretend to be.  However, I have wanted to make this tutorial for a while so I went and found a great [Twitter Bootstrap 3 Facebook template](http://www.bootply.com/render/96266) that I used to create this.  You should download it before you go any further in this tutorial. This tutorial assumes you have meteor installed in your dev environment.  It is also assuming you understand some basics about meteor.

### Structuring the application

In your terminal run `meteor create facebook` then remove the files `facebook.html`, `facebook.js`, and `facebook.css` then make sure you add twitter-bootstrap to your project by running `meteor add twbs:bootstrap` followed by creating a file structure like the one outlined below.  
I use the same basic file structure for all of my meteor apps.

    client/
      css/
      js/
      lib/
      views/
      compatibility/ (optional)
    lib/
    public/
      img/
    private/
    server/
      api/ (optional)
      config/
      helpers/
      lib/
      models/
      publications/

Now to just get it out of the way now add [this](https://github.com/krishamoud/meteor-facebook/blob/master/client/css/facebook.css) css file to `client/css/` **This is the only place where I will mention css**  

Great!  Now that we got that out of the way we can get to the fun stuff.  

### Account Creation

Every website needs a way to register.  This used to be hard but meteor makes it easy.
I used the `accounts-password` package built by MDG to make the login.  
In your terminal run `meteor add accounts-password` or go to `.meteor/packages` and add to the file
`accounts-password`  

Now we have all the methods necessary to build a register/login system.  
In `client/views/` we need to make a `main.html` file and a `main.js`.  
`main.html` should look like this

{% highlight html %}
<head>
    <meta http-equiv="content-type" content="text/html; charset=UTF-8">
    <meta charset="utf-8">
    <title>Facebook Theme Demo</title>
    <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1">
</head>
{% endhighlight %}

Nothing crazy.  

Next we need to get a way to render templates based on route.  I used [iron-router](https://github.com/iron-meteor/iron-router) for this project although recently [flow-router](https://github.com/meteorhacks/flow-router) has gained some popularity.  
Add it the same way you did the `accounts-password` package.  

`meteor add iron:router`  

Now that we have that we are going to route `login` and `register` to the correct templates.  
I put my router code in the `lib/` folder because iron-router allows for server side routing and if you run it in the `client/` folder then you lose the server side functionality.  So create a file `lib/router.js` then add the following code.

{% highlight javascript %}
Router.route('/register',{
    template:"register"
});

Router.route('/login',{
    template:"login"
})
{% endhighlight %}

All this is doing is saying "When a user hits the route '/login' render the 'login' template" and same for the registration page.

Now we have to actually create these templates.

I am very partial to having one folder per template which (in my opinion) should have exactly two files in it.

* 1: the html file
* 2: the javascript file

First we build the registration page.

First create a folder `client/views/register`  

Then create the files `client/views/register/register.html`, `client/views/register/register.js` 


{% highlight html %}
<template name="register">
    <div class="container">
        <div id="signupbox" style="margin-top:50px" class="mainbox col-md-6 col-md-offset-3 col-sm-8 col-sm-offset-2">
            <div class="panel panel-info">
                <div class="panel-heading">
                    <div class="panel-title">Sign Up</div>
                    <div style="float:right; font-size: 85%; position: relative; top:-10px"><a id="signinlink" href="/login">Sign In</a></div>
                </div>
                <div class="panel-body">
                    <form id="signupform" class="form-horizontal" role="form">
                        <div id="signupalert" style="display:none" class="alert alert-danger">
                            <p>Error:</p>
                            <span></span>
                        </div>
                        <div class="form-group">
                            <label for="email" class="col-md-3 control-label">Email</label>
                            <div class="col-md-9">
                                <input type="text" class="form-control" name="email" placeholder="Email Address">
                            </div>
                        </div>
                        <div class="form-group">
                            <label for="firstname" class="col-md-3 control-label">First Name</label>
                            <div class="col-md-9">
                                <input type="text" class="form-control" name="firstname" placeholder="First Name">
                            </div>
                        </div>
                        <div class="form-group">
                            <label for="lastname" class="col-md-3 control-label">Last Name</label>
                            <div class="col-md-9">
                                <input type="text" class="form-control" name="lastname" placeholder="Last Name">
                            </div>
                        </div>
                        <div class="form-group">
                            <label for="password" class="col-md-3 control-label">Password</label>
                            <div class="col-md-9">
                                <input type="password" class="form-control" name="passwd" placeholder="Password">
                            </div>
                        </div>
                        <div class="form-group">
                            <!-- Button -->
                            <div class="col-md-offset-3 col-md-9">
                                <button id="btn-signup" type="submit" class="btn btn-info"><i class="icon-hand-right"></i> &nbsp; Sign Up</button>
                                <span style="margin-left:8px;">or</span>
                            </div>
                        </div>
                        <div style="border-top: 1px solid #999; padding-top:20px" class="form-group">
                            <div class="col-md-offset-3 col-md-9">
                                <button id="btn-fbsignup" type="button" class="btn btn-primary"><i class="icon-facebook"></i>   Sign Up with Facebook</button>
                            </div>
                        </div>
                    </form>
                </div>
            </div>
        </div>
    </div>
</template>
{% endhighlight %}

This again is nothing crazy.  

The javascript that correlates to this page.

{% highlight javascript %}
// ----------------------------------------------------------------------------
// @Date: 
// @author: 
// @description: This is where the user will register for the site
// ----------------------------------------------------------------------------

// ----------------------------------------------------------------------------
// Template Event Map
// ----------------------------------------------------------------------------
Template.register.events({
    'submit #signupform':function(e){
        e.preventDefault();
        var email = $("input[name='email']").val();
        var firstname = $("input[name='firstname']").val();
        var lastname = $("input[name='lastname']").val();
        var password = $("input[name='passwd']").val();
        var username = firstname + lastname;
        try {
            if(!email.length) throw new Meteor.Error("need email", "You must have an email");
            if(!firstname.length) throw new Meteor.Error("need name", "You must input your first name");
            if(!lastname.length) throw new Meteor.Error("need lastname", "You must input your last name");
            if(password.length < 6) throw new Meteor.Error("password length", "Your password must be at least 6 characters in length");
            Accounts.createUser({username: username, email: email, password: password,
                                    profile: {firstname: firstname, lastname: lastname}}, function(err, id){
                if(!err) {
                    Router.go("/")
                }
            });
        } catch (e) {
            console.log(e);
        }
    }
})
{% endhighlight %}
This isn't crazy either but essentially what is happening is the template `register` is listening for a submit event from the signup form.

Inside the function we first get the users information (i.e. email, firstname, etc) then create a username that is just the first and last name put together (this is for demonstration purposes only).  

After that we go into a try/catch statement that does some basic validation.  A user must have an email, firstname, lastname, and their password must be greter than 6 characters long. 

After it passes all of these checks it calls a method from the `accounts` package called `createUser` which takes alll that info and creates a user out of it.  It will either return an error as the first argument or the newly created userId as the second argument.  

If there are no errors then iron-router will route us to the `"/"` route which we have not yet defined.  

This page is now done.  

### Login page

A user can now create an account.  Now they have to be able to log in.  

Create a folder `client/views/login` and again add two files to it. `login.html` and `login.js`

`login.html` will also not be a big deal.

{% highlight html %}
<template name="login">
    <div class="container">
        <div id="loginbox" style="margin-top:50px;" class="mainbox col-md-6 col-md-offset-3 col-sm-8 col-sm-offset-2">
            <div class="panel panel-info">
                <div class="panel-heading">
                    <div class="panel-title">Sign In</div>
                    <div style="float:right; font-size: 80%; position: relative; top:-10px"><a href="#">Forgot password?</a></div>
                </div>
                <div style="padding-top:30px" class="panel-body">
                    <div style="display:none" id="login-alert" class="alert alert-danger col-sm-12"></div>
                    <form id="loginform" class="form-horizontal">
                        <div style="margin-bottom: 25px" class="input-group">
                            <span class="input-group-addon"><i class="glyphicon glyphicon-user"></i></span>
                            <input id="login-username" type="text" class="form-control" name="username" value="" placeholder="username or email">
                        </div>
                        <div style="margin-bottom: 25px" class="input-group">
                            <span class="input-group-addon"><i class="glyphicon glyphicon-lock"></i></span>
                            <input id="login-password" type="password" class="form-control" name="password" placeholder="password">
                        </div>
                        <div class="input-group">
                            <div class="checkbox">
                                <label>
                                    <input id="login-remember" type="checkbox" name="remember" value="1"> Remember me
                                </label>
                            </div>
                        </div>
                        <div style="margin-top:10px" class="form-group">
                            <!-- Button -->

                            <div class="col-sm-12 controls">
                                <button id="btn-login" href="#" type='submit' class="btn btn-success">Login  </button>
                                <a id="btn-fblogin" href="#" class="btn btn-primary">Login with Facebook</a>

                            </div>
                        </div>
                        <div class="form-group">
                            <div class="col-md-12 control">
                                <div style="border-top: 1px solid#888; padding-top:15px; font-size:85%">
                                    Don't have an account!
                                    <a href="/register">
                                        Sign Up Here
                                    </a>
                                </div>
                            </div>
                        </div>
                    </form>
                </div>
            </div>
        </div>
    </div>
</template>
{% endhighlight %}

This is just a bootstrap page with a form on it.

`login.js` will be just as boring.
{% highlight javascript %}
// ----------------------------------------------------------------------------
// @Date: 
// @author: 
// @description: This is where the user will login
// ----------------------------------------------------------------------------

// ----------------------------------------------------------------------------
// Template Event Map
// ----------------------------------------------------------------------------
Template.login.events({
    'submit #loginform':function(e){
        e.preventDefault();
        var username = $('input[name="username"]').val();
        var password = $('input[name="password"]').val();
        Meteor.loginWithPassword(username,password, function(err){
            if(!err) {
                Router.go("/")
            }
        })
    }
})
{% endhighlight %}
The template `login` is listening for a submit even from the `#login` form finding the username, password then calling a meteor method called `loginWithPassword` which takes two arguments, `username` and `password` then has a callback function that's called with no arguments on success or a single error argument on failure.

Great!  We now have fully functioning login and registration for our soon to be facebook clone.


### Random Users (optional step)

Now that we have a working login/registration setup, lets not use it.  I used [randomuser.me](https://randomuser.me/) to create a bunch of profiles so I could test out the network stuff that will come later.

If you want to do the same here is what I did.

First I added `http` to my `.meteor/packages` file so that I could make some request and create a bunch of users.

Next in `server/lib/` I created a file called `startup.js` which then runs a function that runs when the meteor server first starts up.

`server/lib/startup.js`
{% highlight javascript %}
Meteor.startup(function(){
    var users = Meteor.users.find().count();
    if(!users) {
        for(var i = 0; i < 100; i++){
            HTTP.get("http://api.randomuser.me/", function(err,res){
                var fields = res.data.results[0].user;
                delete fields['password'];
                delete fields['salt'];
                delete fields['md5'];
                delete fields['sha1'];
                delete fields['sha256'];
                delete fields['registered'];
                delete fields['dob'];
                delete fields['HETU'];
                delete fields['INSEE'];
                delete fields['phone'];
                delete fields['cell']
                delete fields['version'];
                var user = {
                    username: fields.username,
                    email: fields.email,
                    password:"password",
                    profile: fields
                }
                Accounts.createUser(user);
            })
        }
    }
})
{% endhighlight %}

All this code is doing is checking if there are no users when the server starts.  If there are no users then it goes into a `for` loop making 100 http calls to the api endpoint, the response is a fake user.  I deleted the fields I didn't want or need then created a user object and used the built in meteor method `Accounts.createUser(user)` to create 100 accounts that would have the same data structure. 

Done.

### Profile page

This will be the first somewhat complicated page but we will build it in chunks and it wont be that bad.

First we need a route.  

Add this to your `lib/router.js` file.
{% highlight javascript %}
Router.route('/profile/:username',{
    template:"profileFeed"
});
{% endhighlight %}

This is basically saying that when someone routes to '/profile/anyRandomUsername` it will render the template profileFeed.

We need to think about what a profile will look like.  It will have a navbar, a profile picture, profile details, and a "feed" of "stories" that involve the profile owner.

Create a folder `client/views/profile` and inside of that we are going to have **two** additional folders.  One is going to be the template where we show all of the profile users info and the other will be where we show all of the profile users "stories" or feed items.

The folders that should be made are `client/views/profile/profileDetails` and `client/views/profile/profileFeed` which will both have two files in them each. `client/views/profile/profileFeed/profileFeed.html`, `client/views/profile/profileFeed/profileFeed.js`, `client/views/profile/profileDetails/profileDetails.html` and `client/views/profile/profileDetails/profileDetails.js`

Since we defined `profileFeed` as the template where the router will take us then we should build that first.

Here is the html that I came up with 

{% highlight html %}
<template name="profileFeed">
    <div class="wrapper">
        <div class="box">
            <div class="row row-offcanvas row-offcanvas-left">
                <!-- sidebar -->
                {% raw %}
                {{>sidebar}}
                {% endraw %}
                <!-- /sidebar -->
                <!-- main right col -->
                <div class="column col-sm-10 col-xs-11" id="main">
                    <!-- top nav -->
                    {% raw %}
                    {{>topnav}}
                    {% endraw %}
                    <!-- /top nav -->
                    <div class="padding">
                        <div class="full col-sm-9">
                            <!-- content -->
                            <div class="row">
                                {% raw %}
                                {{>profileDetails}}
                                {% endraw %}
                                <!-- main col right -->
                                <div class="col-sm-7">

                                    <div class="well">
                                        <form class="form-horizontal" role="form">
                                            <h4>What's New</h4>
                                            <div class="form-group" style="padding:14px;">
                                                <textarea class="form-control" name="new-post" placeholder="Post a status!"></textarea>
                                            </div>
                                            <button class="btn btn-primary pull-right new-post" type="button">Post</button>
                                            <ul class="list-inline">
                                                <li><a href=""><i class="glyphicon glyphicon-upload"></i></a></li>
                                                <li><a href=""><i class="glyphicon glyphicon-camera"></i></a></li>
                                                <li><a href=""><i class="glyphicon glyphicon-map-marker"></i></a></li>
                                            </ul>
                                        </form>
                                    </div>
                                </div>

                            </div>
                        </div>
                    </div>
                </div>
                <!-- /main -->
            </div>
        </div>
    </div>
    {% raw %}
    {{>modal}}
    {% endraw %}
</template>
{% endhighlight %}

We have some templates in there that we have not defined yet, namely `sidebar`, `topnav`, `modal` and `profileDetails`.  Lets define those now.

To avoid building up a cluttered `client/views/` folder I like to lump some of the more reusable templates together into a folder called `client/views/common` which is where I put things like `sidbar`, `topnav` and `modal` so to follow along you should create this folder as well.

Inside `client/views/common` we are going to create **three** folders.  `client/views/common/modal`, `client/views/common/sidebar`, and `client/views/common/topnav` which will each have two respective files in each the same as we have done with every other folder/file that we have defined.

We should make those now so meteor doesn't yell at us.

### Sidebar
`client/views/common/sidebar.html`
{% highlight html %}
<template name="sidebar">
    <!-- sidebar -->
    <div class="column col-sm-2 col-xs-1 sidebar-offcanvas" id="sidebar">

        <ul class="nav">
            <li><a href="#" data-toggle="offcanvas" class="visible-xs text-center"><i class="glyphicon glyphicon-chevron-right"></i></a></li>
        </ul>

        <ul class="nav hidden-xs" id="lg-menu">
            <li class="active"><a href="#featured"><i class="glyphicon glyphicon-list-alt"></i> Featured</a></li>
            <li><a href="#stories"><i class="glyphicon glyphicon-list"></i> Stories</a></li>
            <li><a href="#"><i class="glyphicon glyphicon-paperclip"></i> Saved</a></li>
            <li><a href="#"><i class="glyphicon glyphicon-refresh"></i> Refresh</a></li>
        </ul>
        <ul class="list-unstyled hidden-xs" id="sidebar-footer">
            <li>
              <a href="http://usebootstrap.com/theme/facebook"><h3>Bootstrap</h3> <i class="glyphicon glyphicon-heart-empty"></i> Bootply</a>
            </li>
        </ul>

        <!-- tiny only nav-->
      <ul class="nav visible-xs" id="xs-menu">
            <li><a href="#featured" class="text-center"><i class="glyphicon glyphicon-list-alt"></i></a></li>
            <li><a href="#stories" class="text-center"><i class="glyphicon glyphicon-list"></i></a></li>
            <li><a href="#" class="text-center"><i class="glyphicon glyphicon-paperclip"></i></a></li>
            <li><a href="#" class="text-center"><i class="glyphicon glyphicon-refresh"></i></a></li>
        </ul>

    </div>
    <!-- /sidebar -->    
</template>
{% endhighlight %}
This is directly stolen from the bootstrap theme.

### Top Nav Bar
`client/views/common/topnav.html`
{% highlight html %}
<template name="topnav">
    <div class="navbar navbar-blue navbar-static-top">
        <div class="navbar-header">
            <button class="navbar-toggle" type="button" data-toggle="collapse" data-target=".navbar-collapse">
                <span class="sr-only">Toggle</span>
                <span class="icon-bar"></span>
                <span class="icon-bar"></span>
                <span class="icon-bar"></span>
            </button>
            <a href="/" class="navbar-brand logo">b</a>
        </div>
        <nav class="collapse navbar-collapse" role="navigation">
            <form class="navbar-form navbar-left">
                <div class="input-group input-group-sm" style="max-width:360px;">
                    <input class="form-control" placeholder="Search" name="srch-term" id="srch-term" type="text">
                    <div class="input-group-btn">
                        <button class="btn btn-default" type="submit"><i class="glyphicon glyphicon-search"></i></button>
                    </div>
                </div>
            </form>
            <ul class="nav navbar-nav">
                <li>
                    <a href="/profile/{% raw %}{{currentUser.username}}{% endraw %}"><img src="{% raw %} {{currentUser.profile.picture.thumbnail}}{% endraw %}" height="28px" width="28px" alt="">{% raw %} {{fullname currentUser}}{% endraw %}</a>
                </li>
                <li>
                    <a href="/"><i class="glyphicon glyphicon-home"></i> Home</a>
                </li>
                <li>
                    <a href="#postModal" role="button" data-toggle="modal"><i class="glyphicon glyphicon-plus"></i> Post</a>
                </li>
                <li>
                    <a href="/notifications"><i class="glyphicon glyphicon-envelope"></i> <span class="badge">{% raw %} {{friendRequestCount}} {% endraw %}</span> Friend Requests</a>
                </li>
            </ul>
            <ul class="nav navbar-nav navbar-right">
                <li class="dropdown">
                    <a href="#" class="dropdown-toggle" data-toggle="dropdown"><i class="glyphicon glyphicon-user"></i></a>
                    <ul class="dropdown-menu">
                        <li class="logout"><a href="#">Logout</a></li>
                    </ul>
                </li>
            </ul>
        </nav>
    </div>
</template>
{% endhighlight %}

If you didn't notice, we have **two** different helper methods in here as well as some other handlebars stuff going on.  I'll tackle them one at a time.

`<a href="/profile/{% raw %}{{currentUser.username}}{% endraw %}">` is doing nothing more than linking to the **currently signed in** users profile.  `currentUser` is a method that meteor has built for us in spacebars (meteor flavored handlebars) that gives us access to the current signed in user document.  

`<img src="{% raw %} {{currentUser.profile.picture.thumbnail}}{% endraw %}" height="28px" width="28px" alt="">` is doing something similar.

The two methods of interest are {% raw %}{{fullname currentUser}}{% endraw %} and {% raw %}{{friendRequestCount}}{% endraw %}.  Let's define those now.

{% highlight javascript %}
// ----------------------------------------------------------------------------
// @Date: 
// @author: 
// @description: This is where top nav stuff happens
// ----------------------------------------------------------------------------

// ----------------------------------------------------------------------------
// Template Event Map
// ----------------------------------------------------------------------------
Template.topnav.events({
    'click .logout':function(){
        Meteor.logout(function(err){
            if(!err) {
                Router.go("/");
            }
        })
    }
})


// ----------------------------------------------------------------------------
// Template Helper Map
// ----------------------------------------------------------------------------
Template.topnav.helpers({
    fullname:function(user){
        return user ? user.profile.name.first + " " + user.profile.name.last : null;
    },
    friendRequestCount:function(){
        return 0;
    }
}
{% endhighlight %}

First you'll notice that I created an event for when the logout class is clicked.  It is straightforward.

Next in the helpers we have some very basic methods.  `fullname` takes the current user as an argument and returns the first and last name combined.  You can do this in handlebars but I didn't want to keep typing the dot notation.  

Second, we have `friendRequestCount` since we have no infrastructure for friends right now it will just return 0.  This will be modified soon.

### Modal
The last common template we need to define is the modal.  It looks like this.

{% highlight html %}
<template name="modal">
    <!--post modal-->
    <div id="postModal" class="modal fade" tabindex="-1" role="dialog" aria-hidden="true">
      <div class="modal-dialog">
      <div class="modal-content">
          <div class="modal-header">
              <button type="button" class="close" data-dismiss="modal" aria-hidden="true">�</button>
                Update Status
          </div>
          <div class="modal-body">
              <form class="form center-block">
                <div class="form-group">
                  <textarea class="form-control input-lg" autofocus="" name="new-post" placeholder="What do you want to share?"></textarea>
                </div>
              </form>
          </div>
          <div class="modal-footer">
              <div>
              <button class="btn btn-primary btn-sm new-post" data-dismiss="modal" aria-hidden="true">Post</button>
                <ul class="pull-left list-inline"><li><a href=""><i class="glyphicon glyphicon-upload"></i></a></li><li><a href=""><i class="glyphicon glyphicon-camera"></i></a></li><li><a href=""><i class="glyphicon glyphicon-map-marker"></i></a></li></ul>
              </div>
          </div>
      </div>
      </div>
    </div>
</template>
{% endhighlight %}

Vanilla bootstrap.

### Profile Details

The last template we need to make is the `profileDetails` template so that our `profileFeed` template will render.

`client/views/profile/profileDetails/profileDetails.html`
{% highlight html %}
<template name="profileDetails">
    <!-- main col left -->
    <div class="col-sm-5">

         <div class="panel panel-default">
           <div class="panel-thumbnail"><img src="{{profilePicture}}" class="img-responsive"></div>
           <div class="panel-body">
             <p class="lead">{% raw %}{{fullname}}{% endraw %}</p>
             <p>{% raw %}{{friendCount}}{% endraw %} Friends</p>
             <p><button class="btn btn-primary add-friend">Add Friend</button></p>
             <p>
                {% raw %}
                {{#each newFriends}}
                    <a href="/profile/{{this.profile.username}}"><img src="{{this.profile.picture.thumbnail}}" height="28px" width="28px"></a>
                {{/each}}
                {% endraw %}
             </p>
           </div>
         </div>


         <div class="panel panel-default">
           <div class="panel-heading"><h4>About</h4></div>
             <div class="panel-body">
                {% raw %}
                {{about}}
                {% endraw %}
             </div>
         </div>



         <div class="panel panel-default">
           <div class="panel-heading"><h4>Some Header</h4></div>
           <div class="panel-body">
               Fill this with whatever you want
            </div>
         </div>
     </div>
</template>
{% endhighlight %}

Again more bootstrap but now we have some template helpers in there but none of them are using the `currentUser`.  Why not? Because it isn't gauranteed that the profile we are on is the same as the person who is currently signed in.  Let's define those helpers now!

{% highlight javascript %}
Template.profileDetails.helpers({
    fullname:function(){
        var username = Router.current().params.username;
        var user = Meteor.users.findOne({username:username});
        return user ? user.profile.name.first + " " + user.profile.name.last : null;
    },
    profilePicture:function() {
        var username = Router.current().params.username;
        var user = Meteor.users.findOne({username:username});
        return user ? user.profile.picture.large : null;
    },
    friendCount:function(){
        return 0;
    },
    newFriends:function(){
        return [];
    },
    about:function(){
        var username = Router.current().params.username;
        var user = Meteor.users.findOne({username:username});
                return user ? user.profile.location.street + " " +
                              user.profile.location.city + ", " + user.profile.location.state + " " +   
                              user.profile.location.zip : "";
    },
    storyCount:function(){
        return 0;
    }
})


Template.profileDetails.events({
    'click .add-friend':function(){
        // soon!
    }
})
{% endhighlight %}

Great!  Now we have the helpers defined.  The only problem is that our `Meteor.users.findOne()` calls are going to return undefined because we have not *published* the needed user documents.

(Stories and friends return empty because we haven't built that yet)

### Publishing User Documents

Publishing to the client is a core feature of meteor and we need to know how to do it in order to do a lot of things in meteor.  We need profile user information but we don't want to give *too* much info.

Publications happen on the server so let's make it happen.

In `server/publications/` we should create a file called `userPublications.js`
This file will get pretty long in the future but for right now we only need one publication and it's easy.
{% highlight javascript %}
Meteor.publish("userData", function(username){
    return Meteor.users.find({username:username});
})
{% endhighlight %}
You can restrict fields if you want but I was lazy and didn't.

Now that we have the publication we now have to *subscribe* to it.
This will happen on the client side.

### Subscribing to User Documents
In `client/views/profile/profileDetails/profileDetails.js` add the following code.
{% highlight javascript %}
Template.profileDetails.onCreated(function(){
    var self = this;
    var username = Router.current().params.username;
    self.autorun(function(){
        username = Router.current().params.username;
        self.subscribe("userData", username, {
            onReady:function(){
                var user = Meteor.users.findOne({username: username});
                if(!user) {
                    Router.go("/");
                }
            }
        });
    })

})
{% endhighlight %}

What's happening is actually pretty straightforward.  When the template is created an `autorun` function is called once to subscribe on the current users profile.  Anytime the username in the router *changes* it is going to resubscribe because we are going to need the new users profile info.  If the user doesn't exist it will route us back to "/" which again we haven't defined yet.

Great!  Now all of our helper functions work!

### Creating Stories (status/post to friends wall)
Our site is really coming along.  Believe it or not but this clone is almost done.

Stories are actually incredibly easy to create.  

#### Step 1: Define the Mongo Collection
Since we want the collection to exist on both the client and server we need to define it in a neutral place.  I use the `lib/` folder.

First create a file `lib/collections.js` and add one line to it.
`Stories = new Mongo.Collection("stories");`
Now we can create stories!

The only problem is if we remove the `insecure` package that comes in every meteor application, anyone can CRUD our documents.  So first and formost, remove the `insecure` package from `.meteor/packages` and then we can use the built in Meteor `allow/deny` rules that look like this.
{% highlight javascript %}
Stories.allow({
    insert:function(userId, doc) {
        return !!userId;
    },
    update:function(userId, doc) {
        return !!userId;
    }
})
{% endhighlight %} 

Any time a document is trying to be created, destroyed, or updated it will check these methods to see if it is allowed to do so.  

These are super basic restrictions which you can edit to your hearts content but I just want to make sure that the user is signed in in order to create or update a `Story` document.

I prefer to work in booleans so I use double-not notation to convert the type to booleans you can however simply do `return userId` and it will work the same (because undefined is a falsey value).

#### Step 2: Creating a story from the profile page
We have done nothing on `client/views/profile/profileFeed/profileFeed.js` so now we are going to!

If we think about the way facebook works.  You can either post a status from your own page or if you are on someone else's page you can post to their page.  We are going to implement that now.

{% highlight javascript %}
Template.profileFeed.events({
    'click .new-post':function(e){
        e.preventDefault();
        var profileUser = Meteor.users.findOne({username:Router.current().params.username});
        var currentUser = Meteor.user();
        var story = $('textarea[name="new-post"]').val();
        if(story.length) {
            Stories.insert({
                createdBy: currentUser._id, // the Meteor.userId()
                createdFor: profileUser._id, // the owner of the profile
                storyImage: null, // for the future we can add images in our story
                storyText: story, // the text that is the story
                creatorName: currentUser.profile.name.first + " " + currentUser.profile.name.last, // the creator
                creatorUsername: currentUser.profile.username, // so we can link to the creators profile
                creatorThumbnail: currentUser.profile.picture.thumbnail, // so we can have a picture in the story
                createdForName: profileUser.profile.name.first + " " + profileUser.profile.name.last, // the person recieving the post
                createdForUsername: profileUser.profile.username, // so we can link to the recievers profile
                createdForThumbnail: profileUser.profile.picture.thumbnail, // so we can see the recievers picture
                likes: [], // so we can see who's liked the post
                createdAt: new Date(), // good practice IMO
                comments: [] // comment array
            });
            $('textarea[name="new-post"]').val(""); // reset the text box when done
        }

    }

})

Template.profileFeed.helpers({
    statusPlaceholder:function(){
        var profileUser = Meteor.users.findOne({username:Router.current().params.username});
        if(profileUser && profileUser._id === Meteor.userId()){
            return "Update your status";
        } else {
            return "Post to their wall!";
        }
    }
})
{% endhighlight %}

Ok, there are **two** main things happening here.  I'll go over the `statusPlaceholder` first.  

All `statusPlaceholder` is doing is telling whether or not you are on your own page or of a different users' page.  Then you should replace your current `textarea` on `profileFeed.html` with 

`<textarea class="form-control" name="new-post" placeholder="{{statusPlaceholder}}"></textarea>` 

All that's happening is you are being returned the correct string from the function.

The `click .newPost` is much more interesting.  What's happening first is we get the profile owner's _id, then we get the current logged in users' _id that way we can know if he or she is posting to someone else' wall or their own.  

The fields have all been commented on why they are in the document.

You can now create stories!
#### Showing the stories

There are three pretty simple steps to accomplish this.

Step 1: create a publication

In `server/publications/userPublications.js` add the following lines.

{% highlight javascript %}
Meteor.publish("profileStories", function(username){
    var user = Meteor.users.findOne({username:username}, {fields: {_id:1}});
    return Stories.find({createdFor: user._id});
})
{% endhighlight %}

This is pretty straighforward.  It's finding the user based on the username because we are going to have access to the username in the url.  After that it finds the stories *created for* that user and returns them.

Step 2: subscribe on the client side.

In `client/views/profile/profileFeed/profileFeed.js` add the following code.

{% highlight javascript %}
Template.profileFeed.onCreated(function(){
    var self = this;
    Tracker.autorun(function(){
        var username = Router.current().params.username;
        self.subscribe("profileStories", username);
    })

})
{% endhighlight %}

This is a block that happens once the `profileFeed.html` template is created.  It will subscribe to `"profileStories"` with the username in the router as the first argument.  I put the username inside the autorun function so that if the user visits another profile page immediately after landing on the profile page the first time.  If it is not inside the autorun function then the subscription will never resubscribe and you will have the incorrect documents for the second profile page.

Step 3: render the stories in html

Inside `client/views/profile/profileFeed/profileFeed.js` we now need to return the correct `stories`

Add this code to `profileFeed.js` inside the `Template.profileFeed.helpers`
{% highlight javascript %}
stories:function(){
        var profileUser = Meteor.users.findOne({username:Router.current().params.username}, {fields: {_id:1}});
        return profileUser ? Stories.find({createdFor: profileUser._id}, {sort: {createdAt:-1}, limit: 10}) : [];
    }
{% endhighlight %}

This is getting the profile user first, then if the user exists (more a thing for the template to have time to retirve the user doc) it returns the stories that were `createdFor` the profile user.  It orders them from most recent to least recent and limits it to 10 documents.


**to be continued**
