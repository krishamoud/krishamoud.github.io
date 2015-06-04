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
                    {{>topnav}}
                    <!-- /top nav -->
                    <div class="padding">
                        <div class="full col-sm-9">
                            <!-- content -->
                            <div class="row">
                                {{>profileDetails}}
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
    {{>modal}}
</template>
{% endhighlight %}

We have some templates in there that we have not defined yet, namely `sidebar`, `topnav`, `modal` and `profileDetails`.  Lets define those now.

To avoid building up a cluttered `client/views/` folder I like to lump some of the more reusable templates together into a folder called `client/views/common` which is where I put things like `sidbar`, `topnav` and `modal` so to follow along you should create this folder as well.

Inside `client/views/common` we are going to create **three** folders.  `client/views/common/modal`, `client/views/common/sidebar`, and `client/views/common/modal` which will each have two respective files in each the same as we have done with every other folder/file that we have defined.


**to be continued**
