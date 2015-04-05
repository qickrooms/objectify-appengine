# Version 4+ #

Version 4 and later of Objectify are published in Maven Central.  If you wish to download a jar or check for the latest version number, [view the repository here](http://search.maven.org/#search%7Cga%7C1%7Cg%3A%22com.googlecode.objectify%22).

To add Objectify to your Maven project, add this to your `pom.xml`:
```
  <dependencies>
    <dependency>
      <groupId>com.googlecode.objectify</groupId>
      <artifactId>objectify</artifactId>
      <version>check for latest version</version>
    </dependency>
  </dependencies>
```

# Version 3 #

Historic versions of Objectify are available in a special [Objectify Maven repository](http://maven.objectify-appengine.googlecode.com/git/).

**Note:  This repository will not be updated.  You should use v4 and Maven Central.**

To add Objectify v3.1 to your Maven project, add this to your `pom.xml`:
```
  <dependencies>
    <dependency>
      <groupId>com.googlecode.objectify</groupId>
      <artifactId>objectify</artifactId>
      <version>3.1</version>
    </dependency>
  </dependencies>

  <repositories>
    <repository>
      <id>objectify-appengine</id>
      <url>http://maven.objectify-appengine.googlecode.com/git/</url>
    </repository>
  </repositories>
```