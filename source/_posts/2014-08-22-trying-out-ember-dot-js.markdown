---
layout: post
title: "Trying out ember.js"
date: 2014-08-22 21:01:06 +0200
comments: true
categories: 
---


Trying out ember js

I wanted to build an editor for one of my sites where users can edit texts, add, sort, and edit pictures, add embedded content and I did not wanted to use a WYSIWYG editor because this would be user created content and currently they only can input markdown or upload one picture. 

### The tutorial video is good

My first video which was my introduction to ember is really good. You can find it on the emberjs official  site among the other guides but here it is: 

<iframe width="560" height="315" src="//www.youtube.com/embed/1QHrlFlaXdI" frameborder="0" allowfullscreen></iframe>

I found this answers all my immdiate questions about ember.js, was not too slow so I did not felt that it would be better if it would be written down and because it has some weird things. 

There was some things which were not really immediately obvious, forgive me if there are easier mode to do these. 

### Wrapping the models in Ember objects:

If you have a model array and want that the views automatically change   you change a property on some of the element of the array, then you have to wrap the individual objects. This is easily achieavable like this in your controler's model function:

``` javascript
return [{title: "hello"}, {title: "world"}].map( function(elem){
    return Ember.Object.create(elem);
});

```

### Accessing controller model data

It was not obvious how I can access the actual model of the data but it is fairly simple:

``` javascript
    In your controller you can call:
    this.currentModel

```


### Whats the difference between route and controller


Calling actions with parameters



Rendering partial views based on model prop

