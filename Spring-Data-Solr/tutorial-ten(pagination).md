
In the earlier parts of my Spring Data Solr tutorial, we have implemented a simple search function which is used to search the information of todo entries. The current implementation of our search function shows all search results in a single page. This is not a viable solution for most real life applications because the number of search results can be so large that search function is no longer usable.

This blog post provides us the solution to that problem by describing how we can paginate the query results or our search function with Spring Data Solr.

This blog post is divided into five sections:

The first section describes how we can request the correct page manually and talks about the different return types of query methods.
The second section describes how we can obtain the search result count by adding a custom method to our repository.
The third section describes how we can paginate the search results of query methods.
The fourth section teaches us to paginate the search results of dynamic queries.
The fifth and last section describes how we can configure and use a technique called web pagination.
These blog posts provides additional information which helps us to understand the concepts of described in this blog post:
Running Solr with Maven
Spring Data Solr Tutorial: Introduction to Solr
Spring Data Solr Tutorial: Configuration
Spring Data Solr Tutorial: Query Methods
Spring Data Solr Tutorial: Adding Custom Methods to a Single Repository
Spring Data Solr Tutorial: Dynamic Queries
Spring Data Solr Tutorial: Sorting
Let’s get started.

Few Minutes of Theory

Before we will start making modifications to our example application, we will take a short look at the theory behind pagination. This section is divided in two subsections which are described in the following:

The first section describes how we can specify the pagination options of our query.
The second section describes the different return types of a query method.
Let’s move on.

Specifying the Wanted Page
The used pagination options are specified by using the PageRequest class which implements the Pageable interface.

The typical pagination requirements are given in the following:

Get the query results belonging to a single page.
Get the query results belonging to a single page when the query results are sorted by using the value of a single field.
Get the query results belonging to a single page when the query results are sorted by using the values of multiple fields and the sort order of different fields is the same.
Get the query results belonging to a single page when the query results are sorted by using the values of multiple fields and the sort order of different fields is not the same.
Let’s find out how we can create the PageRequest objects which fulfill the given requirements.

First, we must create a PageRequest object which specifies that we want to get the query results belonging to a single page. We can create the PageRequest object by using the following code:

1
2
//Get the query results belonging to the first page when page size is 10.
new PageRequest(0, 10)
Second, we must create a PageRequest object which specifies that we want to get the results belonging to a single page when query results are sorted by using the value of a single field. We can create the PageRequest object by using the following code:

1
2
3
//Gets the query results belonging to the first page when page size is 10.
//Query results are sorted in descending order by using id field.
new PageRequest(0, 10 Sort.Direction.DESC, "id")
Third, we must create a PageRequest object which specifies that we want to get the results belonging to a single page when the query results are sorted by using multiple fields and the sort order of different fields is same. We can create the PageRequest object by using the following code:

1
2
3
//Gets the query results belonging to the first page when page size is 10.
//Query results are sorted in descending order by using id and description fields.
new PageRequest(0, 10 Sort.Direction.DESC, "id", "description")
Fourth, we must create a PageRequest object which specifies that want to get the query results belonging to a single page when the query results are sorted by using multiple fields and the sort order of different fields is not same. We can create this object by using the following code:

1
2
3
4
5
//Gets the query results belonging to the first page when page size is 10.
//Query results are sorted in descending order order by using the description field
//and in ascending order by using the id field.
Sort sort = new Sort(Sort.Direction.DESC, "description").and(new Sort(Sort.Direction.ASC, "id"))
new PageRequest(0, 10, sort);
We now know how we can create new PageRequest objects. Let’s move on and talk about the different return types of query methods.

Deciding the Return Type of a Query Method
When a query method uses pagination, it can have two return types. These return types are described in the following (We will assume that the name of our model class is TodoDocument):

When we are interested about the pagination metadata, the return type of our query method must be Page<TodoDocument> (Get more information about the Page interface which declares the methods used to obtain the pagination metadata).
When we are not interested about the pagination metadata, the return type of our query method should be List<TodoDocument>.
Getting the Search Result Count

