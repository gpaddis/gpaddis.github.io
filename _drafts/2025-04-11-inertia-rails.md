---
layout: post
title:  "Rails with Inertia.js and React"
tags: rails frontend
---

Each time I wanted to start working on a new a web application, I was confronted with the same dilemma. Should I write the front-end using the server-side templating engine? Should I try to avoid Javascript, or just use a little here and there if I need some interactivity? Would it be better to write a SPA with React or Vue instead?

If you decided to use a modern javascript framework, in the last decade, the market seemed to point only in one direction: build an API and a SPA on top of it. Very often, this lead to complicated communication issues between back-end and front-end, both in the tech stack and across the teams.

New versions of Rails are built as a response against this complexity. Hotwire is a way to get back to the philosophy of the [one person framework](https://world.hey.com/dhh/the-one-person-framework-711e6318), but there's more out there, and I've found that **Inertia.js** with **React** on **Rails** works best for my development workflow.

## The glue between back-end and front-end: Inertia

As described in the [official documentation](https://inertiajs.com/):

> Inertia isn't a framework, nor is it a replacement for your existing server-side or client-side frameworks. Rather, it's designed to work with them. Think of Inertia as glue that connects the two. Inertia does this via adapters.

Inertia was originally written for Laravel by [Jonathan Reinink](https://github.com/reinink), who is also one of the original authors of [Tailwind css](https://tailwindcss.com/). The community created pretty soon an adapter for [Ruby on Rails](https://github.com/inertiajs/inertia-rails).

I always liked the concept and the way the integration with Laravel just works out of the box, so I have been trying to configure Inertia.js in my Rails projects several times. In the end, I always gave up after spending too much time trying to fix obscure bugs, as the Rails adapter documentation was written for an older version of the framework than the one I was using. 

This changed completely with the publication of a new, extensive and detailed [documentation](https://inertia-rails.dev/guide) for the new adapter versions, which is fully compatible with the latest Rails versions. Today, starting a project with Inertia is super easy.

Here are some of the features that I enjoy the most, and a couple of tips for the configuration of your Rails application.

## Using Hotwire and Inertia

If you want to start a new project with Inertia, the best option is to `--skip-javascript`and manage your assets with [`vite_rails`](https://github.com/ElMassimo/vite_ruby/tree/main/vite_rails). However, you might already have a Rails application running with Hotwire and want to try out Inertia, but only on certain pages.

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

## Less N+1

This is nothing new, since it is the same with every frontend application consuming an API, but I noticed that I have more confidence manipulating the data in the views if I am using javascript objects instead of hot ActiveRecord instances. I can optimize the queries in the controller and preload everything I need, being sure that I can't add a call to a model association and introduce a N+1 issue by mistake in my ERB view. Yes, you could always catch them with [Rails Debugbar](https://debugbar.dev/), but it is still nice to have immutable objects that you can transform the way you need without any fear.

## Further Resources

https://evilmartians.com/chronicles/inertiajs-in-rails-a-new-era-of-effortless-integration