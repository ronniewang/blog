
本文翻译自：<http://www.petrikainulainen.net/spring-data-solr-tutorial/>

大多数应用都有搜索的需求，搜索的问题就是经常会耗费过多的系统资源。

把搜索的负载放到一个单独的服务器上是一个好想法，Apache Solr就是一个流行的开源的搜索引擎服务器，提供REST式的HTTP API，这就保证了你可以使用任何语言来使用solr的服务。

下面我们就来看看怎么在Spring项目里使用Solr。

### Introducing Spring Data Solr Tutorial

This ten-part tutorial will help you get started with Spring Data Solr. This tutorial is based on Spring Data Solr 1.0.0.RC1 but I plan to update it when newer versions are released.

Let’s move on and find out what you can learn from this tutorial. This tutorial contains the following blog posts:

* [Running Solr with Maven](https://github.com/ronniewang/blog/blob/master/Spring-Data-Solr/tutorial-two(running-solr-with-maven).md) describes how we can run Solr by using Maven and ensure that each developer uses the same configuration, schema and Solr version.

* [Spring Data Solr Tutorial: Introduction to Solr](https://github.com/ronniewang/blog/blob/master/Spring-Data-Solr/tutorial-three(introduction-to-solr).md) gives a brief introduction to Solr data model, describes how you can create a schema to your Solr instance and describes the usage of Solr’s HTTP API.

* [Spring Data Solr Tutorial: Configuration](https://github.com/ronniewang/blog/blob/master/Spring-Data-Solr/tutorial-four(configuration).md) describes how you can get the required dependencies by using Maven and configure Spring Data Solr.

* [Spring Data Solr Tutorial: CRUD (Almost)](https://github.com/ronniewang/blog/blob/master/Spring-Data-Solr/tutorial-five(crud-almost).md) describes how you can add new to documents to the Solr index, update the information of existing documents and delete documents from the Solr index.

* [Spring Data Solr Tutorial: Query Methods](https://github.com/ronniewang/blog/blob/master/Spring-Data-Solr/tutorial-six(query-methods).md) describes how you search documents from the Solr index by using query methods.

* [Spring Data Solr Tutorial: Adding Custom Methods to a Single Repository](https://github.com/ronniewang/blog/blob/master/Spring-Data-Solr/tutorial-seven(adding-custom-methods-to-a-single-repository).md) describes how you can add custom methods to a single repository.

* [Spring Data Solr Tutorial: Dynamic Queries](https://github.com/ronniewang/blog/blob/master/Spring-Data-Solr/tutorial-eight(dynamic).md) describes how you can create dynamic queries by using the criteria implementation of Spring Data Solr.

* [Spring Data Solr Tutorial: Sorting](https://github.com/ronniewang/blog/blob/master/Spring-Data-Solr/tutorial-nine(sorting).md) describes how you can sort your query results.

* [Spring Data Solr Tutorial: Pagination](https://github.com/ronniewang/blog/blob/master/Spring-Data-Solr/tutorial-ten(pagination).md) describes how you can paginate the query results of query methods and dynamic queries with Spring Data Solr.

* [Spring Data Solr Tutorial: Adding Custom Methods to All Repositories](https://github.com/ronniewang/blog/blob/master/Spring-Data-Solr/tutorial-eleven(adding-custom-methods-to-all-repositories).md) describes how you can add custom methods to all repositories.

Congratulations. You are now ready to start using Spring Data Solr in your own applications. I hope that I was able to convince you that implementing search functions with Spring Data Solr is easy and fun.