Before we can start paginating the search results of our queries, we have to implement a function which is used to get the number of todo entries which matches with given search criteria. This number is required so that we can implement the pagination logic to the frontend.

We can implement this function by following these steps:

Add a custom method to our repository. This method is used to return the search result count.
Create a new service method which uses our custom repository method.
These steps are described with more details in the following subsections.

Adding a Custom Method to Our Repository
At the moment it is not possible to create a count query without adding a custom method to a repository. We can do this by following these steps:

Create a custom repository interface.
Implement the custom repository interface.
Modify the actual repository interface.
Let’s move on and find out how this is done.

Creating a Custom Repository Interface
We can create a custom repository interface by following these steps:

Create an interface called CustomTodoDocumentRepository.
Add a count() method to the created interface. This method takes the used search term as a method parameter.
The source code of the CustomTodoDocumentRepository interface looks as follows:

1
2
3
4
5
6
public interface CustomTodoDocumentRepository {
 
    public long count(String searchTerm);
 
    //Other methods are omitted
}
Implementing the Custom Repository Interface
We can implement the custom repository interface by following these steps:

Create a class called TodoDocumentRepositoryImpl and implement the CustomTodoDocumentRepository interface.
Annotate the class with the @Repository annotation.
Add SolrTemplate field to the class and annotate the field with the @Resource annotation.
Implement the count() method.
Let’s take a closer look at the implementation of the count() method. We can implement this method by following these steps:

Get words of the given search term.
Construct the used search criteria by calling the private constructSearchConditions() method and pass the words of the search term as a method parameter.
Create the executed query by creating a new SimpleQuery object and pass the created Criteria object as a constructor parameter.
Get the search result count by calling the count() method of the SolrTemplate class and pass the created SimpleQuery object as a method parameter.
Return the search result count.
The source code of the TodoDocumentRepositoryImpl class looks as follows:

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
import org.springframework.data.solr.core.SolrTemplate;
import org.springframework.data.solr.core.query.Criteria;
import org.springframework.data.solr.core.query.SimpleQuery;
import org.springframework.stereotype.Repository;
 
import javax.annotation.Resource;
 
@Repository
public class TodoDocumentRepositoryImpl implements CustomTodoDocumentRepository {
 
    @Resource
    private SolrTemplate solrTemplate;
 
    @Override
    public long count(String searchTerm) {
        String[] words = searchTerm.split(" ");
        Criteria conditions = createSearchConditions(words);
        SimpleQuery countQuery = new SimpleQuery(conditions);
 
        return solrTemplate.count(countQuery);
    }
 
    private Criteria createSearchConditions(String[] words) {
        Criteria conditions = null;
 
        for (String word: words) {
            if (conditions == null) {
                conditions = new Criteria("title").contains(word)
                        .or(new Criteria("description").contains(word));
            }
            else {
                conditions = conditions.or(new Criteria("title").contains(word))
                        .or(new Criteria("description").contains(word));
            }
        }
 
        return conditions;
    }
 
    //Other methods are omitted.
}
Modifying the Actual Repository Interface
We can make the custom count() method visible to the users of our repository by extending the CustomTodoRepositoryInterface. The source code of the TodoDocumentRepository looks as follows:

1
2
3
public interface TodoDocumentRepository extends CustomTodoRepository, SolrCrudRepository<TodoDocument, String> {
    //Repository methods are omitted.
}
Using the Custom Repository Method
We can use the created repository method by following these steps:

Modify the TodoIndexService interface.
Implement the modified interface.
These steps are described with more details in the following.

Note: We have to make other changes as well but I will not describe these changes here since they are not related to Spring Data Solr.

Modifying the Service Interface
We have to modify the TodoIndexService interface by adding a new countSearchResults() method to it. This method takes the used search term as a method parameter and returns the search result count. The source code of the TodoIndexService interface looks as follows:

1
2
3
4
5
6
public interface TodoIndexService {
 
    public long countSearchResults(String searchTerm);
 
    //Other methods are omitted.
}
Implementing the Modified Interface
We can implement the countSearchResults() method by following these steps:

