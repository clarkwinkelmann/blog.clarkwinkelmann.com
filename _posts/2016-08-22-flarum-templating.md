---
title: Flarum templating
date:  2016-08-22 16:20:00
---

An interesting discussion was taking place on the [Flarum Gitter](https://gitter.im/flarum/flarum) today.
I figured I should better write an article instead of writing thousands of words in a small text input.

I'm new to [Flarum](http://flarum.org/), I didn't even have time to write my first extension.
But I read all the docs, looked at the source code of most core extensions,
and I must say this is some very impressive technology.

So impressive, that some devs get confused very quickly.

Why use composer, why do I have to write JavaScript to edit the templates, why can't I just paste my HTML there ?

I will try to explain this the best as I can.
Hopefully, I will be able to re-use this post everytime someone gets confused.

## The old way of doing things

In very old web application, you just didn't have *Views*.

For example, in [osCommerce](https://www.oscommerce.com/) (until v2 at least, I don't know if v3 will ever be released)
you had to edit the core files to change how things look.
Want to add a link to the user settings ?
Better edit that `account.php` file in the root folder.

Of course, everything breaks when:

- There is an update to the software (which requires to overwrite the files you edited)
- You want to share your amazing feature with somebody else which has a base installation
- or worse, someone who **already has another extension** installed

I'd like to say nobody does it anymore but guess that ?
I'm actually maintaining a 2005 shop which cannot easily be migrated.

Anyway, most websites now use another technique:

## The current way of doing things

If you come from a [WordPress](https://wordpress.org/) background for example, this is the way things work:

You get your own special directory somewhere like `wp-content/themes/myamazingtheme`.
There, you create `php` template files, which get used for a given request.
If you don't create a file, a default one kicks in and everything works.

In this file, you get access to a few variables, so you know what to display.
You can include other files to prevent duplication.

Important note: Views can not only return HTML.
You may also format your data as XML or JSON for example.

Congratulations, you are using *Views* !
[MVC frameworks](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller) takes this to a higher level,
by using them to separate the visual interface from the data.
I won't go into that, there are loads of good MVC tutorials out there.

In modern frameworks like [Laravel](https://laravel.com/), there are other components which glue everything together.
The only one that you should know to understand my article is the Router, which maps URLs to Controllers.
A Controller then returns a View.

A typical request to a web application could be drawn like this:

    User > Request (GET) > Router > Controller > Fetch data > View (HTML) > Page

## The modern way of doing things

The new trend is JavaScript applications.

How it works: The first time you load the page, the application loads, without worrying of what the current page is.
When it is ready or when you change page, an AJAX request is issued to get the new content, and the interface is updated through JavaScript.

It now works like this:

    User > Request (GET) > Controller > View (HTML) > Empty page (JavaScript App)
    JS App > Request (AJAX) > Controller > Fetch data > View (JSON) > Page update

This changes everything.

We still have a Controller which returns HTML, but there is no data here, just the application bootstrap code.

When the JavaScript application is started, it fetches the data based on the current URL.
No fancy HTML there, the data is returned in JSON format, and the JavaScript takes care of placing this data at the correct place.

Now you think you've seen everything ? Nope.

## MVC on client side

Flarum [uses](http://flarum.org/docs/extend/start/#changing-the-ui) [Mithril](http://mithril.js.org/),
which is itself an **MVC Framework**, but **on the client side** !

This means there is also a Router, some Controllers and of course Views written in JavaScript !

Let's draw a new schema taking that into account.
Here's what happens on a typical page load:

    (Server) User > Request (GET) > Router > Controller > View (HTML) > Page (Mithril App)
    (Client) Mithril App > Router > Controller > Data request [...]
    (Server) Mithril > Request (AJAX) > Router > Controller > Fetch data > View (JSON) > Response
    (Client) [...] > Parse response > Controller > View (Virtual DOM) > HTML update

Now, how do Mithril Views work ?
Pretty much the same as the PHP ones.
The view consists of an HTML structure, which is written inside the page and takes the whole place.

To make things easier and prevent code duplication, Mithril uses Components, which can be "included" from a parent Component.
You can find some example Components in the [Flarum documentation](http://flarum.org/docs/extend/start/#components), like *DiscussionPage*, *PostStream*, *Post*, etc...

How to extend the interface ?
You just need to change the HTML output of the JavaScript Component of the item you want to change on the page.
You can add new Components inside existing ones, too !
From now on the official Flarum doc will tell you where to change things: <http://flarum.org/docs/extend/start/>

One last thing: [Mithril](http://mithril.js.org/), like [React](https://facebook.github.io/react/), uses Virtual DOM.
This is a way to make things faster by computing the "HTML" in memory, and updating only the bits of the page that changed across requests.

The difference for you is that you don't return a "string of HTML" but a "representation of the HTML", which is made of JavaScript objects.
The good news is that you can write HTML almost normally and it will be turned into cryptic objects at compilation time.
Have a look at how it looks on [Mithril's home page](http://mithril.js.org/) or the [Flarum Extensions documentation](http://flarum.org/docs/extend/start/).

## Recap

So what did we learn here (I hope you learned something lol).

- The visual interface of Flarum is a JavaScript MVC application powered by [Mithril](http://mithril.js.org/)
- It talks to a PHP application which just sends data as JSON, not as HTML
- The PHP application does uses Views, but only to render the JavaScript application
- The JavaScript Views are organised as Components, which are placed in a hierarchical way on the page
- To change how something looks like, you have to edit the Component responsible for it
- You can add new Components as child of existing ones

I won't go into further details on the development of Extensions for now.
I think the [official](http://flarum.org/docs/extend/start/) one is pretty good once you know how the technologies used work together.

This is my first tutorial, please give me feedback [on Twitter](https://twitter.com/clarkwinkelmann) or anywhere else that you can find me =)

## Additional notes

There are in fact a few HTML templates in the [views folder of Flarum](https://github.com/flarum/core/tree/master/views).
They are used to display a simple alternative interface to users with JavaScript disabled.

I only cover the problems that newcomers may face when editing Flarum *interface*.
Adding *functionality* is another topic that includes things like *Providers* and *Events* (as opposed with WP *Hooks*).
I'll write another article if there is interest in the matter !
