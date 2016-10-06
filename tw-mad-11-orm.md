% MAD - Android 11: Android ORMs
% Patrick Sturm
% 06.10.2016

## Information

* Any issues with this presentation? Write a ticket or send me a pull request ;).
* Repo: [https://github.com/siyb/tw-mad-11-orm](https://github.com/siyb/tw-mad-11-orm)


# Agenda

## Agenda

* Introduction
* ORM / ORM Concepts
* greenDAO
* sugarorm
* Outlook
    * alternatives (ORM / Databases)

# Introduction

## Introduction - 1 - About the slides

* We will be looking at two different frameworks today
    * greenDAO
    * sugarorm
* Both Frameworks provide ORM functionality, but
    * greenDAO is more advanced (06.10.2016)
    * they use different approaches to implement ORM functionality
    * I personally like the greenDAO approach more, but sugar has its architectural gems

# ORM / ORM Concepts

## ORM / ORM Concepts - 1 - General Information

* ORM stands for Object Relational Mapp(ing/er)
* AKA OR Mapper
* Using ORMs can reduce the amount of code we have to write
    * Reducing the amount of manually written code decreases the amount of bugs we produce
        * Maintenance is “easier” and takes less time
    * It _can_ be more time- and therefore cost effective
* By using an ORM, we have a more native feel when using relational databases
* Also offers: database abstraction!

## ORM / ORM Concepts - 2 - Mapping Data

* ORM provides mappings between Object Oriented programming languages and Relational Databases
   * One-to-One 1:1
   * One-to-Many 1:n
   * Many-to-Many n:m
   * Others (e.g. difference between hasOne - composition and belongsTo - aggregation)
* In order to establish the mapping, tables are “translated” to Objects
   * The mapping can be established automatically …
   * … or manually
* The use of DAOs (Data Access Objects) to abstract database access
* Some languages (like Java) feature APIs that can be used to implement ORMs (JPA – Java Persistence API)

## ORM / ORM Concepts - 3 - Pattern: ActiveRecord

* Database record is contained within an Object
* The Object also provides methods to save, update, delete itself
* The Class contains methods to find instances as well
* Adds other domain logic
* => SRP? Not so much, data and logic are mixed!
* Example: Ruby On Rails ORM

## ORM / ORM Concepts - 4 - Pattern: Data Mapper

* Data Object and Database are separated
* Responsibilities are clearly defined
* Save / update / delete / find are not part of the entity
* Example: Hibernate / JPA - EntityManager

## ORM / ORM Concepts - 5 - Lazy Loading

* Most ORMs support lazy loading for different data types, e.g.:
    * A LazyList<Car> will not contain all cars queried from the database, each individual Car’s data will be queried once it is accessed
    * Relations of the Car might also be loaded lazily, i.e. when the program tries to access it
* Lazy loading is very important on mobile devices, since memory is limited!

## ORM / ORM Concepts - 6 - Data Model Generation

* ORMs can be used on existing databases – more work
* The best idea is to let the ORM create and manage the database accordingly
    * Adds another layer of abstraction, you don’t have any direct control over what the database will look like …
    * … but you can make a good assumption about tables (e.g. join tables)
* As stated above, the ORM will take care of managing the database
    * New tables
    * Database updates if new data types are added
    * Etc.

## ORM / ORM Concepts - 7 - Queries

* Simple Queries are easily executed – in most cases, there is no need to write SQL queries by hand, but …
    * ORMs generally support writing of SQL statements
    * In addition to SQL, ORMs often support so called DSLs (Domain Specific Language) that allow more complex database operations
    * These DSLs may have a completely different syntax than the programming language / query language you are actually working with
        * e.g. JPA criteria queries
    * Alternatively (or in addition), ORMs might provide builders that can be used to create queries, either by using the DAO directly and/or by providing dedicated query builder classes

## ORM / ORM Concepts - 8 - Writing

* Usually, inserting and updating records is accomplished using DAOs
    * An Object is queried from the data base
    * Changes are made using provided setter methods
    * The Object is inserted or updated depending on the programmers need
    * Beware, references might not be inserted / updated automatically, when inserting / updating an Object!

## ORM / ORM Concepts - 9 - Shortcomings / Criticism

* ORMs are a mere abstraction of the relational database
    * Object Orientation != Relational, mapping causes issues!
    * They introduce another layer of complexity
    * Even though this new layer of complexity is not a black box (in almost all cases), debugging can still be tedious, if the problem is situated within the guts of the ORM
* ORMs can be slow
    * Since data is automatically mapped, operations can be time consuming
* ORMs may hide functionality
    * -> Abstraction
    * Data types get abstracted, there is no way one can match all OOP data types to corresponding database types (especially with SQLite)
    * A list of additional reasons why mapping a relation database model to an OOP object model has some shortcomings (from a software engineering perspective) can be found [here](://en.wikipedia.org/wiki/Object-relational_impedance_mismatchhttp)

# greenDAO

## greenDAO - 1 - Introduction

* greenDAO – [http://greendao-orm.com](http://greendao-orm.com)
* greenDAO is an ORM which was written for Android
    * Therefore, it currently only supports SQLite (and I personally doubt that support for other databases will be added in the near future)
* As most Android related open source projects, greenDAO is licensed under the terms of the Apache License 2.0
* greenDAO uses the Data Mapper pattern

## greenDAO - 2 - Introduction cont.

* Design goals of greenDAO
    * Performance
    * Simple API
    * Optimized for Android
    * Small memory footprint
    * Small library size
    * OVERALL: designed for mobile use

## greenDAO - 3 - Introduction cont.

![greenDAO (source: http://greendao-orm.com/)](./greenDAO-orm-320.png)

## greenDAO - 4 - Getting Started

* greenDAO consists of two .jar files
    * The Android library: `compile 'de.greenrobot:greendao:2.0.0'`
    * The Generator library: `compile 'de.greenrobot:greendao-generator:2.0.0'`
* greenDAO uses the freemarker templating engine to generate code

## greenDAO - 5 - Getting Started cont.

* In order to use greeDAO, we need to create two separate projects:
* Generator project
    * The generator project is a standard Java project
    * It uses the greendao-generator-NNN.jar 
    * It takes care of generating source files (the data access layer) for our Android project
    * We need to define a database schema using Java code
    * Every time something changes, we need to rerun the generator project in order to recreate the source files
    * We can automate this process (ANT, Maven, etc)

## greenDAO - 6 - Getting Started cont.

* Android project:
    * Uses the greendao-NNN.jar
    * Should contain a source directory that is only used by greenDAO (e.g.): src/ -> src-greendao/
    * Every time the generator project is executed, .java files are generated and written to the specified folder
    * You need to refresh / rebuild once new .java files have been created
    * The Android project uses the generated .java files to access the database

## greenDAO - 5 - Example: Schema

```java
public class MyGenerator throws Throwable { 
  private static final int SCHEMA_VERSION = 1; 
  private Schema schema = new Schema(SCHEMA_VERSION,
    "my.greendao.example"
  ); 
  private void generateSchema() {
    Entity article = schema.addEntity("Article"); 
    article.addIdProperty(); 
    article.addBooleanProperty("read"); 
    article.addStringProperty("title").notNull(); 
    article.addStringProperty("description"); 
    article.addDateProperty("created"); 
    new DaoGenerator().generateAll(schema, "../path"); 
  }
  public static void main(String[] args) { 
    new MyGenerator().generateSchema(); 
  } 
} 
```

## greenDAO - 5 - Example: Schema Relationships

```java
Property feedId = article.addLongProperty("feedid")
    .getProperty(); 
Entity feed = schema.addEntity("Feed"); 
feed.addIdProperty(); 
feed.addStringProperty("title").notNull();
article.addToOne(feed, feedId);
```

## greenDAO - 6 - Relationships Explained

* Entities that wish to define relations have to define a field that corresponds to the relation
* In our case, article defines a field feedId to store the reference
* We need to set the relation, in our case to-one for our entity manually, to do this:
    * We need to provide the Entity to which we want to have a relation to
    * We need to provide a field that stores a reference (i.e. the id)
* Running this code will generate a lazy getFeed() method in our Article DAO (this method returns the Feed Object and not its id!)
* If we want to create more complex relations (n:m for instance) we have to create a to-many relation from Article to Feed and a to-many relation from Feed to Article

## greenDAO - 6 - Additional Schema Features

* There are many other options we cannot cover in our lecture, please refer to:
    * [more complex relations](http://greendao-orm.com/documentation/relations/)
    * [additional features like inheritance](http://greendao-orm.com/documentation/modelling-entities/)
    * Please note that n:m relations are not directly supported, you need to implement the join table manually and adjust relations accordingly

## greenDAO - 7 - Working With Data

* Now it’s time to run the generator project to create the glue / boilerplate code that we can use to access the database
* greenDAO creates two artifacts for each Entity
    * A data object that holds the actually data (e.g. Article)
    * A DAO object that is used to execute Entity specific operations on the database (e.g. ArticleDAO)
* But first, we have to create the database ;)

```java
new DaoMaster.DevOpenHelper(this, "test-db", null);
```

## greenDAO - 8 - Initializing DaoMaster / DaoSession

```java
private DaoMaster daoMaster; 
private DaoSession daoSession; 
private ArticleDao articleDao; 
private FeedDao feedDao; 

private void initDatabaseAccess() { 
  SQLiteDatabase db = new DaoMaster
    .DevOpenHelper(this, "notes-db", null) 
    .getWritableDatabase(); 
  daoMaster = new DaoMaster(db); 
  daoSession = daoMaster.newSession(); 
  articleDao = daoSession.getArticleDao(); 
  feedDao = daoSession.getFeedDao();
} 
```

## greenDAO - 9 - Inserting Data

```java
private void createSampleArticle() { 
  Feed f = new Feed(); 
  f.setTitle("Awesome Tech Site"); 
  feedDao.insert(f); 

  Article a = new Article(); 
  a.setTitle("Wow, nice title dude"); 
  a.setDescription("This is a fine article..."); 
  a.setRead(false); 
  a.setCreated(new Date()); 
  a.setFeed(f); 
  articleDao.insert(a); 
}
```

## greenDAO - 10 - More Write Operations

* The Entity DAOs feature multiple methods to interact with the Entity on the database level
    * insert(Entity)
    * update(Entity)
    * delete(Entity)
    * dropTable(…)
* In addition, they feature methods to execute CRUD operations within a transaction, e.g. deleteInTx(Entity)
* On top of all that, they facilitate query creation (both raw and ORM query) and they provide table and Entity meta data
* You can check all that stuff out yourself ;)

## greenDAO - 11 - Query Data

```java
private Article querySampleArticle() {
  QueryBuilder<Article> builder = articleDao.queryBuilder();
  builder.where(
    Properties.Title.eq("Wow, nice title dude"));
      builder.and(
        Properties.Description.eq(
          "This is a fine article..."), 
        Properties.Read.eq(false));
  List<Article> articles = builder.list();
}
```

## greenDAO - 12 - More On Querying

* Instead of using builder.list(), you may also use listLazy() which must be manually closed due to the nature of the implementation (use listLazy().close() when you are done)
* listLazy() is probably the best way to query data that needs to be displayed in an Android list
* joins are finally supported as well \o/

# [Example Code](https://github.com/SphericalElephant/android-example-greendao)

# sugarorm

## sugarorm - 1 - Introduction

* Can be obtained from [http://satyan.github.io/sugar/](http://satyan.github.io/sugar/)
* Designed for Android
* Not as advanced as greenDAO
* Does not work by generating code or using reflections but by inheritance
* Does not support 1:n or m:n relations out of the box, has to be manually implemented -.-
* Does not provide a "DSL" like greenDAO does, more crude queries
* Implements the ActiveRecord pattern

## sugarorm - 2 - Example: Manifest

* First of all, we need adjust our manifest

```xml
<!-- declare com.orm.SugarApp or
inheriting classes as application -->
<application android:name="com.orm.SugarApp">
<!-- the name of our database -->
<meta-data android:name="DATABASE"
android:value="sugardb-example.db"/>
<!-- our database schema version -->
<meta-data android:name="VERSION"
android:value="1"/>
<!-- write select queries to log (debugging) -->
<meta-data android:name="QUERY_LOG"
android:value="true"/>
<!-- the location of our entities,
sugar will take care of the rest -->
<meta-data 
android:name="DOMAIN_PACKAGE_NAME"
android:value="com.sphericalelephant.example.sugarorm.entity"/>
</application>
```

## sugarorm - 3 - Example: Model

```java
public class MyEntity extends SugarRecord<MyEntity> {
  private String myString;
  private int myInt;
  private Date myDate;
  private List<String>
    youWouldThinkThisWorksButItDoesnt;
}
```

## sugarorm - 4 - Example: Saving Entity

```java
MyEntity me = new MyEntity();
me.setMyString("Test");
me.setMyInt(1);
me.setMyDate(new Date());
me.save(); // me.delete();
```

## sugarorm - 5 - Example: Query Data

* supports additional find methods, autocomplete is your friend

```java
List<MyEntity> p = 
  MyEntity.find(
    MyEntity.class,
    "myString = ?", "woot!"
  );
```

## sugarorm - 6 - Example: 1:n

* Example for 1:n

```java
public MyOtherEntity getMyOtherEntity() {
  return MyOtherEntity.find(
    MyOtherEntity.class,
    "myEntity = ?", new String{getId()}
  );
}
```

# [Example Code](https://github.com/SphericalElephant/android-example-sugarorm)

# Outlook

## Outlook - 1 - More Android ORMs

* ActiveAndroid
    * Designed for Android
    * Similar to what sugarorm does
    * Very promising!
* ORMLight - http://ormlite.com/
    * Supports Android, is not made for Android
* androrm - http://androrm.com/
    * Designed for Android
    * Uses inheritance instead of generation
* Some more, use Google if you hadn’t enough ;)

## Outlook - 2 - Alternatives to ORM 

* NoSQL
    * Does not mean “NO!” SQL but Not Only SQL
    * But more often than not, NoSQL contains NO SQL ;)
* Possible NoSQL solutions on Android
* Berkeley DB
    * Key / Value store
* CouchDB
    * Document store
    * JSON API
* UnQLite
    * Key / Value store, Document store

# Any Questions?