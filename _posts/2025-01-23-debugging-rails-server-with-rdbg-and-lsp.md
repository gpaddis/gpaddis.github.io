---
layout: post
title: Debug requests with Ruby on Rails, rdbg, ruby LSP and VS Code
tags:
  - ruby
  - rails
  - debugging
---
While sending a GraphQL query to my local Ruby on Rails application, I ran into an error: the response was a 500 server error, without a stack trace. I tried commenting out all the rescue blocks I could find in the controllers, but the very cryptic error didn't change.

```
[b5f2e093-f1cd-478f-9c8c-946a722f0e1f] Redirected to
[b5f2e093-f1cd-478f-9c8c-946a722f0e1f] Completed 500 Internal Server Error in 4ms (ActiveRecord: 0.8ms | Allocations: 3899)
```

Since I didn't know where the error was being raised, I had to put the good old debugger back to work. In my experience, most Ruby on Rails developers (myself included) usually put a `byebug` / `debugger` before or after the line of code they want to inspect, especially when testing. I have also tried debugging Rails requests with VS Code in the past, but I quickly discarded the option since it was very slow and unresponsive.

Things have changed though, and now the combination of [the debug gem](https://github.com/ruby/debug), [ruby LSP](https://github.com/Shopify/ruby-lsp) and VS Code is powerful enough to use the debugging feature with Rails applications of any size. I found the [ruby LSP documentation](https://github.com/Shopify/ruby-lsp/blob/main/vscode/README.md#debugging-live-processes) and started a server with the debugger:

```
bundle exec rdbg -O -n -c -- bin/rails server -p 3000
```

The configuration for VS Code is set in `.vscode/launch.json`:

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "type": "ruby_lsp",
            "request": "attach",
            "name": "Attach to a debugger"
        }
    ]
}
```

You can then start the debugging session with `F5`, setting a check in the  `BREAKPOINTS > rescue any exception`  box. The request will stop on the line of code that raised the error, allowing you to inspect it in order to look for a solution. In my case, it was a missing `flash` method in the request, since I have an API-only application.

Happy debugging!