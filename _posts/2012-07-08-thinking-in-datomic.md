---
layout: post
title: "Thinking in Datomic"
tagline: "Your data is not square"
description: "Datomic is a very new take on databases that requires changing a bit how you think about your data"
category: Datomic
tags: [datomic, datoms, modelling]
---
{% include JB/setup %}
[Datomic](http://datomic.com) is so different than regular databases that your average developer will probably chose to ignore it. But for the developer and startup who takes the time to understand it properly I think it can be a real unfair advantage as a choice for a data layer in your application.

In this article I will deal with the core fundamental definition of how data is stored in Datomic. This is very different from all other databases so before we even deal with querying and transactions I think it's a good idea to look at it.

## Datoms are facts about entities not tables

When you initially look at Datomic you can be fooled into thinking that it is just a better relational database as the query language makes it look that way. 

In fact it is much more similar to the RDF concept of triples, which consist of entities, attribute keys and values. 

Datomic's core data element is the *datom* which is like a triple but adds time to it, so I guess you could say it uses Quadruples.

A datom could look like this:

<table class="table table-striped table-bordered"><tr><th>Entity</th><th>Attribute</th><th>Value</th><th>Time</th></tr>
  <tr><td><code>123</code></td><td><code>:person/name</code></td><td><code>Pelle</code></td><td><code>2012-07-08T20:19:03.176-00:00</code></td></tr></table>

## Entities are not rows

An entity is a collection of related facts. An entity is created by creating an entity id and mapping facts to it. These facts can change through time and datomic indexes and remembers this.

Stuart Halloway from Datomic likes to point out that in real life data is not square. A person table in a SQL database is a large square containing the same shaped facts about everyone.

So while loosely speaking you can think of Datomic's entity id as a primary key in a relational table. Instead of just mapping the columns in that table to the id you can match any attribute in your schema to it.

I've been doing a lot of travelling the last half year so I could create an entity representing me at the beginning of the year:

{% highlight clojure %}
{:db/id 123 :person/name "Pelle Braendgaard" :location/city "Miami, FL" :contact/phone "+1 305-555-5555"}
{% endhighlight %}

This adds 3 facts about my entity. My name, location and phone number.

<table class="table table-striped table-bordered"><tr><th>Entity</th><th>Attribute</th><th>Value</th><th>Time</th></tr>
  <tr><td><code>123</code></td><td><code>:person/name</code></td><td><code>Pelle</code></td><td><code>1970-08-11T20:19:03.176-00:00</code></td></tr>
  <tr><td><code>123</code></td><td><code>:location/city</code></td><td><code>Miami, FL</code></td><td><code>2012-01-01T00:00:00.000-00:00</code></td></tr>
  <tr><td><code>123</code></td><td><code>:contact/phone</code></td><td><code>+1 305-555-5555</code></td><td><code>2012-01-01T00:00:00.000-00:00</code></td></tr>
</table>


When I moved to Santiago, Chile in late January I added new facts:

{:lang='clojure'}
    {:db/id 123 :location/city "Santiago, Chile" :contact/phone "+56 9999 9999"}

This doesn't change the fact that I lived in Miami before then. It just adds new facts to me. That I now lived in Santiago, Chile and had a new phone number.

So Datomic would now have the following datoms about me:

<table class="table table-striped table-bordered"><tr><th>Entity</th><th>Attribute</th><th>Value</th><th>Time</th></tr>
  <tr><td><code>123</code></td><td><code>:person/name</code></td><td><code>Pelle</code></td><td><code>1970-08-11T20:19:03.176-00:00</code></td></tr>
  <tr><td><code>123</code></td><td><code>:location/city</code></td><td><code>Miami, FL</code></td><td><code>2012-01-01T00:00:00.000-00:00</code></td></tr>
  <tr><td><code>123</code></td><td><code>:contact/phone</code></td><td><code>+1 305-555-5555</code></td><td><code>2012-01-01T00:00:00.000-00:00</code></td></tr>
  <tr><td><code>123</code></td><td><code>:location/city</code></td><td><code>Santiago, Chile</code></td><td><code>2012-01-24T00:00:00.000-00:00</code></td></tr>
  <tr><td><code>123</code></td><td><code>:contact/phone</code></td><td><code>+56 9999 9999</code></td><td><code>2012-01-24T00:00:00.000-00:00</code></td></tr>
</table>


At any point I can check where did I live at the beginning of the year. This is always available.

I won't get into how to querying this yet, but have a look at [Datomic's Querying documentation](http://datomic.com/company/resources/query) to wet your beak.

## Thinking outside the square

We are so used to putting related data in these large squares that it can take a bit of time to understand that you don't need to do this anymore.

Datomic namespaces the attributes as a convenience to your application. Instead of creating a schema for a table organize related attributes within the same name space.

## Duck typed data

For example you could create a <code>:person/name</code> attribute. But you could also save yourself some repetition and create a general name attribute lets say <code>:general/name</code> that could be used to name any kind of entity. 

In my application I have people, organizations, apps and many other kind of entities. By having a generic name attribute I can share code. It is irrelevant that the name is for an organization, app or a person.

In some respects you can think of this as duck typing data. If an entity has a name it is a named entity.

Datomic already has one such attribute that it uses in it's schema amongst other things called <code>:db/doc</code>. I'm using this as a generic "description" attribute for my entities. So instead of adding a description column to 15 different tables as I might do in a SQL schema I just add a <code>:db/doc</code> value to any entities that need a human readable and changeable description.

In general when modelling your data and you see various patterns repeat try to name space the attributes and reuse them like you would a Mixin in Ruby code.

### Future posts

In future posts I probably wont try to duplicate any of [Datomic's own documentation](http://datomic.com/company/resources/getting-started) but focus on small areas that I've found useful in getting my mind around Datomic.