Add the countSearchResults() method to the RepositoryTodoIndexService class.
Obtain the search result count by calling the custom repository method and return the search result count.
The relevant part of the RepositoryTodoIndexService class looks as follows:

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
 
import javax.annotation.Resource;
import java.util.ArrayList;
import java.util.List;
 
@Service
public class RepositoryTodoIndexService implements TodoIndexService {
 
    @Resource
    private TodoDocumentRepository repository;
 
    @Override
    public long countSearchResults(String searchTerm) {
        return repository.count(searchTerm);
    }
 
    //Other methods are omitted.
}
Paginate the Query Results of Query Methods

When we are creating our queries by using query methods, we can paginate the query results by following these steps:

Add a new Pageable parameter to the query method. This parameter specifies the details of the fetched page.
Modify the service layer by adding a new Pageable parameter to the search() method of the TodoIndexService interface.
Let’s get started.

Modifying the Repository Interface
We can add pagination support to our repository by adding a Pageable parameter to the query method which is used to build the executed query. Let’s take a look at the declarations of our query methods.

Query Generation from Method Name
When the executed query is created by using the query generation from method name strategy, we have to add a Pageable parameter to the findByTitleContainsOrDescriptionContains() method of the TodoDocumentRepository interface. These source code of our repository interface looks as follows:

1
2
3
4
5
6
7
8
9
import org.springframework.data.domain.Pageable;
import org.springframework.data.solr.repository.SolrCrudRepository;
 
import java.util.List;
 
public interface TodoDocumentRepository extends CustomTodoDocumentRepository, SolrCrudRepository<TodoDocument, String> {
 
    public List<TodoDocument> findByTitleContainsOrDescriptionContains(String title, String description, Pageable page);
}
Named Queries
When are using named queries, we have to add a Pageable parameter to the findByNamedQuery() method of the TodoDocumentRepository interface. The source code of the TodoDocumentRepository interface looks as follows:

1
2
3
4
5
6
7
8
9
10
11
import org.springframework.data.domain.Pageable;
import org.springframework.data.solr.repository.Query;
import org.springframework.data.solr.repository.SolrCrudRepository;
 
import java.util.List;
 
public interface TodoDocumentRepository extends CustomTodoDocumentRepository, SolrCrudRepository<TodoDocument, String> {
 
    @Query(name = "TodoDocument.findByNamedQuery")
    public List<TodoDocument> findByNamedQuery(String searchTerm, Pageable page);
}
@Query Annotation
When the executed query is created by using the @Query annotation, we have to add a Pageable parameter to the findByQueryAnnotation() method of the TodoDocumentRepository interface. The source code of our repository interface looks as follows:

1
2
3
4
5
6
7
8
9
10
11
import org.springframework.data.domain.Pageable;
import org.springframework.data.solr.repository.Query;
import org.springframework.data.solr.repository.SolrCrudRepository;
 
import java.util.List;
 
public interface TodoDocumentRepository extends CustomTodoDocumentRepository, SolrCrudRepository<TodoDocument, String> {
 
    @Query("title:*?0* OR description:*?0*")
    public List<TodoDocument> findByQueryAnnotation(String searchTerm, Pageable page);
}
Modifying the Service Layer
We have to the following modifications to the service layer of our example application:

Add a Pageable parameter to the search() method of the TodoIndexService interface.
Implement the new search() method.
Note: We have to make other changes as well but I will not describe these changes here since they are not related to Spring Data Solr.

The source code of the TodoIndexService interface looks as follows:

1
2
3
4
5
6
7
8
9
import org.springframework.data.domain.Pageable;
import java.util.List;
 
public interface TodoIndexService {
 
    public List<TodoDocument> search(String searchTerm, Pageable page);
 
    //Other methods are omitted.
}
We can use the modified query methods by making the following changes to the search() method of the RepositoryIndexService class:

Get the paginated query results by calling the query method of our repository and pass the used search term and the Pageable object as method parameters.
Return the query results.
Let’s move and take a look at different implementations of the search() method.

