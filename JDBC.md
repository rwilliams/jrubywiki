It is possible to use JDBC directly from within JRuby. A simple function using the MySQL JDBC driver to perform a SELECT is shown below:

```ruby
  def vegetableFinder(vegetable)
    # This function takes a hashmap of vegetables and attempts to
    # find them from our grocery database. For each item found, we
    # call our 'makevegetablesoup' function.

    # Load all required gems
    require "rubygems"
    require "jdbc/mysql"
    require "java"

    begin
      # Prep the connection
      Java::com.mysql.jdbc.Driver
      userurl = "jdbc:mysql://HOST/DATABASE"
      connSelect = java.sql.DriverManager.get_connection(userurl, "USERNAME", "PASSWORD")
      stmtSelect = connSelect.create_statement

      # Define the query
      selectquery = %q{SELECT name, type, size, price
            FROM vegetables
            WHERE type = "#{vegetable["type"]}"
            AND size = "#{vegetable["size"]}}

      # Execute the query
      rsS = stmtSelect.execute_query(selectquery)

      # For each row returned do some stuff
      while (rsS.next) do
        veg = Hash.new
        veg["vegname"] = rsS.getObject("name")
        veg["vegtype"] = rsS.getObject("type")
        veg["vegprice"] = rsS.getObject("size")
        veg["vegsize"] = rsS.getObject("price")
        makevegetablesoup(veg)
      end
    end
    # Close off the connection
    stmtSelect.close
    connSelect.close
    return truth
  end
```

INSERTs and UPDATEs are much the same. Again, using the MySQL JDBC driver a simple insert is shown below:

```ruby
  def groceryInsert(vegetable)
    # Inserts a new vegetable type into the grocery database

    # Load all required gems
    require "rubygems"
    require "jdbc/mysql"
    require "java"

    begin
      # Prep the connection
      Java::com.mysql.jdbc.Driver
      userurl = "jdbc:mysql://HOST/DATABASE"
      connInsert = java.sql.DriverManager.get_connection(userurl, "USERNAME", "PASSWORD")
      stmtInsert = connInsert.create_statement

      # Do the insert
      rsI = stmtInsert.execute_update("INSERT INTO vegetables (name,type,size,price)
          VALUES
               (
               '#{vegetable["name"]}',
               '#{vegetable["type"]}',
               '#{vegetable["size"]}',
               '#{vegetable["price"]}'
               );")
      end
    end
    # Close off the connection
    stmtInsert.close
    connInsert.close

    # Return the number of records inserted
    return rsI
  end
```

Remember to have the JDBC driver for your database in your CLASSPATH!
___
Here's an example of connecting to SQL Server using Microsoft's [JDBC driver](http://msdn.microsoft.com/en-us/sqlserver/aa937724.aspx). The open source [jTDS driver](http://jtds.sourceforge.net/) is another good option.

```ruby
  require 'rubygems'
  require 'java'
  require 'sqljdbc4.jar'

  Java::com.microsoft.sqlserver.jdbc.SQLServerDriver
  url = 'jdbc:sqlserver://host:1433;databaseName=database'
  conn = java.sql.DriverManager.get_connection(url, "user", "password")
  statement = conn.create_statement

  q = "SELECT * FROM vegtables"
  rs = statement.execute_query(q)

  while (rs.next) do
    puts rs.getObject('name')
  end

  statement.close
```

Note: sqljdbc4.jar does NOT work on Java 5.  Use Microsoft's sqljdbc.jar (available in the same download mentioned above) on Java 5, if you get this error:

```
The driver encountered an unknown error: java.lang.NoClassDefFoundError: java/sql/SQLClientInfoException
```

## Using non-activerecord JDBC from JRuby on TorqueBox

For whatever reason TorqueBox has some issues with dynamically loaded drivers and therefore won't function correctly without being explicit. ActiveRecord abstracts this away for you but if you are not using ActiveRecord and just using the plain JDBC driver you'll need to be explicit in using the driver:

```ruby
driver = Java::com.microsoft.sqlserver.jdbc.SQLServerDriver.new
props = java.util.Properties.new
props.setProperty("user", "username")
props.setProperty("password", "password")
url = 'jdbc:sqlserver://servername;instanceName=instance;databaseName=DbName;'
conn = driver.connect(url, props)
```
