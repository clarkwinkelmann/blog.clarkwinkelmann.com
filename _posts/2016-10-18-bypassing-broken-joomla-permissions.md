---
title: Bypassing broken Joomla! permissions
date:  2016-10-18 19:00
---

I just retired an (already) old Joomla! website to replace it with a [new Concrete5 one](https://www.agrijura.ch/).

I backed up the whole Joomla! database and files.
I still needed to access some archive items so I set up the Joomla website on my local machine.

I am not familiar with Joomla! at all, but it's certainly not the first time that I try to launch an "old" website on localhost.

I was afraid that the PHP version would mess everything.
I run 7.0 on my dev machine and the old server hosting the website was still on 5.2.

I imported the database on my local MySQL instance, edited the config file and fired the PHP dev server in the root directory.
Incredibly, it worked !

For reference, here was the Joomla version (does not seem too old - again, I don't know anything about Joomla!):

> | Joomla! Version          | Joomla! 2.5.24 Stable [ Ember ] 25-July-2014 13:00 GMT                  |
> | Joomla! Platform Version | Joomla Platform 11.4.0 Stable [ Brian Kernighan ] 03-Jan-2012 00:00 GMT |

I had access to the administration panel, but what I wanted was to see the old website as it was previously online
(I need to access specific known URLs and it's just a mess in the admin panel).
Here, things were not OK: the home page was displayed, but there was no menu.
Every other page returned an error:

> You are not authorised to view this resource.

A few minutes on [DuckDuckGo](https://duckduckgo.com/?q=joomla+You+are+not+authorised+to+view+this+resource),
or [Google](https://encrypted.google.com/search?q=joomla%20You%20are%20not%20authorised%20to%20view%20this%20resource),
or whatever reveals there could be **A TON** of reasons behind this message.
And even the StackOverflow answers are not very helpful, because each case is very specific.

I tried a few of the suggested actions.
Cleared cache, but it seems it was disabled;
check permissions, everything is public;
empty trash - wait, I still have no idea where is this damn Joomla! trash.
Nothing helped. Still same error.

I didn't want to lose time into debugging this, **I just wanted to see the website**.
Looking for ways to disable the permissions all together didn't yield interesting results either.

So before spending a day on the internet to troubleshoot this I decided to just go into the PHP files and edit whatever needed to be able to see the website.

Fortunately, enabling debug mode is as simple as:

`configuration.php`:

```php
class JConfig {
    // [...]
    public $debug = '1';
    // [...]
}
```

This enables a nice little console at the bottom of each page.

On a problematic page I can see in **Joomla! Debug Console > Errors**:

> **You are not authorised to view this resource.**
>
> **Call stack**
> 
> | #  | Function                             | Location |
> | 1  | JSite->dispatch()                    | JROOT/index.php:42 |
> | 2  | JComponentHelper::renderComponent()  | JROOT/includes/application.php:194 |
> | 3  | JComponentHelper::executeComponent() | JROOT/libraries/joomla/application/component/helper.php:348 |
> | 4  | require_once()                       | JROOT/libraries/joomla/application/component/helper.php:380 |
> | 5  | JController->execute()               | JROOT/components/com_content/content.php:16 |
> | 6  | ContentController->display()         | JROOT/libraries/joomla/application/component/controller.php:761 |
> | 7  | JController->display()               | JROOT/components/com_content/controller.php:74 |
> | 8  | ContentViewArticle->display()        | JROOT/libraries/joomla/application/component/controller.php:722 |
> | 9  | JError::raiseWarning()               | JROOT/components/com_content/views/article/view.html.php:104 |
> | 10 | JError::raise()                      | JROOT/libraries/joomla/error/error.php:276 |

The error (`raiseWarning()` does not seem a really good name here) comes from the `view.html.php` file.

You can disable it by removing the whole `if` block checking if user has access.

`components/com_content/views/article/view.html.php:101`:

```php
// remove:
if ($item->params->get('access-view') != true && (($item->params->get('show_noauth') != true &&  $user->get('guest') ))) {
    JError::raiseWarning(403, JText::_('JERROR_ALERTNOAUTHOR'));
    return;
}
```

This removes the error but the page is still blank.
On pages with "paged" navigation, the navigation now appears, but not the content.
A quick look in the subdirectories of this `article/view` module reveals a template file `tmpl/default.php` where additional checks occur.
By hard-coding `true` into the normal display case the content shows up.

In `components/com_content/views/article/tmpl/default.php:161` change:

```php
<?php if ($params->get('access-view')):?>
```

to:

```php
<?php if (true):?>
```

And voilà.

This does not solve the problem of the menu, but it was not important for me.
It may be a little harder to fix because there are no errors indicating where to look, but I expect a similar menu controller and template somewhere.

I hope this article will be useful to somebody else facing the same issue.
I also put some bold text saying again that **this is a hack to access every page on a local Joomla! instance, do not try this in production !**
