---
layout: post
title:  "Setup path mappings for Xdebug with modman and VS Code"
tags: magento1 xdebug debugging
---

I still have a lot to do with **Magento 1** modules in my daily business, so I often have to turn to Xdebug to check out what's going on in the code. Unfortunately, modules are usually *symlinked* in the Magento `app/code/*` directory: I never open the symlinked directory in my VS code to avoid a version control mess.

To be able to debug working in the real directory, you have to setup [path mappings](https://github.com/felixfbecker/vscode-php-debug#remote-host-debugging) for vscode-php-debug. This must be done for every line in your modman file, which might be very tedious, especially if you do it over and over because you add your .vscode directory to .gitignore, as I always do.

For this purpose, I have written a small python script that takes care of the linking stuff for me: I only have to run it with `modpath /path/to/shop` and I am ready for another exciting debugging session!

I have saved the code in a public gist, just save it in your PATH and run it. :)

{% gist 4028f89b07e4ff56ddf406d910c08c58 %}
