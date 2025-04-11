---
layout: post
title:  "Rails with Inertia.js and React"
tags: rails frontend
---

Every web developer knows the many questions and frustrations that arise when starting to build a full-stack application. Should I write the front-end using the tools available in the back-end repository? Should I try to avoid Javascript, or just use a little here and there? Would it be better to write a SPA instead?

Every decision has a cost, and sometimes avoiding an API and two separate applications for back-end and front-end means sacrificing interactivity and complex features in the front-end. Rails suggests Hotwire as a way to get rid of the complexity, but there's more out there, and I've found that **Inertia.js** with **React** on **Rails** works best for my development workflow.
## The glue between back-end and front-end: Inertia

As described in the [official documentation](https://inertiajs.com/):

> Inertia isn't a framework, nor is it a replacement for your existing server-side or client-side frameworks. Rather, it's designed to work with them. Think of Inertia as glue that connects the two. Inertia does this via adapters.

Inertia was originally written for Laravel by [Jonathan Reinink](https://github.com/reinink), who is also one of the original authors of [Tailwind css](https://tailwindcss.com/). The community created pretty soon an adapter for [Ruby on Rails](https://github.com/inertiajs/inertia-rails).

Since I always liked the concept and the way the integration with Laravel just works out of the box, I have been trying to use Inertia.js in my Rails projects several times. In the end, I always gave up after spending too much time trying to fix obscure bugs, as the Rails adapter documentation was written for an older version of the framework than the one I was using. 

This changed completely with the publication of a new, extensive and detailed [documentation](https://inertia-rails.dev/guide) for the new adapter versions, which is fully compatible with the latest Rails versions. Today, starting a project with Inertia is super easy.

Here are some of the features that I enjoy the most, and a couple of tips for the configuration of your Rails application.

## Using Hotwire and Inertia

In my experience, even though Hotwire and Inertia may coexist in the same application, it is a good idea to keep them in separate layouts. This can be done very easily by creating two different layouts where you load one tool instead of the other. Inertia [can be configured](https://inertia-rails.dev/guide/server-side-setup) in an initializer to load a default layout:

```ruby
# config/initializers/inertia_rails.rb

InertiaRails.configure do |config|
  # Other configuration...
  config.layout = "inertia"
end
```

In this case, the `app/views/layouts/inertia.html.erb` layout will contain only the tags for Inertia and Vite, but not those for Hotwire. The routes rendered with Inertia will use the right template automatically.

## Lazy data evaluation on partial reloads

Docs: [Partial reloads](https://inertia-rails.dev/guide/partial-reloads#lazy-data-evaluation)

```ruby
render inertia: true, props: {
  profile: -> { Current.user.profile },
  posts: Current.user.posts.includes(:comments)
}
```
