Objectify is a Java data access API specifically designed for the Google App Engine datastore.  It occupies a "middle ground"; easier to use and more transparent than JDO or JPA, but significantly more convenient than the Low-Level API.  Objectify is designed to make novices immediately productive yet also expose the full power of the GAE datastore.

  * Objectify lets you persist, retrieve, delete, and query your own **typed objects**.
```
@Entity
class Car {
    @Id String vin; // Can be Long, long, or String
    String color;
}
  
ofy().save().entity(new Car("123123", "red")).now();
Car c = ofy().load().type(Car.class).id("123123").now();
ofy().delete().entity(c);
```
  * Objectify surfaces **all native datastore features**, including batch operations, queries, transactions, asynchronous operations, and partial indexes.
  * Objectify provides **type-safe key and query classes** using Java generics.
  * Objectify provides a **human-friendly query interface**.
  * Objectify can automatically **cache your data in memcache** for improved read performance.
  * Objectify can store polymorphic entities and perform **true polymorphic queries**.
  * Objectify provides a simple, **easy-to-understand transaction model**.
  * Objectify provides built-in facilities to **help migrate schema changes** forward.
  * Objectify provides **thorough documentation** of concepts as well as use cases.
  * Objectify has an **extensive test suite** to prevent regressions.

After you've banged your head against JDO and screamed "Why, Google, why??" enough times, start with the [Concepts](Concepts.md).

You may also wish to read the IBM developerWorks article _Twitter Mining with Objectify-Appengine_, [part 1](http://www.ibm.com/developerworks/java/library/j-javadev2-13/index.html) and [part 2](http://www.ibm.com/developerworks/java/library/j-javadev2-14/index.html).

# Downloads #

Version 4 and later of Objectify is released to the [Maven Central Repository](MavenRepository.md) and can be downloaded directly from there.  Older releases can be found in the Google Code Downloads.

# Github Mirror #

You can submit pull requests to the [official Objectify github repository mirror](https://github.com/stickfigure/objectify).

# Upgrading #

Are you looking for information on how to UpgradeVersion4ToVersion5 or UpgradeVersion3ToVersion4?