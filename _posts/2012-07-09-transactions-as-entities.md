---
layout: post
title: "Datomic transactions as entities"
tagline: "not just meta"
description: "Transactions in Datomic are very powerful and are data entities in their own right"
category: Datomic
tags: [datomic, transactions]
---
{% include JB/setup %}
In the previous post you saw [how datomic data is made up of facts](http://pelle.github.com/Datomic/2012/07/08/thinking-in-datomic/). In this post I will discuss how you add facts to Datomic using transactions.

Just like Clojure code is just data, datomic transactions are also just data. They consist of collections of facts. This is very useful as there is not much to learn. You don't have to learn any particular language just a data format.

I won't go in to detail about how Datomic works, but for running transactions there is a single transactor responsible for running the transactions. For local dev use this is done in memory in the same process. See [Datomic's Architecture page](http://docs.datomic.com/architecture.html) for more.

## The basics

Here is the example I gave in the last article of facts about me:

{% highlight clojure %}
{:db/id 123 :person/name "Pelle Braendgaard" :location/city "Miami, FL" :contact/phone "+1 305-555-5555"}
{% endhighlight %}

I add this to the database like this:

{% highlight clojure %}
(datomic.api/transact conn
    [{:db/id #db/id[:db.part/user -1000001] :person/name "Pelle Braendgaard" :location/city "Miami, FL" :contact/phone "+1 305-555-5555"}])
{% endhighlight %}

Note as I'm creating a new entity I don't have an entity id. I need to create a temporary entity id which is what <code>#db/id[:db.part/user -1000001]</code> does. Don't worry to much about it, there are various ways of doing it, but none are important right now. I spent too much time being confused by this in the beginning.

I can modify an entity by adding new facts

{% highlight clojure %}
(datomic.api/transact conn
    [{:db/id 123123 :location/city "Santiago, Chile" :contact/phone "+56 9999 9999"}])
{% endhighlight %}

I could also have done this by adding individual attributes:

{% highlight clojure %}
(datomic.api/transact conn
    [[:db/add 123123 :location/city "Santiago, Chile"]
     [:db/add 123123 :contact/phone "+56 9999 9999"]])
{% endhighlight %}

You can retract a datom. This doesn't actually remove it, it just marks that fact as no longer valid.

{% highlight clojure %}
(datomic.api/transact conn
    [[:db/retract 123123 :contact/phone]])
{% endhighlight %}

You can also retract a whole entity, which retracts all facts about an entity:

{% highlight clojure %}
(datomic.api/transact conn
    [[:db/retractEntity 123123]])
{% endhighlight %}

You can still query past database values and retrieve it.

## Creating multiple related entities

Since the transactions just consist of a vector of facts you can easily create multiple entities at once:

{% highlight clojure %}
(datomic.api/transact conn
    [{:db/id #db/id[:db.part/user -1000001] :person/name "Pelle Braendgaard" :location/city "Miami, FL" :contact/phone "+1 305-555-5555"}
     {:db/id #db/id[:db.part/user -1000002] :person/name "Bob Smith" :location/city "Coral Gables, FL" :contact/phone "+1 305-555-9999"}])
{% endhighlight %}

If I wanted to relate them to each other I can use a reference type. I won't go into details on schema yet. But see [Datomic's Schema documentation](http://docs.datomic.com/schema.html) for more.

I start out by creating a temporary id for each entity outside the transaction so I can use them to reference each other. You create this temporary id specifying the partition your data is in. While your just playing use <code>:db.part/user</code>. I'm still exploring the benefits of creating multiple partitions and will write that up later.

{% highlight clojure %}
(let [ pelle (datomic.api/tempid :db.part/user)
       bob (datomic.api/tempid :db.part/user) ]

  (datomic.api/transact conn
    [{:db/id pelle :person/name "Pelle Braendgaard" :location/city "Miami, FL" :contact/phone "+1 305-555-5555" :role/friends bob}
     {:db/id bob :person/name "Bob Smith" :location/city "Coral Gables, FL" :contact/phone "+1 305-555-9999" :role/friends pelle}])
{% endhighlight %}

## Database functions

Database functions are clojure or java functions you add to the schema of the database. I haven't touched on the schema yet but here is a very simple example of a function that increments an attribute:

{% highlight clojure %}
{ :db/id #db/id [:db.part/user]
    :db/ident :inc
    :db/fn #db/fn { :lang "clojure"
                    :params [db id attr amount]
                    :code "(let [ e (datomic.api/entity db id)
                                  orig (attr e 0) ]
                            [[:db/add id attr (+ orig amount) ]])"}}
{% endhighlight %}

The clojure function it installs is basically:

{% highlight clojure %}
(defn inc [db id attr amount]
  (let [ e (datomic.api/entity db id)
              orig (attr e 0) ]
        [[:db/add id attr (+ orig amount) ]]))
{% endhighlight %}

It is based the value of the database as it is at the moment of the transaction and then any other parameters you want to pass it. It should return a vector of data elements just like you would when creating a transaction manually. They can also call other functions.

This is can be called by adding it to your transactional data like this:

{% highlight clojure %}
(datomic.api/transact conn
    [[:inc 123123 :person/age 1]])
{% endhighlight %}

Database functions are especially needed when you want to create facts based on existing facts. For example increasing a value in the database.

You could also use it for validation. In this case you throw an exception in your function if something is invalid. The whole transaction will rollback.

It may also be a neat way of abstracting out the creation of common elements within a transaction.

## Transactions as entities

Each transaction has an entity created for it. This by default just has a :db/txInstant timestamp attribute value. But you can add as much information about your transaction that you wish.

This can be useful for auditing by adding user ids, ip addresses etc. But taking it to the extreme lets look at this simple bank application where you transfer funds from one account to the other.

{% highlight clojure %}
(let [txid (datomic.api/tempid :db.part/tx)]
    (datomic.api/transact conn [[:transfer from to amount]
                      {:db/id txid, :db/doc note :ot/from from :ot/to to :ot/amount amount}]))
{% endhighlight %}

The <code>[:transfer ...]</code> section is a database function performing a transfer between accounts.

The important thing here is that we create a new tempid in the <code>:db.part/tx partition</code>. We can add as many facts as we want to this id which will represent the transaction.

This can be queried as if it was just a regular entity:

{% highlight clojure %}
(datomic.api/q '[:find ?tx :in $ :where [?tx :ot/amount _]] (datomic.api/db conn))
{% endhighlight %}

The above returns any entity with a <code>:ot/amount</code> attribute.

This transaction anotation is extremely powerful. Instead of creating your own activity tables you can just annotate the transaction instead.

## Complete example

I've written a [complete example showing most uses of Datomic transactions in a bank like application](https://gist.github.com/2635666) that you can look at.

The [schema](https://gist.github.com/2635666#file_schema.dtm) contains 2 database functions <code>:transfer</code> and <code>:credit</code>. <code>:transfer</code> calls <code>:credit</code> on two accounts with oposite amounts. <code>:credit</code> throws an exception to ensure sufficient funds in an account.

The [transfer function](https://gist.github.com/2635666#L52) calls the above transfer function and annotates the transaction with information about the transaction.



