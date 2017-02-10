---
layout: post
title: "Using In Memory SQLite Database for Testing with FluentNHibernate"
date: 2009-04-04 0400
comments: true
disqus_identifier: 20
categories: [ASP.NET,MVC]
redirect_from: "/archive/2009/04/04/using-in-memory-sqlite-database-for-testing-with-fluentnhibernate.aspx/"
---
I’ve been playing with a little bit of TDD with FluentNHibernate and the
MVC Framework lately and I had a few issues trying to get unit tests
running with an in-memory SQLite database.  There are quite a few blogs
describing how to do this but none of them use FluentNHibernate, so I
thought I’d document the way I achieved this.  I’m not sure that this is
the best way, so if anyone has a better idea please let me know.

I started off with this class to configure my mappings:

```csharp
public class NHibernateMapping
{
    public ISessionFactory BuildSessionFactory()
    {
        return Fluently.Configure()
            .Database(SQLiteConfiguration.Standard.InMemory())
            .Mappings(
                o => o.AutoMappings.Add(
                    AutoPersistenceModel.MapEntitiesFromAssemblyOf<MyDummyEntity>()
                        .WithSetup(a =>
                            {
                                a.IsBaseType = ty => ty.FullName == typeof(DomainEntity).FullName;
                                a.FindIdentity = prop => prop.Name.Equals("Id");
                            }
                        )
                )
            )
        .BuildSessionFactory();
    }
}
```

At first this worked absolutely fine for my tests.  However, no where in
here is the schema for the database actually defined.  My initial tests
passed only because they were creating, loading and saving objects in
the same NHibernate session so they weren’t actually hitting the
database!  NH could supply everything from it’s level 1 cache.  When I
wrote a test to check that an action worked as expected when an invalid
ID was specified it failed with an *ADOException* from NH – because it
now tried to read a row from the database but the table didn’t exist!

I then changed my *NHibernateMapping* class to call *SchemaExport*, but
the test still failed because *SchemaExport* creates the schema and then
closes the connection.  This destroys the in-memory database so when my
test read the table didn’t exist again!

From this
[post](http://www.houseofbilz.com/archive/2008/07/22/active-record-mock-framework.aspx)
I found a connection provider which ensured that the same connection
would always be used.  The code for this class is:

```csharp
public class SQLiteInMemoryTestConnectionProvider :
    NHibernate.Connection.DriverConnectionProvider
{
    private static IDbConnection _connection;

    public override IDbConnection GetConnection()
    {
        if (_connection == null)
            _connection = base.GetConnection();
        return _connection;
    }

    public override void CloseConnection(IDbConnection conn)
    {
    }

    /// <summary>
    /// Destroys the connection that is kept open in order to 
    /// keep the in-memory database alive.  Destroying
    /// the connection will destroy all of the data stored in 
    /// the mock database.  Call this method when the
    /// test is complete.
    /// </summary>
    public static void ExplicitlyDestroyConnection()
    {
        if (_connection != null)
        {
            _connection.Close();
            _connection = null;
        }
    }
}
```

I then modified the *NHibernateMapping* class to expose the NH
configuration and session factory separately, and also allow the
*IPersistenceConfigurer* to be passed it (so that I could use a
different database for testing and live).  The class now looks like
this:

```csharp
public class NHibernateMapping
{

    IPersistenceConfigurer _dbConfig;

    public NHibernateMapping(IPersistenceConfigurer dbConfig)
    {
        _dbConfig = dbConfig;
    }

    public Configuration BuildConfiguration()
    {
        return Fluently.Configure()
            .Database(_dbConfig)
            .Mappings(
                o => o.AutoMappings.Add(
                    AutoPersistenceModel.MapEntitiesFromAssemblyOf<MyDummyEntity>()
                        .WithSetup(a =>
                            {
                                a.IsBaseType = ty => ty.FullName == typeof(DomainEntity).FullName;
                                a.FindIdentity = prop => prop.Name.Equals("Id");
                            }
                        )
                )
            )
        .BuildConfiguration();
    }

    public ISessionFactory BuildSessionFactory()
    {
        return BuildConfiguration().BuildSessionFactory();
    }

}
```

Then, in the test setup, I just need to tell FluentNH to use my test
connection provider, call *SchemaExport*, and create my
*SessionFactory*:

```csharp
[TestInitialize]
public void Init()
{
    var mapping = new NHibernateMapping(
        SQLiteConfiguration.Standard.InMemory()
            .Provider<SQLiteInMemoryTestConnectionProvider>());
    new NHibernate.Tool.hbm2ddl.SchemaExport(m.BuildConfiguration())
        .Execute(true, true, false, true);
    _sessionFactory = m.BuildSessionFactory();
}
```

As I said, I’m not sure if this is the best way to achieve this, so if
someone has a more elegant solution please let me know.

