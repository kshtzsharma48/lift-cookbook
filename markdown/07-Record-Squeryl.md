Relational database persistence with Record and Squeryl
=======================================================



Using a JNDI datasource
===========

Problem
-------

You want to use a JNDI data source for your Record+Squeryl Lift application.

Solution
--------

In `Boot.scala` call `initWithSquerylSession`...

```scala
import javax.sql.DataSource
val ds = new InitialContext().
  lookup("java:comp/env/jdbc/mydb").asInstanceOf[DataSource]

SquerylRecord.initWithSquerylSession(
  Session.create(ds.getConnection(), new MySQLAdapter) )
```

...replacing `mydb` with the name given to your database in your JNDI configuration, and replacing `MySQLAdapter` with the appropriate adapter for the database you are using.


Discussion
----------

The Java Naming and Directory Interface (JNDI) is service provided by the web container (e.g., Jetty, Tomcat, etc) which allows you to configure a database connection in the container and then refer the connection by name in your application. One advantage of this is that you can avoid including database credentials to your Lift source base.

The configuration of JNDI is different for each container, and may vary with versions of the container you use.  The _See Also_ section includes links to the documentation pages for popular containers.

Some environments may also require that you to mention the JNDI resource in your `src/main/webapp/WEB-INF/web.xml` file:

```xml
<resource-ref>
 <res-ref-name>jdbc/mydb</res-ref-name>
 <res-type>javax.sql.DataSource</res-type>
 <res-auth>Container</res-auth>
</resource-ref>
```

See Also
--------

