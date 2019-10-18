---
layout: post
title:  "Debug Magento 1 Modules with VS Code + Xdebug"
tags: magento1 xdebug debugging
---

I still have a lot to do with **Magento 1** modules in my daily business. As everybody else, I often have to turn to Xdebug to check out what's going on in the code. Unfortunately, modules in Magento 1 are usually *symlinked* in the Magento `app/code/*` directory: in my experience, it is better to avoid working in a jungle of symlinked directories, else you might end up with a version control mess. If you want to open the real project directory, you have to let Xdebug know about that, or the request will never hit any of your breakpoints.

To be able to debug Magento 1 modules working in the real directory, you have to set up [path mappings](https://github.com/felixfbecker/vscode-php-debug#remote-host-debugging) for vscode-php-debug. This must be done for every line in your modman file, which might be very tedious, especially if you do it over and over because you add your .vscode directory to .gitignore, as I always do.

For this purpose, I have written a small python script that takes care of the linking stuff for me: I only have to run it with `modpath /path/to/shop` and that's all I need to start another debugging session.

I have saved the code in a public gist, just save it in your PATH and run it. :)

{% gist 4028f89b07e4ff56ddf406d910c08c58 %}