Query Generation from Method Name
When we are building our queries by using the query generation from method name strategy, we can get query results belonging to a specific page by using the findByTitleContainsOrDescriptionContains() method of the TodoDocumentRepository interface.

The relevant part of the RepositoryTodoIndexService class looks as follows:

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
import org.springframework.beans.factory.annotation.Value;
import org.springframework.data.domain.Pageable;
import org.springframework.data.domain.Sort;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
 
import javax.annotation.Resource;
import java.util.ArrayList;
import java.util.List;
 
@Service
public class RepositoryTodoIndexService implements TodoIndexService {
 
    @Resource
    private TodoDocumentRepository repository;
 
    @Override
    public List<TodoDocument> search(String searchTerm, Pageable page) {
        return repository.findByTitleContainsOrDescriptionContains(searchTerm, searchTerm, page);
    }
     
    //Other methods are omitted
}
Named Queries
When we are using named query for building the executed query, we can get the search results belonging to a specific page by using the findByNamedQuery() method of the TodoDocumentRepository interface.

The relevant part of the RepositoryTodoIndexService class looks as follows:

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
import org.springframework.beans.factory.annotation.Value;
import org.springframework.data.domain.Pageable;
import org.springframework.data.domain.Sort;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
 
import javax.annotation.Resource;
import java.util.ArrayList;
import java.util.List;
 
@Service
public class RepositoryTodoIndexService implements TodoIndexService {
 
    @Resource
    private TodoDocumentRepository repository;
 
    @Override
    public List<TodoDocument> search(String searchTerm, Pageable page) {
        return repository.findByNamedQuery(searchTerm, page);
    }
     
    //Other methods are omitted
}
@Query Annotation
When we are building our query by using the @Query annotation, we can get the search results which belongs to a specific page by calling the findByQueryAnnotation() method of the TodoDocumentRepository interface.

The source code of the RepositoryTodoIndexService class looks as follows:

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
import org.springframework.beans.factory.annotation.Value;
import org.springframework.data.domain.Pageable;
import org.springframework.data.domain.Sort;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
 
import javax.annotation.Resource;
import java.util.ArrayList;
import java.util.List;
 
@Service
public class RepositoryTodoIndexService implements TodoIndexService {
 
    @Resource
    private TodoDocumentRepository repository;
 
    @Override
    public List<TodoDocument> search(String searchTerm, Pageable page) {
        return repository.findByQueryAnnotation(searchTerm, page);
    }
     
    //Other methods are omitted
}
Paginating the Query Results of Dynamic Queries

We can paginate the query results of dynamic queries by following these steps:

Add a Pageable parameter to the search() method of our custom repository.
Modify the service layer by adding a Pageable parameter to the search() method of the TodoIndexService interface.
These steps are described with more details in the following subsections.

Changing the Custom Repository
We have to add pagination support to our custom repository. We can do this by following these steps:

Modify the custom repository interface by adding a Pageable parameter to its search() method.
Change the implementation of the search() method by adding pagination support to it.
Let’s move on and find out how this is done.

Changing the Custom Repository Interface
We have to add a Pageable parameter to the search() method declared in the CustomTodoDocumentRepository interface. The source code of our custom repository interface looks as follows:

1
2
3
4
5
6
7
8
9
10
import org.springframework.data.domain.Pageable;
 
import java.util.List;
 
public interface CustomTodoDocumentRepository {
 
    public List<TodoDocument> search(String searchTerm, Pageable page);
 
    //Other methods are omitted.
}
Implementing the Custom Repository Method
Our next step is to add pagination support to the implementation of the search() method. We can implement the search() method of the TodoDocumentRepositoryImpl class by following these steps:

Get the words of the search term.
Construct the used search criteria by calling the private createSearchConditions() method and passing the words of the search term as a method parameter.
Create the executed query by creating a new SimpleQuery object and pass the created Criteria object as a constructor parameter.
Set the pagination options of the query by calling the setPageRequest() method of the SimpleQuery class. Pass the Pageable object as a method parameter.
Get the search results by calling the queryForPage() method of the SolrTemplate class. Pass the created query and the type of the expected return objects as method parameters.
Return the search results by calling the getContent() method of the Page interface.
The source code of the TodoDocumentRepositoryImpl class looks as follows:

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.solr.core.SolrTemplate;
import org.springframework.data.solr.core.query.Criteria;
import org.springframework.data.solr.core.query.SimpleQuery;
import org.springframework.stereotype.Repository;
 
