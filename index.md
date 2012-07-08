---
layout: page
title: Pelle Braendgaard
tagline: Clojure, OAuth, Ruby and Datomic developer

---
{% include JB/setup %}

## Clojure

[Clojure](http://clojure.org) is a lisp like language running on the JVM. Clojure is one of the first languages where time and how data changes over time is a key factor.

To me this makes it almost uniquely suitable for writing financial applications, where time is also one of the most important factors.

I come to Clojure from the Ruby world where I have lived since 2004 (previously I worked in Java, Perl and C).

* [clauth](http://pelle.github.com/clauth) a library for exposing your applications api through OAuth 2.
* [oauthentic](http://github.com/pelle/oauthentic) a library for integrating your application with another api through OAuth2.
* [Bux](http://pelle.github.com/bux/) a money and currency manipulation library for clojure
* [slugger](https://github.com/pelle/slugger) a smart Unicode text to 7-bit slug generator similar to Stringex in Ruby.

## Datomic

[Datomic](http://datomic.com) is the database from Rich Hickey the original creator of Clojure. It brings many of his ideas of immutability and time to the database world. This is very new but is very different than both traditional relational databases and more modern NoSQL database. It has the potential to really revolutionize data in large scale financial apps.

I will be highlighting my own experiments with it here.

## OAuth

Security is an important part of financial applications. I was one of the earliest ruby developers to focus on OAuth and other security aspects. I've written a number of OAuth libraries for both Ruby and Clojure:

Clojure:

* [Clauth](http://pelle.github.com/clauth) a library for exposing your applications api through OAuth 2.
* [Oauthentic](http://github.com/pelle/oauthentic) a library for integrating your application with another api through OAuth2.

Ruby:

* [OAuth Ruby Gem](https://github.com/oauth/oauth-ruby) The original OAuth 1 Ruby Gem. I wrot it originally but no longer maintain it
* [Rails OAuth Plugin](https://github.com/pelle/oauth-plugin) Rails plugin for creating an OAuth provider and consumer (Supports OAuth1 and Oauth2)

## Posts

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>
