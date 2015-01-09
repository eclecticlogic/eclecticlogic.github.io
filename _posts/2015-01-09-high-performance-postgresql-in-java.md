---
layout: post
title: High Performance Postgresql in Java with Pedal
author: Karthik Abram
tags: java postgresql pedal 
---

### Overview

Postgresql is one of the most popular open-source databases in use. It has good standards support and an impressive feature set - rich data types and advanced runtime management. The Pedal framework for Java enables fast inserts (we are talking orders of magnitude faster) into Postgresql through the [Copy](http://www.postgresql.org/docs/9.4/static/sql-copy.html) command directly at the JPA entity level. In this article, we provide an overview of the Pedal framework and show how to use the Copy command.   
 
The Pedal framework consists of three libraries - dialect, tx and loader. [pedal-dialect](http://www.eclecticlogic.com/pedal-dialect) enables dialect (i.e., database) and provider (e.g., Hibernate) level features such as retrieval of schema name, mapped table name given a JPA entity, user-types (arrays, bit strings), etc. It also provides support for the Copy command in Posgresql. [pedal-tx](http://www.eclecticlogic.com/pedal-tx) is a Java-8 only framework that allows for transaction demarcation using Java 8 lambdas, transaction attached storage and transaction attached pre/post commit lambdas. In addition it also provides a DAO abstraction layer with support for fluent JQL/HQL and native queries. With [pedal-loader](http://www.eclecticlogic.com/pedal-loader) data population scripts for db-unit tests can be written in a Groovy DSL while working at the JPA entity level (with mapped column types, object-level foreign-keys, etc). 

### Using the Copy Command

To enable copy support, first create an instance of `com.eclecticlogic.pedal.dialect.postgresql.CopyCommand`, ideally setup as a Spring-bean. It requires access to `com.eclecticlogic.pedal.provider.ProviderAccessSpi` which can be configured by creating a `com.eclecticlogic.pedal.provider.hibernate.HibernateProviderAccessSpiImpl.HibernateProviderAccessSpiImpl` passing it a reference to the `EntityManagerFactory`. Using Spring with Java-based configuration, the code would look like this:

```
    @Bean
    HibernateProviderAccessSpiImpl hibernateProvider(EntityManagerFactory factory) {
        HibernateProviderAccessSpiImpl impl = new HibernateProviderAccessSpiImpl();
        impl.setEntityManagerFactory(factory);
        return impl;
    }

    @Bean
    public CopyCommand copyCommand(ProviderAccessSpi provider) {
        CopyCommand command = new CopyCommand();
        command.setProviderAccessSpi(provider);
        command.setConnectionAccessor(new TomcatJdbcConnectionAccessor());
        return command;
    }
``` 

The connection accessor is specific to the connection pool you are using. It is used to get a handle to the underlying JDBC-4 compliant Postgresql native connection. Pedal-dialect ships with support for the following connection pools:

1. [BoneCP](http://jolbox.com/)
2. [Commons DBCP 2](http://commons.apache.org/proper/commons-dbcp/)
3. [Hikari](https://github.com/brettwooldridge/HikariCP)
4. [Tomcat JDBC](http://tomcat.apache.org/tomcat-7.0-doc/jdbc-pool.html) 

To create one for an unsupported connection pool, implement the `com.eclecticlogic.pedal.connection.ConnectionAccessor` interface.

So what does the code to actual insert rows using the `CopyCommand` look like? Its simple. Create instance of your entity and copy them into a `CopyList<T>`. Then pass the `CopyList` instance to the copy command. Here is the test code in pedal-dialect:

```
        CopyList<ExoticTypes> list = new CopyList<>();

        // The copy-command can insert 100k of these per second.
        for (int i = 0; i < 10; i++) {
            ExoticTypes et = new ExoticTypes();
            et.setLogin("copyCommand" + i);
            BitSet bs = new BitSet(7);
            bs.set(1);
            bs.set(3);
            bs.set(4);
            et.setCountries(bs);
            et.setAuthorizations(Sets.newHashSet("a", "b", "b", "c"));
            if (i != 9) {
                et.setScores(Lists.newArrayList(1L, 2L, 3L));
            } else {
                et.setScores(Lists.<Long> newArrayList());
            }
            et.setStatus(Status.ACTIVE);
            et.setCustom("this will be made uppercase");
            list.add(et);
        }

        copyCommand.insert(entityManager, list);
```

The `CopyCommand` supports a subset of JPA features. Check [here](http://www.eclecticlogic.com/pedal-dialect#CopyCommandSupportedFeatures) for the current supported feature set. 

For a moderate row size (100 to 200 bytes), 100k inserts using `EntityManager.persist()` took 34 seconds while the `CopyCommand` took 1.4 seconds! That is a 24x speed-up. Make sure you read up on the limitations of the copy feature in Postgresql. In addition, the Copy command will bypass JPA/Hibernate interceptors. So don't expect Envers to work with it. 
