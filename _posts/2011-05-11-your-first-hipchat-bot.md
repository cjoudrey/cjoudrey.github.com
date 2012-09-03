---
layout: post
title: "Your first HipChat bot"
---

A few weeks ago I stumbled upon a [promising new startup](https://hipchat.com) while searching for a hosted group chat solution.

Besides being feature-rich and easy to use, [HipChat](https://hipchat.com) allows you to write your own client by exposing an XMPP entry-point.

If you have never heard of [XMPP](http://en.wikipedia.org/wiki/Extensible_Messaging_and_Presence_Protocol) it is a common chat protocol used by services such as [Google Talk](http://code.google.com/apis/talk/open_communications.html#which) and [Facebook Chat](http://developers.facebook.com/docs/chat/).

<!-- more -->

## wobot

[wobot](https://github.com/cjoudrey/wobot/blob/master/README.md) is my attempt at abstracting the XMPP protocol in [Node.js](http://nodejs.org) and offering a simple API to write your own bot.

<img src="/images/2011-05-11-your-first-hipchat-bot/screen1.png"/ >

## Installing wobot

To get started you will need a few things:

  - Node.js ([http://nodejs.org/#download](http://nodejs.org/#download))
  - npm ([http://npmjs.org](http://npmjs.org))

Additionally, wobot depends on the [node-xmpp](https://github.com/astro/node-xmpp) module which requires the following:

  - libexpat1-dev: `apt-get install libexpat1-dev`
  - libicu-dev: `apt-get install libicu-dev`

Once you have installed the build dependencies, install wobot in your working directory using npm:

{% highlight bash %}
mkdir ~/mybot
cd ~/mybot
npm install wobot
{% endhighlight %}

## Configurations

Once wobot is installed you will need to add a new member to your HipChat group and login as that member.

Under "My Account" > "XMPP/Jabber Info" you will find the following:

<img src="/images/2011-05-11-your-first-hipchat-bot/screen2.png"/ >

Instantiate the `wobot.Bot` class as follows:

{% highlight javascript %}
var wobot = require('wobot');
var bot = new wobot.Bot({
  jid: '1234_12345@chat.hipchat.com/bot',
  password: 'yourpassword',
  name: 'XMPP Bot'
});
bot.connect();
{% endhighlight %}

After running the above script, you should see your bot in HipChat's Lobby.

## Auto-joining on connect

Once the bot is connected you will likely want it to join one or many channels. This can be done as follows:

{% highlight javascript %}
bot.on('connect', function() {
  this.join('1234_bot_testing@conf.hipchat.com');
});
{% endhighlight %}

## Reacting to a message

Whenever a message is sent to a channel your bot is in, the `message` event will be emitted.

Here is a simple example of a echo bot:

{% highlight javascript %}
bot.on('message', function(channel, from, msg) {
  if (from === this.name) return false;
  this.message(channel, '@' + from + ' you just said: ' + msg);
});
{% endhighlight %}

<img src="/images/2011-05-11-your-first-hipchat-bot/screen3.png"/ >

## Conclusion

The goal of this article was to briefly introduce wobot.

There are many [more examples](https://github.com/cjoudrey/wobot/tree/master/examples) and [detailed documentation](https://github.com/cjoudrey/wobot/blob/master/README.md) on GitHub.

-Christian Joudrey