import javax.annotation.Resource;
import java.util.ArrayList;
import java.util.List;
 
@Repository
public class TodoDocumentRepositoryImpl implements CustomTodoDocumentRepository {
 
    @Resource
    private SolrTemplate solrTemplate;
 
    @Override
    public List<TodoDocument> search(String searchTerm, Pageable page) {
        String[] words = searchTerm.split(" ");
 
        Criteria conditions = createSearchConditions(words);
        SimpleQuery search = new SimpleQuery(conditions);
        search.setPageRequest(page);
 
        Page results = solrTemplate.queryForPage(search, TodoDocument.class);
        return results.getContent();
    }
 
    private Criteria createSearchConditions(String[] words) {
        Criteria conditions = null;
 
        for (String word: words) {
            if (conditions == null) {
                conditions = new Criteria("title").contains(word)
                        .or(new Criteria("description").contains(word));
            }
            else {
                conditions = conditions.or(new Criteria("title").contains(word))
                        .or(new Criteria("description").contains(word));
            }
        }
 
        return conditions;
    }
 
    //Other methods are omitted.
}
Using the Custom Repository
Before we can use the modified repository method, we have to make the following changes to the service layer of our example application:

Add a Pageable parameter to the search() method of the TodoIndexService interface.
Implement the search() method.
These steps are described with more details in following.

Note: We have to make other changes as well but I will not describe these changes here since they are not related to Spring Data Solr.

Modifying the Service Interface
We have to add a Pageable parameter to the search() method of the TodoIndexService interface. The source code of the TodoIndexService looks as follows:

1
2
3
4
5
6
7
8
9
import org.springframework.data.domain.Pageable;
import java.util.List;
 
public interface TodoIndexService {
 
    public List<TodoDocument> search(String searchTerm, Pageable page);
 
    //Other methods are omitted.
}
Implementing the Service Interface
When we are building our by using the criteria API of Spring Data Solr, we can get the query results by calling the search() method of our custom repository and passing the user search term and the Pageable object as method parameters.

The source code of the RepositoryTodoIndexService class looks as follows:

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
import org.springframework.data.domain.Pageable;
import org.springframework.stereotype.Service;
import javax.annotation.Resource;
import java.util.List;
 
@Service
public class RepositoryTodoIndexService implements TodoIndexService {
 
    @Resource
    private TodoDocumentRepository repository;
 
    @Override
    public List<TodoDocument> search(String searchTerm, Pageable page) {
        return repository.search(searchTerm, page);
    }
 
    //Other methods are omitted.
}
Using Web Pagination

One question is still left unanswered. That is question is:

Where the pagination options used to paginate the query results of our queries are specified?
We will create the pagination options of our queries by using a technique called web pagination. This technique is based on a custom argument resolver class called PageableArgumentResolver. This class parses pagination information from HTTP request and makes it possible to add a Pageable method parameter to controller methods.

This section describes how we can configure and use this technique in our example application. It is divided into three subsections:

The first subsection describes how we can configure the PageableArgumentResolver class.
The second subsection describes how we can use it.
The last subsection talks about the pros and cons of web pagination.
Let’s find out how we can use this technique in our example application.

Configuration
This subsection describes how we can configure the PageableArgumentResolver class which is to used extract pagination options from HTTP requests. Let’s find out how we do this by using Java configuration and XML configuration.

Java Configuration
We can add a custom argument argument resolver by making the following changes to the ExampleApplicationContext class:

Override the addArgumentResolvers() method of the WebMvcConfigurerAdapter class.
Implement the addArgumentResolvers() method by creating new PageableArgumentResolver object and adding the created object to the list of argument resolvers which is given as a method parameter.
The relevant part of the ExampleApplicationContext class looks as follows:

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
import org.springframework.data.web.PageableArgumentResolver;
import org.springframework.web.method.support.HandlerMethodArgumentResolver;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;
import org.springframework.web.servlet.mvc.method.annotation.ServletWebArgumentResolverAdapter;
 
