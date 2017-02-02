---
title: Step-by-step creation of my Flarum Emoji extension
date:  2017-02-02 01:00
---

This is a step-by-step guide on how I created my EmojiOne Area Picker extension for Flarum. For more info about my research, trial&error and sources or information, see [my other article]({% post_url 2017-02-02-my-first-flarum-extension %}).

## Before we start

This was done with Flarum beta 6 on an Ubuntu 16.04 system. I cannot guarantee it will apply to future versions of Flarum.

I have a local Flarum install in `/var/www/flarum`. This is referred as the *Flarum root*.

In code snippets, `[...]` indicates that nothing changed from the last snippet of the same file.

## Workbench setup

> Based on [this discussion](https://discuss.flarum.org/d/1608-extension-development-using-composer-repositories-path).

Create a new `workbench` folder under the Flarum install and register it as a repository in Flarum's `composer.json`:

```json
    "repositories": [
        {
            "type": "path",
            "url": "workbench/*/"
        }
    ],
```

## Extension setup

Create new new folder with the name of the extension inside the `workbench` folder, in my case `workbench/flarum-ext-emojionearea`. That's where my extension will be placed and I will refer to it as the *Extension folder*.

Next create a `composer.json` file inside the extension folder with your details:

```json
{
    "name": "clarkwinkelmann/flarum-ext-emojionearea",
    "description": "Add EmojiOne Area emoji picker to Flarum",
    "type": "flarum-extension",
    "require": {
        "flarum/core": "^0.1.0-beta.6"
    },
    "extra": {
        "flarum-extension": {
            "title": "EmojiOne Area"
        }
    }
}
```

Then require the extension in Flarum's `composer.json` file with the following line:

    "clarkwinkelmann/flarum-ext-emojionearea": "*@dev"

Running `composer update` will add a symlink from the `vendor` directory to the extension folder, so we don't need to run a Composer command every time we change our extension in the workbench.

## Javascript setup

All following paths are relative to the *extension path*.

Javscript code will be placed in `js/forum` in the extension folder. In this folder we will need two files:

`js/forum/package.json`:

```json
{
    "private": true,
    "devDependencies": {
        "gulp": "^3.8.11",
        "flarum-gulp": "^0.1.0"
    },
}
```

`js/forum/Gulpfile.js`:

```javascript
var flarum = require('flarum-gulp');

flarum({
    modules: {
        'clarkwinkelmann/emojionearea': [
            'src/**/*.js'
        ]
    }
});
```

We also create the javascript bootstrap file that will setup our extension.

`js/forum/src/main.js`:

```javascript
app.initializers.add('clarkwinkelmann-emojionearea', () => {
    // Nothing yet
});
```

When running `gulp` or `gulp watch` this will create an `extension.js` file in the `js/forum/dist` folder.

## LESS setup

This is easy, create a `less/forum/extension.less` file.

## PHP setup

We only need PHP to register our assets, this is done in `bootstrap.php` at the root of the extension folder:

```php
<?php

use Flarum\Event\ConfigureClientView;
use Illuminate\Contracts\Events\Dispatcher;

return function (Dispatcher $events) {
    $events->listen(ConfigureClientView::class, function (ConfigureClientView $event) {
        if ($event->isForum()) {
            $event->addAssets([
                __DIR__ . '/js/forum/dist/extension.js',
                __DIR__ . '/less/forum/extension.less',
            ]);
            $event->addBootstrapper('clarkwinkelmann/emojionearea/main');
        }
    });
};
```

## Coding javascript

> Inspired by the [flagrow/upload](https://github.com/flagrow/upload) extension.

We will use the [mervick/emojionearea](https://github.com/mervick/emojionearea) jQuery plugin, so we first need to add it as a dependency:

`js/forum/package.json`:

```json
{
    "private": true,
    "devDependencies": {
        "gulp": "^3.8.11",
        "flarum-gulp": "^0.1.0",
        "emojionearea": "^3.1.5"
    }
}
```

We need to create our own button. For that, we create a new file for our `EmojiAreaButton` that is based on the base `flarum/Component`:

`js/forum/src/components/EmojiAreaButton.js`:

```javascript
import Component from "flarum/Component";
import icon from "flarum/helpers/icon";

export default class EmojiAreaButton extends Component {

    view() {
        return m('div', {className: 'Button Button-emojionearea hasIcon Button--icon'}, [
            icon('smile-o', {className: 'Button-icon'}),
            m('span', {className: 'Button-label'}, 'Emojis'),
            m('span', {className: 'Button-emojioneareaContainer'})
        ]);
    }

}
```

The `.Button-emojionearea` class allows us to target the button with CSS and the `.Button-emojioneareaContainer` element will contain the picker that we will load with jQuery.

We need to run the `$.emojioneArea()` method after this component has been loaded. For that we make use of Mithril's `config` attribute. This allows you to supply a method that will be called each time a DOM item is redrawn. The `isInitialized` parameter informs us the method has already been called.

```javascript
    view() {
        return m('div', {config: this.configArea.bind(this), className: '[...]'}, [
            // [...]
        ]);
    }

    configArea(element, isInitialized) {
        if (isInitialized) return;

        var $container = $(element).find('.Button-emojioneareaContainer');
    }
```

We can then load our javascript plugin in the `configArea()` method. We load emojiOneArea on a new jQuery `<div/>` so it does not try to find a `textarea` and we put the picker inside our `$container`:


```javascript

    configArea(element, isInitialized) {
        // [...]

        $('<div />').emojioneArea({
            container: $container,
            standalone: true,
            hideSource: false,
            events: {
                emojibtn_click: function (button, event) {
                    // emoji clicked
                }
            }
        });
    }
```

We can then add our button to the post editor.

`js/forum/src/main.js`:

```javascript

import {extend} from "flarum/extend";
import TextEditor from "flarum/components/TextEditor";
import EmojiAreaButton from 'clarkwinkelmann/emojionearea/components/EmojiAreaButton';

app.initializers.add('clarkwinkelmann-emojionearea', () => {
    extend(TextEditor.prototype, 'controlItems', function (items) {
        var emojiButton = new EmojiAreaButton;
        items.add('clarkwinkelmann-emojionearea', emojiButton, 0);
    });
});
```

In order for the jQuery plugin to load, we need to add it to our Gulpfile as a `file`.

`js/forum/Gulpfile.js`:

```javascript

var flarum = require('flarum-gulp');

flarum({
    files: [
        'node_modules/emojionearea/dist/emojionearea.js'
    ],
    modules: {
        'clarkwinkelmann/emojionearea': [
            'src/**/*.js'
        ]
    }
});
```

After running `npm install` and `gulp` (or `gulp watch`) in `js/forum`, the extension can be enabled in the Flarum install and the Emoji picker is loaded inside the button. Clicking on an emoji does not do anything yet.

To add text to the post, we need to call the `insertAtCursor()` method on the `TextEditor` instance. This is not accessible from our button by default, we need to save a pointer when we extend the editor.

We add the attribute to our button:

`js/forum/src/components/EmojiAreaButton.js`:

```javascript
// [...]

export default class EmojiAreaButton extends Component {

    init() {
        this.textEditor = null;
    }

    // [...]

}
```

And fill it at extend time:

`js/forum/src/main.js`:

```javascript
// [...]

app.initializers.add('clarkwinkelmann-emojionearea', () => {
    extend(TextEditor.prototype, 'controlItems', function (items) {
        var emojiButton = new EmojiAreaButton;
        emojiButton.textEditor = this;
        items.add('clarkwinkelmann-emojionearea', emojiButton, 0);
    });
});
```

Now we can access it when an emoji is clicked:

`js/forum/src/components/EmojiAreaButton.js`:

```javascript
    configArea(element, isInitialized) {
        // [...]

        var editor = this.textEditor;

        $('<div />').emojioneArea({
            // [...]
            events: {
                emojibtn_click: function (button, event) {
                    var shortcode = button.data('name');
                    editor.insertAtCursor(shortcode);
                }
            }
        });
    }
```

And it works ! Some optimisations were not shown here, have a look at the [source code](https://github.com/clarkwinkelmann/flarum-ext-emojionearea).

## Coding CSS/LESS

The `less/forum/extension.less` file contains a few rules to hide the parts of the picker we do not need. Have a look at it [here](https://github.com/clarkwinkelmann/flarum-ext-emojionearea/blob/master/less/forum/extension.less) on GitHub for the details.

> This CSS part is my own invention [because I did not find any other extension doing it](https://discuss.flarum.org/d/4651-how-to-import-css-from-dependency)

To include the CSS of the picker, I use another (normal) Gulp file to copy the CSS needed to a `dist` folder, much like the javascript part:

`css/forum/package.json`:

```json
{
    "private": true,
    "devDependencies": {
        "gulp": "^3.8.11",
        "emojionearea": "^3.1.5"
    }
}
```

`css/forum/Gulpfile.js`:

```javascript
var gulp = require('gulp');

gulp.task('default', function() {
    gulp.src([
        'node_modules/emojionearea/dist/emojionearea.css'
    ]).pipe(gulp.dest('dist'));
});
```

Running `npm install` and `gulp` in `css/forum` produces a `css/forum/dist/emojionearea.css` file that will be included in version control. We also add this file to the `bootstrap.php` file:


```php
<?php

use Flarum\Event\ConfigureClientView;
use Illuminate\Contracts\Events\Dispatcher;

return function (Dispatcher $events) {
    $events->listen(ConfigureClientView::class, function (ConfigureClientView $event) {
        if ($event->isForum()) {
            $event->addAssets([
                __DIR__ . '/js/forum/dist/extension.js',
                __DIR__ . '/less/forum/extension.less',
                __DIR__ . '/css/forum/dist/emojionearea.css',
            ]);
            $event->addBootstrapper('clarkwinkelmann/emojionearea/main');
        }
    });
};
```

To finish properly, add a `README.md`, `LICENSE.txt` and `.gitignore` file. Here's what I used in the `.gitignore` file:

```
**/node_modules
/vendor
```

In my case I did not even had to run `composer install` in the extension folder, but if I had unit tests (for example) I would have to run composer. That's where the `/vendor` rule in the gitignore would be useful.

Now your extension is ready for `git init` and `git commit` !

Have a look at:

- [The GitHub repo](https://github.com/clarkwinkelmann/flarum-ext-emojionearea)
- [The discussion on Flarum Discuss](https://discuss.flarum.org/d/4787-emoji-picker)

> Written with [StackEdit](https://stackedit.io/).
