---
title: My first Flarum extension
date:  2017-02-02 01:00
---

This is a diary on the creation of my first Flarum extension. For the step-by-step tutorial (without all the trial and error rambling) have a look at [this other article]({% post_url 2017-02-02-step-by-step-flarum-emoji-extension %}).

## Starting point

I just replaced the manually crafted [ZetaMode Forum](https://forum.zetamode.com/) with [Flarum](http://flarum.org/).

Both [ZetaMode](https://zetamode.com/) and Flarum being in beta, this seemed like a good idea. I'm already expecting things to break anyway, so I prefer to help debug and improve Flarum over building custom in-house stuff.

Of course, new forum means new user suggestions. The first thing reported was it was hard for new users to guess the emoji shortcodes.

In the previous forum, I added a link to [emoji.codes](http://emoji.codes/) at the bottom of the autocomplete popup. But there is no such link in the Flarum emoji popup, nor is an emoji picker included.

I did a quick search to find if anybody had already made one. I found [this thread](https://discuss.flarum.org/d/3206-emojione). A few people were looking for the same thing as me. Also, there was a link to a jQuery plugin named [emojionearea](https://github.com/mervick/emojionearea).

Since it looked like nobody had created an extension for that before, I decided to give it a try. I tried to find other existing emoji picker libraries on the web. I found this one by [@tommoor](https://github.com/tommoor/emojione-picker) and this one by [@OneSignal](https://github.com/OneSignal/emoji-picker). The first was written for React, which is probably hard to integrate with Mythril, and the second one used a hell lot of files.

I hesitated. The OneSignal one had more stars on GitHub, but the meverick one had more options. Also, I've seen other Flarum extensions use jQuery plugins so I was more confident to success with it.

## Coding the extension

I went back to have a look at the [official doc](http://flarum.org/docs/extend/start/). The start of this page is a bit fishy because I've read somewhere there is no more `extensions` directory.

I remembered seeing a composer dev tutorial on the forum, which I quickly [found again](https://discuss.flarum.org/d/1608-extension-development-using-composer-repositories-path). That tutorial is really straightforward and it was easy to setup the project.

I then proceeded to the javascript part of the extension. There, the official doc is really easy to follow. I created a `js/forum` folder inside my extension folder with a `package.json` and `Gulpfile.js`.

In addition to my own javascript code for UI integration, I needed to import the jQuery plugin. Adding the file path inside the `modules` section was not working. I found this [geotags](https://github.com/Avatar4eg/flarum-ext-geotags/blob/master/js/forum/Gulpfile.js) plugin that also used a jQuery plugin and found that I could use a `files` section to add raw JS imports (the code snippet above is the final version)

Then, I had to find out which components I had to extend. Again the official doc is great to explain how it works, but I had to look inside other's extensions source code to find out which was the right component and how it worked.

I first had a look at the [flarum/flarum-ext-emoji](https://github.com/flarum/flarum-ext-emoji), which does something similar to what I want to do (it implements the emoji autocomplete, but not a real picker). But it turned out it was not necessary to hack into the `ComposerBody` component like they had to do with the autocomplete.

The [flagrow/upload](https://github.com/flagrow/upload) extension is actually more similar to what I want to do. It adds a button to the post composer, and this button alters the content of the post.

First I tried to use the emojionearea plugin the way it was meant to be: by just running `$textarea.emojioneArea()`. But I quickly found out it was destroying the whole post composer. Indeed, this plugin creates its own `contenteditable` area and hides the original textarea, which breaks the editor. By adjusting the plugin settings, I can change which divs get replaced / used, but there was no way to make it keep the original textarea untouched.

Hopefully, this plugin also had a "standalone" mode. I was able to embed this box under the post editor. I was then able to use the plugin events to detect an emoji selection and use the Flarum `TextEditor` functions to insert the emoji in the post. Great !

Last part, I needed to put this selector inside a Flarum button and hide its own button style (the standalone version displays the current selected emoji and a button with an icon - but it does not fit nicely with Flarum). Some custom CSS in `less/forum/extension.less` did the trick, hiding what's not required. `opacity: 0` on the original button allows the user to click on the original button through the Flarum button.

The tricky part was to include the original CSS file of EmojiOne Area into the extension as well.

Even after [desperately asking for help](https://discuss.flarum.org/d/4651-how-to-import-css-from-dependency) on the forum, I still have not found any example of extension doing that. Let's consider I am the first ever person to do it and have to decide myself how to do it.

The easy option would have been to use the Gulpfile inside the `js/forum` folder to copy the CSS to a dist folder. Unfortunately, the Flarum gulp helper does not seem to have an option to copy files. Plus, doing this in the `js` folder didn't seem right.

So I created a new `css/forum` folder in my project folder, with another `package.json` and `Gulpfile.js`. There I load the same emojionearea library, as well as a standard Gulp. With a simple gulp task, I copy the CSS file I need to a `dist` folder that is included in my version control.

```javascript
var gulp = require('gulp');

gulp.task('default', function() {
    gulp.src([
        'node_modules/emojionearea/dist/emojionearea.css'
    ]).pipe(gulp.dest('dist'));
});
```

There is no real back-end code for this extension, the only thing to do is to register the assets. I decided to put everything in `bootstrap.php`, but if I had more files I should probably store events listener in an `src` folder and only list these in the bootstrapping file like most extensions do.

I then commited the extension folder to my GitHub account, created a new release tagged `v0.1.0` and registered it on [Packagist](https://packagist.org/). That was my first time adding a package to [Composer](https://getcomposer.org/). That was really easy and straightforward.

In conclusion, creating front-end extensions is not as scary as it looked like for me. I will now try to create more complex ones. Please tell me if you want other articles from me on the subject !

**Links:**

- [The code on GitHub](https://github.com/clarkwinkelmann/flarum-ext-emojionearea)
- [The discussion on Flarum Discuss](https://discuss.flarum.org/d/4787-emoji-picker)

> Written with [StackEdit](https://stackedit.io/).
