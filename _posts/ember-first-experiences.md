title: Ember - First Experiences
tags:
- Ember
- js
date: 2014-09-12 16:27:05
---

Here starts my baptism of fire with Ember.js, the highs, the lows, and the hacking!

My story is pretty typical to a point. I built a single page web app in native JS and a spattering (well, more of a spilt paint bucket) of jQuery, only to realise after I'd finished that Ember existed. Short of wanting to jump out of the window once I'd realised how much time I might have saved, I decided to see what would actually be involved in converting the app to Ember. 

<!-- more -->

So, first things first I imported the library and started bootstrapping my app. To my surprise, the basics came off without a hitch, understanding how Ember operates seemed like second nature to me. I come from a backend PHP background using highly convention driven opinionated frameworks, so Ember wasn't a huge lift. Whilst those terms (highly contention driven, opinionated) may sound scary, they're some of the main attractions of Ember to me.

Whilst researching Ember, I found a lot of arguments of "Ember forces me to program a certain way", and "what if I want to do things my way, angular let's me do that". Whilst there is some truth in those arguments, the conventions and opinionated principles that Ember builds on are designed to make your life easier, trust me, no one is out to try and make your life harder. 

##Concepts and Conventions

Think about it this way, the Ember team have set out some guidelines on how they consider things should be done. Those decisions have come from a team of experienced and well learned developers, who have all made the same mistakes as you and I in the past, and set out to try and fix those. It's a well known fact that people operate better with boundaries (cite), they provide guidance on an agreed upon way of doing things. Having said that, if you want to get down and dirty with the engine under the hood, you can, nothing is stopping you having all the flexibility you want. However, most developers don't need to do this. Ember exposes a complete set of APIs that enable you to build a well structured, and maintainable web app. The APIs are intuitive, and simple to use, once you understand the design patterns behind them. One other important thing to note about convention driven frameworks is that if all the developers in your company follow the same conventions, along with an entire community that follows the same conventions, then you know you can look at another Ember developers code or even entire application and immediately know what is going on, and where to find things. The same can't be said for other frameworks that leave those conventions up to you. 

The barrier to entry with Ember is what people have referred to as a steep learning curve. Well, for me this wasn't really the case. The learning curve was shallow as I already had a tonne of experience with the same principles and design patterns that Ember uses, however I understand that not everyone is in the same boat as me. If you've come from building JS apps using native JS and jQuery for example, and have never used, or heard of a design pattern, or even MVC for that matter, the learning curve will be greater. However, bear in mind that that learning curve isn't strictly a learning curve for Ember, those design patterns and principles are core to software engineering, and you really should know them anyway, regardless of Ember. Knowing and understanding design patterns will make you a better developer, guaranteed. (Sources)

##First up, Ember vs Ember Data

I digress, back to Ember. So what stumbling blocks did I encounter? First was understanding the difference between Ember, and Ember data. This isn't always clear, as tutorials often use both together, so let me clarify. Ember provides the stateful application framework upon which you build your app. It's responsible for providing routing, controllers, views, templates (thankfully Ember makes a separation of views and templates, more on that in another blog post) and data bindings (to name a few). The model layer of Ember is left up to you, you can use static data, AJAX requests or just about any data source you can use in a browser. Ember data is a data store and persistency layer that works seamlessly with Ember (as expected) that provides model objects, type casting on model properties, relationships between model objects and persistency to an external data source (e.g. a REST api). Ember data makes it easy to model your data layer and keep that data in sync with your remote data source. No other JS framework that I'm aware of has anything similar to Ember data.  

##Project structure

So, Ember vs Ember data was stumbling block number one. The second was working out where to put my code, business logic and all that. Ember has certain conventions on how to handle events and actions, however it's not always abundantly clear when to use which. For example, one can create a computed property on the model, the controller or the view, there isn't much guidance on which option you choose. Same thing goes for actions, they can be placed on the controller, specific route, or application route, and bubble up from the controller to the app route. Events are another option when it comes to dealing with user interaction, and you may be tempted to use event bindings on the view to respond to user actions when in reality what you really need is an action on the controller. So for the sale of clarity, let me try and clarify:

###Computed properties:
Computed (or "observable") properties are properties that depend on something else, that can be another property that when changed causes the computed property to be re-evaluated, or depend on some external dependcy like formatting a date to a relative date for example. 
If the data that you're computing directly relates to a model objects data, then you're usually best putting the computed property on the model object itself. If however the property relates to perhaps a group of model objects, the number of model objects, or the state of some other properties in a controller (eg menu open or closed, user logged in or out) then the property should go in the controller. Lastly, if the property relates to something that is purely display driven, like a currency symbol for example, then the view is the most appropriate place. Computed properties can be placed on ANY ember object, this includes models, views, controllers, and any objects they contain in their declaration, ie.e, anthing within the `extend()` function call.

```[js]
var Person = Ember.Object.extend({
  // these will be supplied by `create`
  firstName: null,
  lastName: null,

  fullName: function() {
    var firstName = this.get('firstName');
    var lastName = this.get('lastName');

   return firstName + ' ' + lastName;
  }.property('firstName', 'lastName')
});

var tom = Person.create({
  firstName: 'Tom',
  lastName: 'Dale'
});

tom.get('fullName') // 'Tom Dale'


```

In the above example, if either `firstName` or `lastName` changes, then `fullName` will be re-evaluated. Any bindings that are attached to `fullname` will then be updated.

