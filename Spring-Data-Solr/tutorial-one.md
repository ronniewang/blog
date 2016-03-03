
本文翻译自<http://www.petrikainulainen.net/spring-data-solr-tutorial/>

Most of the applications must have a some kind of a search function. The problem is that search functions are often huge resource hogs and they can kill the performance of your application by causing heavy load to the database.

That is why transferring that load to an external search server is a great idea. Apache Solr is a popular open source search server which is used via REST-like HTTP API. This ensures that you can use Solr from virtually any programming language.

Although the ability to support any programming language is a great marketing benefit, the question that probably interests you is: How I can use Solr in my Spring powered applications?

Introducing Spring Data Solr Tutorial

This ten-part tutorial will help you get started with Spring Data Solr. This tutorial is based on Spring Data Solr 1.0.0.RC1 but I plan to update it when newer versions are released.

Let’s move on and find out what you can learn from this tutorial. This tutorial contains the following blog posts:

* [Running Solr with Maven](http://www.petrikainulainen.net/programming/maven/running-solr-with-maven/) describes how we can run Solr by using Maven and ensure that each developer uses the same configuration, schema and Solr version.

* [Spring Data Solr Tutorial: Introduction to Solr](http://www.petrikainulainen.net/programming/solr/spring-data-solr-tutorial-introduction-to-solr/) gives a brief introduction to Solr data model, describes how you can create a schema to your Solr instance and describes the usage of Solr’s HTTP API.

* [Spring Data Solr Tutorial: Configuration](http://www.petrikainulainen.net/programming/solr/spring-data-solr-tutorial-configuration/) describes how you can get the required dependencies by using Maven and configure Spring Data Solr.

* [Spring Data Solr Tutorial: CRUD (Almost)](http://www.petrikainulainen.net/programming/solr/spring-data-solr-tutorial-crud-almost/) describes how you can add new to documents to the Solr index, update the information of existing documents and delete documents from the Solr index.

* [Spring Data Solr Tutorial: Query Methods](http://www.petrikainulainen.net/programming/solr/spring-data-solr-tutorial-query-methods/) describes how you search documents from the Solr index by using query methods.

* [Spring Data Solr Tutorial: Adding Custom Methods to a Single Repository](http://www.petrikainulainen.net/programming/solr/spring-data-solr-tutorial-adding-custom-methods-to-a-single-repository/) describes how you can add custom methods to a single repository.

* [Spring Data Solr Tutorial: Dynamic Queries](http://www.petrikainulainen.net/programming/solr/spring-data-solr-tutorial-dynamic-queries/) describes how you can create dynamic queries by using the criteria implementation of Spring Data Solr.

* [Spring Data Solr Tutorial: Sorting](http://www.petrikainulainen.net/programming/solr/spring-data-solr-tutorial-sorting/) describes how you can sort your query results.

* [Spring Data Solr Tutorial: Pagination](http://www.petrikainulainen.net/programming/solr/spring-data-solr-tutorial-pagination/) describes how you can paginate the query results of query methods and dynamic queries with Spring Data Solr.

* [Spring Data Solr Tutorial: Adding Custom Methods to All Repositories](http://www.petrikainulainen.net/programming/solr/spring-data-solr-tutorial-adding-custom-methods-to-all-repositories/) describes how you can add custom methods to all repositories.

Congratulations. You are now ready to start using Spring Data Solr in your own applications. I hope that I was able to convince you that implementing search functions with Spring Data Solr is easy and fun.
