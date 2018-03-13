---
layout: post
title: "PostgreSQL notes: Full text search"
excerpt: "Querying a document is different from querying a typical string or text data type"
---

I'm currently reading the excellent [__Mastering PostgreSQL__](https://masteringpostgresql.com/) by Dimitri Fontaine. A while ago HN was talking about full text search in the context of both Elasticsearch and postgresql.

PostgreSQL provides support to search documents. But the general consensus seems to be that it can only be used in cases where scaling is not required.

A document is the unit of searching in a full text search system; for example, a magazine article or email message.

Full Text Searching (or just text search) provides the capability to identify natural-language documents that satisfy a query, and optionally to sort them by relevance to the query.

Eg: in a library full of books, find the books that "mention about" a particular search term.
Eg: find all books about theft. The query should return books that have the word theft. But also the words synonyms and derivatives! stolen, robbery, abscond etc. And books that mention more of these should be ranked higher than the other ones that don't.

In Postgres it can be done using full text indexing.

Full text indexing allows documents to be preprocessed and an index saved for later rapid searching. Preprocessing includes: Parsing documents into tokens, Converting tokens into lexemes, Storing preprocessed documents optimized for searching.

A data type tsvector is provided for storing preprocessed documents, along with a type tsquery for representing processed queries.

---

Basic searching using `@@` match operator. which returns true if a tsvector (document) matches a tsquery (query). It doesn't matter which data type is written first.

```
SELECT 'a fat cat sat on a mat and ate a fat rat'::tsvector @@ 'cat & rat'::tsquery;
 ?column?
----------
 t

SELECT 'fat & cow'::tsquery @@ 'a fat cat sat on a mat and ate a fat rat'::tsvector;
 ?column?
----------
 f
```

---

__to_tsvector() and to_tsquery() functions:__

```
SELECT to_tsvector('fat cats ate fat rats') @@ to_tsquery('fat & rat');
 ?column? 
----------
 t
```

Notice that the document has only rats, but the query is for singular rat. It returns true because the to_tsvector function normalizes the lexemes. So rats will become as rat.

---

FOLLOWED BY tsquery operator:

```
SELECT to_tsvector('fatal error') @@ to_tsquery('fatal <-> error');
 ?column? 
----------
 t

SELECT to_tsvector('error is not fatal') @@ to_tsquery('fatal <-> error');
 ?column? 
----------
 f
```

---

__phrase\_totsquery()__

```
SELECT phraseto_tsquery('cats ate rats');
       phraseto_tsquery        
-------------------------------
 'cat' <-> 'ate' <-> 'rat'

SELECT phraseto_tsquery('the cats ate the rats');
       phraseto_tsquery        
-------------------------------
 'cat' <-> 'ate' <2> 'rat'
```

It converts the english phrase to tsquery will the followed by operators (<-> and <n>).

---

#### searching a table

__Query to print the title of each row that contains the word friend in its body field is:__

```sql
SELECT title
FROM pgweb
WHERE to_tsvector('english', body) @@ to_tsquery('english', 'friend');
```

This will also find related words such as friends and friendly, since all these are reduced to the same normalized lexeme.

__Select the ten most recent documents that contain create and table in the title or body:__

```sql
SELECT title
FROM pgweb
WHERE to_tsvector(title || ' ' || body) @@ to_tsquery('create & table')
ORDER BY last_mod_date DESC
LIMIT 10;
```