The benefit of computed properties being part of the core framework, means that they're implementation is super effecient, the same can't be said of other frameworks that have imlpemented the same via dirty checking. Dirty checking requires maintaing the previous value of EVERY bound variable, and checking the value at the end of each run loop to see if it has changed. This is a collosal waste of compute cycles and will seriously degrage your performance as your app grows, force you to do things [like this](http://java.dzone.com/articles/improving-angular-dirty).

For further reading, see: [http://emberjs.com/api/classes/Ember.ComputedProperty.html](http://emberjs.com/api/classes/Ember.ComputedProperty.html)


###Actions:
Actions exist to alter the state of the application in some way. Think of an action as transitioning to a new view, submitting a form, or any other action that you would want to catch and manually alter say properties on a model, or open a menu for example. Actions start at the controller level, if not found they bubble up to the controllers route, and if not found there, up to the application route. Knowing where to put the action can be tricky, but I follow the following rule: if the action is affecting something in the current model, or view state, it should belong in the controller. If the action should be shared amongst controllers that may be nested under the same route (think split view, master/detail layouts) then place the action in the main controllers route. If the action relates to overall system state, place it in the application route. Actions are really simple to hook up:

JS:
```js
App.ListController = Ember.Controller.extend({
  // the initial value of the `search` property
  search: '',

  actions: {
    query: function() {
      // the current value of the text field
      var query = this.get('search');
      this.transitionToRoute('search', { query: query });
    }
  }
});
```

HTML:
```html
<header>
  {{input type="text" value=search action="query"}}
</header>
```

In the above example, the controller property `search` is being bound to the value of an input box, thus changing the input box will change the search value and vice versa. The input box is generated via a template helper that binds the enter keydown event, to submit the action specified in `"action"`, in this case, `query` on the controller. On order to declare an action, just define a function below an "actions" property on the controller, simple as that. Actions can recieve an input as arguments. Most of the built in actions can send some data about the action, in the case above, the argument would be the value of "search", in this case, unnecessary due to the binding anyway.

For further reading, see: [http://emberjs.com/guides/controllers/#toc_storing-application-properties](http://emberjs.com/guides/controllers/#toc_storing-application-properties)

###Events:
Views can respond to all the regular DOM events that you'd expect. Sometimes you will have to decide if the interaction you're dealing with is an action or better handled as an event. I use the simple rule that if the interaction does not affect the state of anything within the application, be that controllers or models, or need to do any behind the scenes Ajax requests or something similar, then it may be a good candidate for a view event. Examples of this are social sharing icons, scroll tracking, modifying dom elements based non state driven values (scroll, screen width, etc), or just about anything that has a dependency on jQuery (plugins for example). Events are super easy to hook up, and follow a similar convention to actions  

JS:
```js
App.ClickableView = Ember.View.extend({
  click: function(evt) {
    alert("ClickableView was clicked!");
  }
});
```
HTML:
```html
{{#view "clickable"}}
This is a clickable area!
{{/view}}
```

In the above example, a click event is being defined on the view called `Clickable`, then a click handler is being added to the template via the view helper. Events bubble up from the target view to each parent view in succession, until the root view.

For further reasing, see: [http://emberjs.com/guides/views/handling-events/](http://emberjs.com/guides/views/handling-events/)

##Routes

Third was understanding how to use routes properly. Ember follows a route first principle, this means that everything within an amber app is driven off a route. Routes collect models, controllers and views, they're essentially the conductor between all 3 and mediate the actions between them.

At first this seems like a strange concept, I found myself thinking "I just want to bind this model to this view". Whilst it is indeed possible to use parts of the MVC triad outside of a route context, Ember has this embedded route system for a reason. Firstly, you're making a web app remember, and what single thing is the driving factor for all web apps (HTTP aside)? The URL. The URL is central to everything we do on the web, the source of truth, so why not use the URL to drive application state? Seems like a logical thing to assume. If you grab a URL, copy it, and paste it in an email to your mum (mom) then she would expect to be able to click that link and see what you see. This is largely the case for server side apps, so why not client side? Well, the Ember team decided on this design paradigm from the outset. As the route ties everything together, you automatically get the benefit of shareable URLs, forward and back buttons work, and sub routes allow you to do all sorts of cool nested state handling. So don't try and fight it, embrace it, routes make perfect sense that they should hold the state of your application.

So how does one define a route, it's dead simple:

```js
App.Router.map(function() {
  this.route("about", { path: "/about" });
  this.route("favorites", { path: "/favs" });
});
```

Now, when the user visits /about, Ember.js will render the about template. Visiting /favs will render the favorites template. Note that you can leave off the path if it is the same as the route name. In the above example, the 2nd argument for the "about" route could be left off.

Within templates, there's a handy template helper `{{link-to}};`

```html
{{#link-to 'about'}}<img class="logo">{{/link-to}}
```

Simple, right?

For further reading, see: [http://emberjs.com/guides/routing/defining-your-routes/](http://emberjs.com/guides/routing/defining-your-routes/)

---

##Final thoughts

Once I got the hang of the above, everything else was pretty straight forward, assuming you know MVC. The data bindings for templates are incredibly powerful, the handlebars template syntax is straight forward, components are so useful and easy to write, Ember Data is just incredible, and everything feels cohesive as it's all been developed with the same conventions and opinions. I can't stress enough how awesome Ember.js is, go try it out!

In my next blog post on Ember, I'll go through some getting started points to get you on your way, in the mean time, checkout the [Ember guides](http://emberjs.com/guides/)