---
title: Javascript redirect and Piwik tracking
date:  2016-09-02 23:20:00
---

Today I set up a custom shortlink: <https://zetamode.com/questionnaire> pointing to a Google Form
(Click it if you speak french and are interested in online fashiown game, we're doing a survey !)

The [zetamode.com](https://github.com/zetamode/zetamode.com) website is hosted with GitHub Pages, so I cannot go with an HTTP redirect.
And I want to track the origin of the shortlink users through my Piwik instance.

[Of course] I did not find anybody who had the same use-case on Google.
But I found two interesting articles: [Here](http://stackoverflow.com/a/36846720/3133038) someone explains how to replicate
the behavior of the [jekyll-redirect-from](https://github.com/jekyll/jekyll-redirect-from) plugin (without tracking) and
[here](http://stackoverflow.com/questions/8692503/javascript-redirect-with-google-analytics) there are a few interesting ways
to do a javascript redirect while tracking it with Google Analytics.
I find [this one](http://stackoverflow.com/a/8692588/3133038) particularly interesting.
It involves turning the async tracking code into a sync one, and put the redirect after the tracking code.
This prevents the redirect from happening before the tracking, and we don't need hard-coded waiting times.

You can have a look at the actual file [here](https://github.com/zetamode/zetamode.com/blob/gh-pages/_layouts/redirect.html), but below is a more generic version you are welcome to edit to fit your needs:

{% gist clarkwinkelmann/fa56404d69b36ed5f67a561fbd41d5ce redirect.html %}

And to use it:

{% gist clarkwinkelmann/fa56404d69b36ed5f67a561fbd41d5ce shorturl.html %}

**Note:** The [SO answer I based my code on](http://stackoverflow.com/a/36846720/3133038) states that
you should not use `page.redirect_to` as the parameter name, as it will conflict with the `jekyll-redirect-to` plugin,
which I believe is enabled on GitHub Pages (didn't test myself).

So far it seems this method works !

Sources:

- <http://stackoverflow.com/a/36846720/3133038>
- <http://stackoverflow.com/a/8692588/3133038>
