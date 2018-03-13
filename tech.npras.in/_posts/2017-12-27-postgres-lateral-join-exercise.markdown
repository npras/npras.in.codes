---
layout: post
title: "PostgreSQL notes: Writing lateral join for a Top-n usecase"
excerpt: "2 lateral joins combining 3 tables in a single query"
---

I'm currently reading the excellent [__Mastering PostgreSQL__](https://masteringpostgresql.com/) by Dimitri Fontaine. It had an exercise in testing out an MVP's usecase by writing sql queries.

The MVP is just the typical blog app. It has a list of `categories`. An `article` has a title, content and pubdate and also belongs to any one category. An article can have many `comments`.

And an usecase is to list the most recent articles per category with the latest three comments on each article.

The output should look like this:

```
 category  │             pubdate              │                         title                          │                        three_comments
════════════╪══════════════════════════════════╪════════════════════════════════════════════════════════╪═══════════════════════════════════════════════════════════════
 box office │ 2018-01-27 09:34:33.545312+05:30 │ Ut Distinctio Non Vero Sequi                           │ [                                                            ↵
            │                                  │                                                        │     {                                                        ↵
            │                                  │                                                        │         "content": "Quis dolorum placeat fuga...",           ↵
            │                                  │                                                        │         "comment_pubdate": "2018-01-27T10:58:22.545312+05:30"↵
            │                                  │                                                        │     },                                                       ↵
            │                                  │                                                        │     {                                                        ↵
            │                                  │                                                        │         "content": "sequi natus magna quam qu...",           ↵
            │                                  │                                                        │         "comment_pubdate": "2018-01-26T19:48:49.545312+05:30"↵
            │                                  │                                                        │     },                                                       ↵
            │                                  │                                                        │     {                                                        ↵
            │                                  │                                                        │         "content": "id similique dolor autem ...",           ↵
            │                                  │                                                        │         "comment_pubdate": "2018-01-25T08:00:51.545312+05:30"↵
            │                                  │                                                        │     }                                                        ↵
            │                                  │                                                        │ ]
 box office │ 2018-01-27 02:28:29.545312+05:30 │ Officia Vero Libero Dignissimos Maxime                 │ [                                                            ↵
            │                                  │                                                        │     {                                                        ↵
            │                                  │                                                        │         "content": "repellat nihil consequat ...",           ↵
            │                                  │                                                        │         "comment_pubdate": "2018-01-25T11:29:39.545312+05:30"↵
            │                                  │                                                        │     },                                                       ↵
            │                                  │                                                        │     {                                                        ↵
            │                                  │                                                        │         "content": "corrupti blanditiis ad qu...",           ↵
            │                                  │                                                        │         "comment_pubdate": "2018-01-22T11:35:08.545312+05:30"↵
            │                                  │                                                        │     },                                                       ↵
            │                                  │                                                        │     {                                                        ↵
            │                                  │                                                        │         "content": "architecto impedit occaec...",           ↵
            │                                  │                                                        │         "comment_pubdate": "2018-01-19T06:44:02.545312+05:30"↵
            │                                  │                                                        │     }                                                        ↵
            │                                  │                                                        │ ]
 music      │ 2018-01-27 08:05:33.545312+05:30 │ Minus Distinctio Molestias Error Voluptates            │ [                                                            ↵
            │                                  │                                                        │     {                                                        ↵
            │                                  │                                                        │         "content": "blanditiis iure repellat ...",           ↵
            │                                  │                                                        │         "comment_pubdate": "2018-01-27T06:40:10.545312+05:30"↵
            │                                  │                                                        │     },                                                       ↵
            │                                  │                                                        │     {                                                        ↵
            │                                  │                                                        │         "content": "dicta sit possimus nisi a...",           ↵
            │                                  │                                                        │         "comment_pubdate": "2018-01-24T00:37:09.545312+05:30"↵
            │                                  │                                                        │     },                                                       ↵
            │                                  │                                                        │     {                                                        ↵
```

Here's the query that uses 2 lateral joins to get the desired result.

```sql
\set articles_count 2
\set comments_count 3

select
    category.name as category,
    lat_article.pubdate as pubdate,
    lat_article.title as title,
    jsonb_pretty(lat_article.agg_comments) as three_comments
  from sandbox.category
    left join lateral
    (
      select
          article.title,
          article.pubdate,
          jsonb_agg(lat_comment) as agg_comments
        from sandbox.article
          left join lateral
          (
            select
                comment.pubdate as comment_pubdate,
                substring(comment.content from 1 for 25) || '...' as content
              from sandbox.comment
              where comment.article = article.id
              order by comment.pubdate desc
              limit :comments_count
          ) as lat_comment on true
        where article.category = category.id
        group by article.id
        order by article.pubdate desc
        limit :articles_count
    ) as lat_article on true
  order by category.name, lat_article.pubdate desc
;
```

The thought process to arrive at this query might be something like this:

* write the sql output that you want to see, and build the query based on that.
* list all categories first, in the order that you want. This forms the outer query,
* for each category listed, list the most recent 2 articles. Use left join lateral to get this. This forms the first lateral join part of the above query,
* for each article listed, list the most recent 3 comments. Again use left join lateral to get this. And this forms the second lateral join.