* List of [Squeryl database adapters](http://squeryl.org/api/org/squeryl/adapters/package.html). 
* An example [Jetty JNDI configuration](http://www.assembla.com/spaces/liftweb/wiki/Apache_and_Jetty_Configuration) on the Lift wiki.
* [Jetty 6 JNDI documentation](http://docs.codehaus.org/display/JETTY/JNDI).
* Eclipse [Jetty JNDI documentation](http://wiki.eclipse.org/Jetty/Howto/Configure_JNDI_Datasource).
* [Tomcat 7](http://tomcat.apache.org/tomcat-7.0-doc/jndi-resources-howto.html) JNDI information.

Adding validation to a field
=============================

Problem
-------

You want to add validation to a field in your model.

Solution
--------

Override `validations`. For example:

```scala
val title = new StringField(this, 256) {
  override def validations = valMinLen(1, "Title cannot be blank") _ :: 
    super.validations
}
```

In your snippet you can check the validations, for example:

```scala
val thing = MyThing.createRecord.title(title)
thing.validate match {
  case Nil =>
    // No validation problems, so code here to save thing
    S.redirectTo("/success")
  case xs => // One or more validation problems! 
    S.error(xs)  
}
```

In your template, you can reference the column to show any error:

```html
<p class="lift:Msg?id=title_id&errorClass=error">Msg to appear here</p>
```

Discussion
----------

The built-in validations are:

* `valMinLen` -- validate a string is at least a given length, as shown above.
* `valMaxLen` -- validate that a string is not above a given length.
* `valRegex` -- validate a string matches the given pattern.

An example of regular expression validation would be:

```scala
import java.util.regex.Pattern

val url = new StringField(this, 1024) {
  override def validations = 
    valRegex( Pattern.compile("^https?://.*"), 
              "URLs should start http:// or https://") _ :: 
    super.validations
}
```


See Also
--------

* Source for [BaseField.scala](https://github.com/lift/framework/blob/master/core/util/src/main/scala/net/liftweb/util/BaseField.scala) which includes the definition of `StringValidators`.



Implementing custom validation logic
====================================

Problem
-------

You want to provide your own validation logic and apply it to a field in a record.

Solution
--------

Implement a function from the type of field you want to validate to `List[FieldError]`. Perhaps we want to ensure that no-one added to the database can have the same name, so we need to provide a `String => List[FieldError]` function:

```scala
class Person private () extends Record[Person] with KeyedRecord[Person] {

  override def meta = Person

  @Column(name = "id")
  override val idField = new LongField(this)
 
  val name = new StringField(this, 100) {
    override def validations = 
      valUnique("Please change your name") _ :: super.validations
  }
  
  def valUnique(errorMsg: ⇒ String)(name: String): List[FieldError] = 
    Person.byName(name) match {
      case Some(name) => FieldError(this.name, errorMsg) :: Nil
      case _ => Nil
  }

}
```


Discussion
----------

By convention validation functions have two argument lists: the first for the error message; the second to receive the value to validate.  This allows you to easily re-use your validation function on other fields.

The `FieldError` you return needs to know the field it applies to as well as the message to display.  In the example the field is `name`, but we've used `this.name` to avoid confusion with the `Some(name)` in the pattern match or the `name` passed as an argument to `valUnique`.

In case you're wondering, the implementation of `Person.byName` might be:

```scala
def byName(name: String) = 
  from(YourSchema.people)
  (p => where(lower(p.name) === lower(name)) 
  select (l)).headOption
```


See Also
--------

* Source for [BaseField.scala](https://github.com/lift/framework/blob/master/core/util/src/main/scala/net/liftweb/util/BaseField.scala) which includes the definition of `StringValidators`.

Modify a field value before it is set
=====================================

Problem
-------

You want to modify the value of a field, so the value in your model is the modified version.

Solution
--------

Override `setFilter`. For example, to remove leading and trailing whitespace entered by the user:

```scala
val title = new StringField(this, 256) {
   override def setFilter = trim _ :: super.setFilter
}
```

Discussion
----------

The built-in filters are:

* `crop` -- enforces the field's min and max length by truncation.
* `trim` -- applies `String.trim` to the field value.
* `toUpper` and `toLower` -- change the case of the field value.
* `removeRegExChars` -- removes matching regular expression characters.
* `notNull` -- coverts null values to an empty string.

See Also
--------

* Source for [BaseField.scala](https://github.com/lift/framework/blob/master/core/util/src/main/scala/net/liftweb/util/BaseField.scala) which includes the definition of the filters. 




Put a random value in a column
==============================

Problem
-------

You need a column to hold a random value.

Solution
--------

Use `UniqueIdField`: 

```scala
val myColumn = new UniqueIdField(this,32) {}
```

The size value, 32 in this example, controls the number of characters in the random field.


Discussion
----------

The field is a kind of `StringField` and the default value for the field comes from `StringHelpers.randomString`.  

Note the `{}` in the example: this is required as `UniqueIdField` is an abstract class.


See Also
--------

* Source for [StringHelpers](https://github.com/lift/framework/blob/master/core/util/src/main/scala/net/liftweb/util/StringHelpers.scala).




Automatic created and updated timestamps for a Squeryl Record
=============================================================

Problem
-------

You want created and updated fields on your records and would like them automatically updated when a row is added or updated.

Solution
--------

Define the following traits:

```scala
trait Created[T <: Created[T]] extends Record[T] {
  self: T =>
  val created: DateTimeField[T] = new DateTimeField(this) { 
    override def defaultValue = Calendar.getInstance
  }
}

trait Updated[T <: Updated[T]] extends Record[T] {
  self: T =>

  val updated = new DateTimeField(this) { 
    override def defaultValue = Calendar.getInstance
  }

  def onUpdate = this.updated(Calendar.getInstance)

}

trait CreatedUpdated[T <: Updated[T] with Created[T]] extends 
  Updated[T] with Created[T] { 
    self: T => 
}
```

Add to your model, for example: 

```scala
class YourRecord private () extends Record[YourRecord] 
  with KeyedRecord[Long] with CreatedUpdated[YourRecord] {
    override def meta = YourRecord
    //field entries ...
}
```

Finally, arrange for the `updated` field to be updated:

```scala
class YourSchema extends Schema {
  ...
  override def callbacks = Seq(       
    beforeUpdate[YourRecord] call {_.onUpdate} 
  ) 
  ... 
```

Discussion
----------

_This recipe requires Lift 2.5 or later._

Although there is a built in `net.lifetweb.record.LifecycleCallbacks` trait in which allows you trigger behaviour onUpdate, afterDelete and so on, it is only for use on individual Fields, rather than Records. As our goal is to update the `updated` field when any part of the Record changes, we can't use the `LiftcycleCallbacks` here.

Instead, the `CreatedUpdated` trait simplifies adding an `updated` and `created` fields to a Record, but we do need to remember to add a hook into the schema to ensure the `updated` value is changed when a record is modified.  This is why we set the `callbacks` on the Schema.

It should be noted that `onUpdate` is only called on full updates and not on partial updates with Squeryl. A full update is when the object is altered and then saved; a partial update is where you attempt to alter many objects via a query. 

If you're interested in other automations for Record, the Squery schema callbacks also support other triggered behaviours:

* `beforeInsert` and `afterInsert`
* `afterSelect`
* `beforeUpdate` and `afterUpdate`
* `beforeDelete` and `afterDelete`


See Also
--------

* [Explanation of full vs partial update in Squeryl](http://squeryl.org/inserts-updates-delete.html).
* Mailing list discussion [regarding LifecycleCallbacks](https://groups.google.com/d/msg/liftweb/G4U14pQbZZ4/V24YvhUPvEEJ).
Logging SQL
===========

Problem
-------

You want to see the SQL being executed by Record with Squeryl.


Solution
--------

Add the following anytime you have a Squeryl season, such as just before your query:

```scala
org.squeryl.Session.currentSession.setLogger( s => println(s) )
```

By providing a `String => Unit` function to `setLogger`, Squeryl will execute that function with the SQL it runs. In this example, we are simply printing the SQL to the console.


Discussion
----------

This recipe is not specific to Lift, and will work wherever you use Squeryl.

See Also
--------
* Squeryl [getting started](http://squeryl.org/getting-started.html) page.
* Squeryl page on [logging the generated SQL](http://squeryl.org/miscellaneous.html)


Model a column with MySQL MEDIUMTEXT
======================================

Problem
-------

You want to use MySQL's `MEDIUMTEXT` for a column, but `StringField` doesn't have this option.

Solution
--------

Use Squeryl's `dbType`:

```scala
on(mytable)(t => declare(
  t.mycolumn defineAs dbType("MEDIUMTEXT")
))
```

Discussion
----------

You can continue to use `StringField`, but regardless of the size you pass, the schema will be:

```sql
create table mytable (
    mycolumn MEDIUMTEXT not null
);
```

This recipe is not specific to Lift, and will work wherever you use Squeryl.

See Also
--------

* Squeryl [schema defintion](http://squeryl.org/schema-definition.html) page.
* [MySQL, Squeryl and MEDIUMTEXT with Record](https://groups.google.com/forum/?fromgroups#!topic/liftweb/TXbDGdX54LQ) mailing list discussion.