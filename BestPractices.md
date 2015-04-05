

# Registering Your Entities #

The first question you will have is "when and how should I register my entity classes?"  The obvious answer is to do it at application startup in a servlet context listener or an init servlet - wherever your application starts running.  However, there is an easier way:

## Use Your Own Service ##

This guarantees that your entities are registered before you use Objectify, but doesn't necessarily impact application startup for requests which do not access the datastore.

Instead of using the `ObjectifyService` static `ofy()` method, use your own.

```
public class OfyService {
    static {
        factory().register(Thing.class);
        factory().register(OtherThing.class);
        ...etc
    }

    public static Objectify ofy() {
        return ObjectifyService.ofy();
    }

    public static ObjectifyFactory factory() {
        return ObjectifyService.factory();
    }
}
```

Now in your code, import your own static `ofy()`:

```
import static com.yourcode.OfyService.ofy;

Thing th = ofy().load().type(Thing).id(123L).now();
```

Another advantage of doing this is that when you want to use [Wrappers](Wrappers.md), you can easily change the return type of your static `ofy()` to your own custom class.

## How NOT To Register Entities ##

You might think that you could register an entity as a static initializer for the entity class itself:

```
public class ThingA
{
    static { ObjectifyService.register(ThingA.class); }
    // ... the rest of the entity definition
}
```

This is dangerous!  Because Java loads (and initializes) classes on-demand, Objectify cannot guarantee that your class will be registered at the time that it is fetched from the database.  For example, suppose you execute a query that might return several different kinds of entities:

```
Query<Object> things = ofy().load().ancestor(someParent);  // could find both ThingA and ThingB entities
things.first().now();    // throws IllegalStateException!
```

When Objectify tries to reconstitute an object of type ThingA, it won't be able to because the ThingA class will not yet have been loaded and the static initializer will not have been called.  If your application actually does use a ThingA before this query is executed, it will work - and in fact, it may work 99.99% of the time.  But do you really want to hunt down mysterious IllegalStateExceptions 0.01% of the time?

## Automatic Scanning ##

Most J2EE-style frameworks, including Appengine's JDO/JPA system, do classpath scanning and can automatically register classes that have @Entity or other relevant annotations.  This is convenient and could easily be added to Objectify without changing a single source file.  There are, however, several reasons why this isn't part of the core:

  1. This feature requires either [Scannotations](http://scannotation.sourceforge.net/) or [Reflections](http://code.google.com/p/reflections/), bringing in 5-6 dependency jars.
  1. Developers would need to add a startup hook to your web.xml (a ServletContextListener) in order to trigger this scanning.
  1. Classpath scanning is **slow** because it opens each .class and .jar file in your project and processes every single class file with a bytecode manipulator.  For a moderately sized project this easily adds 3-5 seconds to your application initialization time.  That's 3-5 additional seconds that real-world users must sit waiting while your application cold-starts.

Of these issues, the last is the most fatal.  If you think "My application gets a lot of traffic!  I don't need to worry about cold starts!", you are overlooking the fact that App Engine starts and stops instances to meet demand all the time - at least one user somewhere is going to be affected on every spinup.  Plus this happens every time you redeploy your application!  There is no escaping cold-start time.

Furthermore, classpath scanning costs accumulate.  If you use other tools that perform classpath scanning (Weld, Spring, JAX-RS, etc), they each will also spend 3-5s scanning your jars.  It isn't hard to push your cold-start time into the tens of seconds.

That said, 3-5s might be reasonable for your specific project.  It should be very easy to add as your own ServletContextListener that calls Reflections and registers the @Entity classes.  Spring and other framework users should examine the [Extensions](Extensions.md).

# Use Batch Gets Instead of Queries #

In SQL, all data lives in tables and is accessed through queries.  It is best not to imagine the Appengine datastore this way - conceptually shift to thinking of the datastore as a key-value store that happens to also let you index and query some values.

The reason this shift is important is because your most effective tool when working with chunks of data is the batch load and save.  A batch get-by-key will fetch many entities in parallel; running individual queries for each would take a relative eternity.  Asynchronous queries can only provide limited help because GAE limits you to 10 concurrent requests.

Furthermore, a batch get-by-key can be efficiently cached.  Use Objectify's `@Cache` annotation and your load() may never need a trip to the datastore.

Of course, batch gets and queries are not necessarily fungible operations - but when they are, use a batch get.

# Use Indexes Sparingly #

Each index adds cost.  There is an up-front cost (two writes per newly-written single-property index; _four_ if you change an index value) plus ongoing storage fees.  For most GAE apps, most of your storage costs will be index data - it grows quickly.

Objectify does not index anything by default; you must explicitly request it with the `@Index` annotation.  Use it wisely.

# Interesting discussions related to Objectify #

  * IBM developerWorks' _Twitter Mining with Objectify-Appengine_, [part 1](http://www.ibm.com/developerworks/java/library/j-javadev2-13/index.html) and [part 2](http://www.ibm.com/developerworks/java/library/j-javadev2-14/index.html)

  * [Review of Objectify/Twig/SimpleDS](http://borglin.net/gwt-project/?page_id=604)
  * [Original release  announcement on GAE-Java](http://groups.google.com/group/google-appengine-java/browse_thread/thread/4467986eaf01788b/d3a1678a44242c25)
  * [Google I/O 2008 - Under the Covers of the Google App Engine Datastore](http://sites.google.com/site/io/under-the-covers-of-the-google-app-engine-datastore) (required watching for anyone that uses App Engine!)
  * [Google I/O 2009 - Scalable, Complex Apps on App Engine](http://www.youtube.com/watch?v=AgaL6NGpkB8)
  * [Differences between Twig and Objectify plus example of million user fanout](http://groups.google.com/group/google-appengine-java/browse_thread/thread/f20d922ffecb310c)
  * [Reducing start-up time guide](http://www.answercow.com/2010/03/google-app-engine-cold-start-guide-for.html)
  * [Recucing start-up time blog entry](http://turbomanage.wordpress.com/2010/03/26/appengine-cold-starts-considered/)
  * [David M. Chandler's blog posting about Objectify](http://turbomanage.wordpress.com/2010/01/28/simplify-with-objectify/) and [Objectify 2](http://turbomanage.wordpress.com/2010/02/09/generic-dao-for-objectify-2/)
  * [GWT Example App with Objectify](http://iqbalyusuf.wordpress.com/gwt-uibinder-with-jax-rs-jersey/)
  * [Example of a Cursor based IteratingTask base class](http://groups.google.com/group/objectify-appengine/msg/14a326058a0870be)
  * [Controlling user access to data with queries](http://groups.google.com/group/objectify-appengine/browse_thread/thread/afa0d43de5db483f)
  * [Jersey + Guice on Google App Engine Java](http://iqbalyusuf.wordpress.com/jersey-guice-on-google-app-engine-java/)
  * [How to emulate @Unique](http://groups.google.com/group/objectify-appengine/browse_thread/thread/25a9abfc86f8be51) and [a longer discussion](http://groups.google.com/group/objectify-appengine/browse_thread/thread/5edec724ca69719e/b7536d44d028ef02) of enforcing uniqueness.
  * [Simple tutorial for 'putting' data in the datastore using Objectify](http://www.fishbonecloud.com/2010/11/use-objectify-to-store-data-in-google.html)