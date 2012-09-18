* This README is from @ryandotsmith gist: https://gist.github.com/1655019  

# A Backbone.js demo app (Sinatra Backend)

Oct 16 2010

## Updates

* 04/10/2011 - Updated application.js and application.rb thanks to @rebo's comments


**In this article, I will walk through some simple steps to get a [demo app][2] up and running with [Backbone.js][3] and [Sinatra][4] on [Heroku][5].**

A few days ago I came across an interesting link on Hacker News. This link led me to a delightful discovery, Backbone.js. Backbone is a MVc where c stands for Collections. More info [here][6].

Onto the demo app. The idea is to make a campfire-like app. In this demo I will
use [Sinatra][7] for the backend. The structure of the app should look like
this:

```bash
  - application.rb
  - public
  - - index.html
  - - jquery.js
  - - backbone.js
  - - underscore.js
  - - application.js
```
The view of our web app should look like this:

[![View] (http://img.skitch.com/20101023-8bh2d2ttebtc3t1wqm1yk4wf1w.preview.jpg)][10]


Let's begin with the HTML.

```html
<!-- views/index.html -->

<!DOCTYPE HTML>
<html>

  <head>
    <meta http-equiv="content-type" content="text/html;charset=UTF-8" />
    <script src="underscore.js"></script>
    <script src="jquery.js"></script>
    <script src="backbone.js"></script>
    <style type='text/css'>
      textarea {
        height:500px;
        width:500px;
      }
      input[type="text"] {
        width: 450px;
      }
      #chatArea {
        width:600px;
        margin: 10px auto;
      }
    </style>
  </head>

  <body>
    <div id="chatArea">
      <textarea id='chatHistory'></textarea>
      <form method="post" action="#" id= 'chatForm' name="newMessage" onsubmit="return false">
        <input name= 'newMessageString' type="text" />
        <input type="submit" value='send'/>
      </form>
    </div>
  </body>

  <script src="application.js"></script>
</html>
```

Great! Notice I am returning false when we submit the form. Also,
I am including underscore, jquery, backbone and at the bottom of the
file I am including our application.js file. This is where all of our backbone implementation will go.
If you have not already done so, please download jquery, underscore & backbone and
place those files inside of the public directory.

```javascript
// public/application.js
var Message = Backbone.Model.extend({});
```
Easy enough. Inside the code block of extend() is where we could setup our
initializers and other domain specific methods. For now, the defaults will be
good enough.

Next we need to create a collection to hold onto all of our messages. We will
add some more code to application.js

```javascript
// public/application.js
var Message = Backbone.Model.extend({});

var MessageStore = Backbone.Collection.extend({
 model: Message,
   url: 'http://localhost:4567/messages'
});
var messages = new MessageStore;
```

Notice how we specify the model in our Collection. If we do this,
we can use shortcut methods that can create instances of Message and add them to our collection.

Great! Now that we have our Model and Collection set up. We can work on building a View.

```javascript
// public/application.js
var Message = Backbone.Model.extend({});

var MessageStore = Backbone.Collection.extend({
 model: Message,
   url: 'http://localhost:4567/messages'
});
var messages = new MessageStore;

var MessageView = Backbone.View.extend({

   events: { "submit #chatForm" : "handleNewMessage" }

  , handleNewMessage: function(data) {
    var inputField = $('input[name=newMessageString]');
    messages.create({content: inputField.val()});
    inputField.val('');
  }

  , render: function() {
    var data = messages.map(function(message) { return message.get('content') + 'n'});
    var result = data.reduce(function(memo,str) { return memo + str }, '');
    $("#chatHistory").text(result);
    return this;
  }

});
```

The code to note here is the **events** object and the **handleNewMessage** function.
The **render** function should always be present in your view object and should take
care of drawing the view. So, in my events object I can specify all sorts of events
to watch and then specify a function to call when that event is triggered.
The signature for the selector, in our case **submit #chartForm** is **event selector**.
If no selector is present, the listener is bound the entire view object.

So you might be wondering about the "view object." Unlike Rails' views,
Backbone views our responsible for a much smaller portion of the page.
Think about an HTML table that has rows. The Backbone.js pattern would
have you make a Model instance for each row.

You should also take note of the **messages.create({ ... }) **function call
in the previous code snippet. This is the shortcut I was alluding to a
few paragraphs ago. This short cut is creating a new **Message, **calling **save()**
and then adding it to our collection via the **add() **function. Since the **add() **
function was invoked, we can add a listener that reacts to this change. In the case
of our chat application, when a new message is added to the collection, we want
to update the display with our new message and any other messages in the system.


```javascript
// public/application.js
var Message = Backbone.Model.extend({});

var MessageStore = Backbone.Collection.extend({
 model: Message,
   url: 'http://localhost:4567/messages'
});
var messages = new MessageStore;

var MessageView = Backbone.View.extend({

   events: { "submit #chatForm" : "handleNewMessage" }

  , handleNewMessage: function(data) {
    var inputField = $('input[name=newMessageString]');
    messages.create({content: inputField.val()});
    inputField.val('');
  }

  , render: function() {
    var data = messages.map(function(message) { return message.get('content') + 'n'});
    var result = data.reduce(function(memo,str) { return memo + str }, '');
    $("#chatHistory").text(result);
    return this;
  }

});

messages.bind('add', function(message) {
  messages.fetch({success: function(){view.render();}});
});
```

The first thing to point out is **messages.fetch({ ... }) **. This is a function from Backebone's **Collection** class.
If you are interfacing with a RESTful backend, it will fetch the index of the collection and merge
the data into your Backbone.js collection. **fetch()** takes a success and error callback. Our success callback
calls **render()** on our **view** object. Last but not least, we connect our HTML markup with our MVc.

```javascript
// public/application.js
var Message = Backbone.Model.extend({});

var MessageStore = Backbone.Collection.extend({
 model: Message,
   url: 'http://localhost:4567/messages'
});
var messages = new MessageStore;

var MessageView = Backbone.View.extend({

   events: { "submit #chatForm" : "handleNewMessage" }

  , handleNewMessage: function(data) {
    var inputField = $('input[name=newMessageString]');
    messages.create({content: inputField.val()});
    inputField.val('');
  }

  , render: function() {
    var data = messages.map(function(message) { return message.get('content') + 'n'});
    var result = data.reduce(function(memo,str) { return memo + str }, '');
    $("#chatHistory").text(result);
    return this;
  }

});

messages.bind('add', function(message) {
  messages.fetch({success: function(){view.render();}});
});

var view = new MessageView({el: $('#chatArea')});

setInterval(function(){
  messages.fetch({success: function(){view.render();}});
},1000)

```

**el** is the element that our view object is bound to. (Backbone.js will make an **el** for us if not specified) Remember when I gave the example of view objects being coupled to a row of an HTML table? Well, in the spirit of that example, each **tr **would correspond to an **el.**

Now let's build an API using Sinatra.

```ruby
# application.rb

require 'sinatra'
require 'json'

@@data = []
@@count = 0

get '/' do
  File.read(File.join('public', 'index.html'))
end

get '/messages' do
  content_type :json
  {:models => @@data }.to_json
end

post '/messages' do
  content_type :json
  message = JSON.parse(params[:model]).merge(:id => @@count += 1 )
  @@data << message
  message.to_json
end

```
This app will work wonderfully on [heroku][5].

Also, here is a link to the [git repo][11].

**<http://fire-camp.heroku.com/>**

[2]: http://fire-camp.heroku.com/
[3]: http://documentcloud.github.com/backbone/
[4]: http://www.sinatrarb.com
[5]: http://www.heroku.com
[6]: http://documentcloud.github.com/backbone
[7]: http://www.sinatrarb.com/
[9]: http://pixel.quantserve.com/pixel/p-19UtqE8ngoZbM.gif
[10]: http://img.skitch.com/20101023-8bh2d2ttebtc3t1wqm1yk4wf1w.jpg
[11]: https://github.com/ryandotsmith/fire-camp
