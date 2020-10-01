---
layout: post
title:  "Counting related models without N+1 and counter cache"
tags: rails laravel ruby
---

<!--
# N+1 problem
# Rails counter cache
# SQL solution
# Module
 -->

Example: a `ForumThread` has many associated replies. If we want to render a list of forum threads and display the number of replies present for each post, we can do it like this:

```ruby
irb(main):017:0> ForumThread.unscoped.map {|t| t.replies.size }
ForumThread Load (0.2ms) SELECT "forum_threads".* FROM "forum_threads"
(0.1ms) SELECT COUNT(*) FROM "replies" WHERE "replies"."forum_thread_id" = ? [["forum_thread_id", 2]]
(0.1ms) SELECT COUNT(*) FROM "replies" WHERE "replies"."forum_thread_id" = ? [["forum_thread_id", 3]]
(0.1ms) SELECT COUNT(*) FROM "replies" WHERE "replies"."forum_thread_id" = ? [["forum_thread_id", 4]]
(0.1ms) SELECT COUNT(*) FROM "replies" WHERE "replies"."forum_thread_id" = ? [["forum_thread_id", 5]]
(0.1ms) SELECT COUNT(*) FROM "replies" WHERE "replies"."forum_thread_id" = ? [["forum_thread_id", 6]]
(0.1ms) SELECT COUNT(*) FROM "replies" WHERE "replies"."forum_thread_id" = ? [["forum_thread_id", 7]]
(0.1ms) SELECT COUNT(*) FROM "replies" WHERE "replies"."forum_thread_id" = ? [["forum_thread_id", 8]]
(0.1ms) SELECT COUNT(*) FROM "replies" WHERE "replies"."forum_thread_id" = ? [["forum_thread_id", 9]]
(0.1ms) SELECT COUNT(*) FROM "replies" WHERE "replies"."forum_thread_id" = ? [["forum_thread_id", 10]]
(0.1ms) SELECT COUNT(*) FROM "replies" WHERE "replies"."forum_thread_id" = ? [["forum_thread_id", 11]]
(0.1ms) SELECT COUNT(*) FROM "replies" WHERE "replies"."forum_thread_id" = ? [["forum_thread_id", 12]]
(0.1ms) SELECT COUNT(*) FROM "replies" WHERE "replies"."forum_thread_id" = ? [["forum_thread_id", 13]]
(0.1ms) SELECT COUNT(*) FROM "replies" WHERE "replies"."forum_thread_id" = ? [["forum_thread_id", 14]]
(0.1ms) SELECT COUNT(*) FROM "replies" WHERE "replies"."forum_thread_id" = ? [["forum_thread_id", 15]]
(0.1ms) SELECT COUNT(*) FROM "replies" WHERE "replies"."forum_thread_id" = ? [["forum_thread_id", 16]]
(0.1ms) SELECT COUNT(*) FROM "replies" WHERE "replies"."forum_thread_id" = ? [["forum_thread_id", 17]]
(0.1ms) SELECT COUNT(*) FROM "replies" WHERE "replies"."forum_thread_id" = ? [["forum_thread_id", 19]]
(0.1ms) SELECT COUNT(*) FROM "replies" WHERE "replies"."forum_thread_id" = ? [["forum_thread_id", 20]]
(0.1ms) SELECT COUNT(*) FROM "replies" WHERE "replies"."forum_thread_id" = ? [["forum_thread_id", 21]]
=> [15, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 11, 12, 11, 3, 1]
```

This performs the ForumThread query + N queries for each replies count, which can quickly become a lot of queries if we have many posts to display and no pagination.

Rails' built-in solution to the problem is the [counter cache](https://guides.rubyonrails.org/association_basics.html#options-for-belongs-to-counter-cache). However, the counter cache requires additional setup (you must add a new database column), the value must be initialized... there are some downsides.

What I was missing is Laravel's method [withCount](https://laravel.com/docs/8.x/eloquent-relationships#counting-related-models). You can pass the name of a relation while loading a collection and it will add the count of the association as an additional model attribute. The whole thing is translated to a single SQL query, which avoids N+1 issues.

Looking for a similar solution, I found an interesting [Medium post by Eric Anderson](https://medium.com/@eric.programmer/the-sql-alternative-to-counter-caches-59e2098b7d7). I decided to tweak the logic a bit to make it available as a concern with a flexible scope that I could include in any model where I wanted to load a dynamic count of the relations. The API is simple: just include `WithCounts` in a model and load the collection with the method `with_counts(:relation1, :relation2)`:

```ruby
irb(main):018:0> ForumThread.unscoped.with_counts(:replies).map(&:replies_count)
ForumThread Load (0.4ms) SELECT forum_threads.*, (SELECT COUNT(replies.id) FROM replies WHERE forum_thread_id = forum_threads.id) AS replies_count FROM "forum_threads"
=> [15, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 11, 12, 11, 3, 1]
```

The dynamic `replies_count` attribute is added with a subquery, the database takes care of the calculation for us and does not perform additional queries, thus avoiding N+1 problems. At the moment, the concern can only calculate the relation count for *has_many* and *has_many polymorphic* relations, but I plan to extend it to work it with other relation types as well. I will update the gist accordingly.

The source code is available below.

{% gist ceab19bd7a0fc5e5cc53ee329ffda73f %}