import java.util.List;
 
//Annotations are omitted.
public class ExampleApplicationContext extends WebMvcConfigurerAdapter {
 
    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> argumentResolvers) {
        PageableArgumentResolver pageableArgumentResolver = new PageableArgumentResolver();
        argumentResolvers.add(new ServletWebArgumentResolverAdapter(pageableArgumentResolver));
    }
 
    //Other methods are omitted.
}
XML Configuration
We can configure a custom argument resolver by making the following changes to the exampleApplicationContext.xml file:

Use the argument-resolvers element of the mvc namespace for configuring the custom argument resolvers.
Configure the PageableArgumentResolver bean inside the argument-resolvers element.
The relevant part of the exampleApplicationContext.xml file looks as follows:

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-3.2.xsd">
    <mvc:annotation-driven>
        <mvc:argument-resolvers>
            <bean id="pageagleArgumentResolver" class="org.springframework.data.web.PageableArgumentResolver"/>
        </mvc:argument-resolvers>
    </mvc:annotation-driven>
 
    <!-- Configuration is omitted. -->
</beans>
Usage
After we have configured the PageableArgumentResolver class by using one of methods which were described earlier, we can add Pageable method parameters to our controller methods. The search() method the TodoController class is a good example of this. The relevant part of its source code looks as follows:

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
import org.springframework.data.domain.Pageable;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;
 
import javax.annotation.Resource;
import java.util.List;
 
@Controller
public class TodoController {
 
    //Fields are omitted.
 
    @RequestMapping(value = "/api/todo/search/{searchTerm}", method = RequestMethod.GET)
    @ResponseBody
    public List<TodoDTO> search(@PathVariable("searchTerm") String searchTerm, Pageable page) {
        //Implementation is omitted.
    }
 
    //Other methods are omitted.
}
However, adding the Pageable argument to the controller method is not enough. We still have to add the pagination options to the HTTP request. This is done by adding special request parameters to the request. These request parameters are described in the following:

The page.page request parameter specifies the requested page.
The page.size request parameter specifies the page size.
The page.sort request parameter specifies the property which is used to sort the query results.
The page.sort.dir request parameter specifies the sort order.
Let’s spend a moment and think about the pros and cons of web pagination.

Pros and Cons
The web pagination has both pros and cons which we should be aware of before making the decision to use it in our applications. Let’s find out what these are.

Pros
Using web pagination has one major benefit:

It is an easy and simple to transfer pagination options from the web layer to the repository layer. All we have to do is to configure a custom argument resolver, add a Pageable parameter to a controller method and send the pagination options by using specific request parameters. This is a lot simpler than processing pagination options in our code and manually creating a PageRequest object.

Cons
The cons of using web pagination are described in the following:

Web pagination creates a dependency between the web layer and Spring Data. This means that the implementation details of the repository layer leaks into to the upper layers of our application. Although purists will probably claim that this is huge mistake, I don’t share their opinion. I think that abstractions should make our life easier, not harder. We must also remember that the law of leaky abstractions states that all-non trivial abstractions, to some degree, are leaky.
One real disadvantage of web pagination is that we can use it only if our search results are sorted by using a single field. Although this is perfectly fine for most use cases, there are situations when this becomes a problem. If this happens, we have to process pagination options manually.
Summary

We have now added the pagination of search results to our example application. This tutorial has taught us the following things:

We learned to create new PageRequest objects.
We learned that we can select the return type of our query method from two different options.
We learned to paginate the query results of query methods and dynamic queries.
We know how we can use web pagination, and we are aware of its pros and cons.
The next part of my Spring Data Solr Tutorial describes how we can add custom methods to all Spring Data Solr repositories.

P.S. The example applications of this blog post are available at Github (query methods and dynamic queries).

If you want to learn how to use Spring Data Solr, you should read my Spring Data Solr tutorial.
