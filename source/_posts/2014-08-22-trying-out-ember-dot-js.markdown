---
layout: post
title: "Trying out ember.js"
date: 2014-08-22 21:01:06 +0200
comments: true
categories: 
draft: true
published: false
---


I wanted to build an editor for one of my sites where users can edit texts, add, sort, and edit pictures, add embedded content and I did not wanted to use a WYSIWYG editor because this would be user created content and currently they only can input markdown or upload one picture.  

### The tutorial video is good

My first video which was my introduction to ember is really good. You can find it on the emberjs official  site among the other guides but here it is: 

<iframe width="100%" height="500px" src="//www.youtube.com/embed/1QHrlFlaXdI" frameborder="0" allowfullscreen></iframe>

I found this answers all my immdiate questions about ember.js, was not too slow so I did not felt that it would be better if it would be written down and because it has some weird things. 

There was some things which were not really immediately obvious, forgive me if there are easier ways to do these. 

### Wrapping the models in Ember objects:

If you have a model array and want that the views automatically refresh as  you change a property on some of the element of the array, then you have to wrap the individual objects. This is easily achieavable like this in your controler's model function:

``` javascript
return [{title: "hello"}, {title: "world"}].map( function(elem){
    return Ember.Object.create(elem);
});

```

### Accessing controller model data

It was not obvious how I can access the actual model of the data but it is fairly simple. In your controller you can use the property `currentModel`:

``` javascript
    this.currentModel

```

### Rendering partial views based on model property

The templates don't  really have ifs. I always think about ifs pollute the view code or not. Ember.js creators decided that it pollutes. If you want to render different template for different models then this is a problem. Fortunately this is easily solvable like this:


{% raw %}
``` javascript

In the controller:

    model: function(){
        return [{ title: "hello"}, {title: "world"}].map( function(elem){
                elem.element_template = "the-name-of-the-template"
        });
    }


In the template:
{{#each}}
    {{partial element_template}}
{{/each}}


<script type="text/x-handlebars" data-template-name="the-name-of-the-template">
        {{title}}
</script>

```
{% endraw %}

### Whats the difference between route and controller

As it seems to me, there are no real differences between a Route and a Controller. Even something's functions called  ArrayController can simply be implemented in a Route. 

### Calling actions with parameters

This is fairly easy, I don't know now why I felt that this needs to be included in my list:

Define actions in your controller:

{% raw %}
``` javascript

    action: { 
        doSomething: function(param,param2){
                alert(param);
                console.log(param2)
        }
    }

```

{% endraw %}

You can pass params to the `action` helper, separated by spaces:

{% raw %}
``` javascript

    {{action doSomething "hello World" "hello console"}}

```
{% endraw %}



