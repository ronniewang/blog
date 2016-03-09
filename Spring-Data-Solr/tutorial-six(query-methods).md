We have learned how we can configure Spring Data Solr. We have also learned how we can add new documents to the Solr index, update the information of existing documents and delete documents from the Solr index. Now it is time to move forward and learn how we can search information from the Solr index by using Spring Data Solr.

The requirements of our search function are given in the following:

The search function must return all todo entries which title or description contains the given search term.
The search must be case insensitive.
We can implement the search function by following these steps:

Create a query method.
Use the created query method.
Let’s move on and find out how we can implement the search function by using query methods.

These blog entries provide additional information which helps us to understand the concepts described in this blog entry:
Running Solr with Maven
Spring Data Solr Tutorial: Introduction to Solr
Spring Data Solr Tutorial: Configuration
Spring Data Solr Tutorial CRUD (Almost)
Creating the Query Method

Query methods are methods which are

added to the repository interface.
used to specify the search query which is executed when the query method is called.
used to build static queries (queries which have always the same amount of query parameters).
We can create query methods with Spring Data Solr by using the following techniques:

Query generation from the method name
Named queries
@Query annotation
These techniques are described with more details in the following subsections.

Query Generation from the Method Name
The query generation from the method name is a query generation strategy where the executed query is parsed from the name of the query method.

The name of the query method must start with a special prefix which identifies the query method. These prefixes are: find, findBy, get, getBy, read and readBy. This prefix is stripped from the method name when the executed query is parsed.
Property expressions are used to refer to the properties of our document class.
Special keywords are used together with property expressions to specify the operators used in the created query. These keywords are added to the name of the query method after a property expression.
We can combine property expressions by adding either And or Or keyword between them.
The parameter count of the query method must be equal with the number of property expressions used in its name.
We can get more information about the property expressions and the repository keywords by reading the following resources:

Spring Data Solr Reference Manual: Query Creation
Spring Data Solr Reference Manual: Repository Query Keywords
As we remember, our search function must return all todo entries which title or description contains the given search term. Also, our document class has two properties which we are using in this query. These properties are called title and description. We can create the method name which fulfils the requirements of our search function by following these steps:

Add the findBy prefix to start of the method name.
Add the property expression of the title property after the prefix.
Add the Contains keyword after the property expression.
Add the Or keyword to the method name.
Add the property expression of the description property after the Or keyword.
Add the Contains keyword to the method name.
Add two method parameters to our query method. The first parameter matched against the title property and second one is matched against the description property.
The source code of the TodoDocumentRepository interface looks as follows:

1
2
3
4
5
6
7
import org.springframework.data.solr.repository.SolrCrudRepository;
import java.util.List;
 
public interface TodoDocumentRepository extends SolrCrudRepository<TodoDocument, String> {
 
    public List<TodoDocument> findByTitleContainsOrDescriptionContains(String title, String description);
}
Note: This query method will not work if the search term contains more than one word.

Named Queries
Named queries are queries which are declared in a separate properties file and wired to the correct query method. The rules concerning the properties file used to declare the named queries are described in the following:

The default location of the properties file is META-INF/solr-named-queries.properties but we can configure the location by using the namedQueriesLocation property of the @EnableSolrRepositories annotation.
The key of each named query is created by using the following formula: [The name of the document class].[The name of the named query].
Named queries which are configured in the properties files are matched with the query methods of our repository interface by using the following rules:

The name of query method which executes the named query must be same than the name of the named query.
If the name of the query method is not the same than the name of the named query, the query method must be annotated with the @Query annotation and the name of the named query must be configured by using the name property of the @Query annotation.
We can implement query method with named queries by following these steps:

Specify the named query.
Create the query method.
These steps are described with more details in the following.

Specifying the Named Query
We can create a named query which fulfils the requirements of our search function by following these steps:

Create the properties file which contains the named queries.
Create a key for the named query by using the formula described earlier. Since the name of our document class is TodoDocument, the key of our named query is TodoDocument.findByNamedQuery.
Create the named query by using the Lucene query parser syntax. Because our query must return documents which title or description contains the given search term, our query is: title:*?0* OR description:*?0* (The ?0 is replaced with the value of the query method’s first parameter).
The content of the META-INF/solr-named-query.properties file looks as follows:

1
TodoDocument.findByNamedQuery=title:*?0* OR description:*?0*
Creating the Query Method
We can create the query method which uses the created named query by following these steps:

Add findByNamedQuery() method to the TodoDocumentRepository interface. This method returns a list of TodoDocument objects and takes a single String parameter called searchTerm.
Annotate the method with the @Query annotation and set the value of its name property to ‘TodoDocument.findByNamedQuery’. This step is not required since the name of the query method is the same than the name of the named query but I wanted to demonstrate the usage of the @Query annotation here.
The source code of the TodoDocumentRepository interface looks as follows:

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
import org.springframework.data.solr.repository.Query;
import org.springframework.data.solr.repository.SolrCrudRepository;
 
import java.util.List;
 
public interface TodoDocumentRepository extends SolrCrudRepository<TodoDocument, String> {
 
    @Query(name = "TodoDocument.findByNamedQuery")
    public List<TodoDocument> findByNamedQuery(String searchTerm);
}
@Query Annotation
The @Query annotation can be used to specify the query which is executed when a query method is called. We can create the query method which fulfils the requirements of our search function by following these steps:

Add findByQueryAnnotation() method to the TodoDocumentRepository interface. This method returns a list of TodoDocument objects and it has a single String parameter called searchTerm.
Annotate the method with the @Query annotation.
Set the executed query as the value of the @Query annotation. We can create the query by using the Lucene query parser syntax. Because our query must return documents which title or description contains the given search term, the correct query is: title:*?0* OR description:*?0* (The ?0 is replaced with the value of the query method’s first parameter).
The source code of the TodoDocumentRepository interface looks as follows:

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
import org.springframework.data.solr.repository.Query;
import org.springframework.data.solr.repository.SolrCrudRepository;
 
import java.util.List;
 
public interface TodoDocumentRepository extends SolrCrudRepository<TodoDocument, String> {
 
    @Query("title:*?0* OR description:*?0*")
    public List<TodoDocument> findByQueryAnnotation(String searchTerm);
}
Using the Created Query Method

We can use the created query method by following these steps:

Declare the search() method in the TodoIndexService interface.
Add the implementation of the search() method to the RepositoryTodoIndexService class.
These steps are described with more details in the following subsections.

Declaring the Search Method
Our first step is to declare the search method in the TodoIndexService interface. The relevant part of the TodoIndexService interface looks as follows:

1
2
3
4
5
6
import java.util.List;
 
public interface TodoIndexService {
 
    public List<TodoDocument> search(String searchTerm);
}
Implementing the Search Method
Our second step is to implement the search() method. Our implementation is rather simple. It obtains a list of TodoDocument objects by calling the correct method of the TodoDocumentRepository interface and returns that list.

The name of the query method depends from the technique used to create the query method. The different implementations are described in the following.

Query Generation from the Method Name
When we are generating the query from the name of the query method, our implementation obtains a list of TodoDocument objects by calling the findByTitleContainsOrDescriptionContains() method of the TodoDocumentRepository interface and returns that list.

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
import org.springframework.stereotype.Service;
import javax.annotation.Resource;
import java.util.List;
 
@Service
public class RepositoryTodoIndexService implements TodoIndexService {
 
    @Resource
    private TodoDocumentRepository repository;
 
    @Override
    public List<TodoDocument> search(String searchTerm) {
        return repository.findByTitleContainsOrDescriptionContains(searchTerm, searchTerm);
    }
}
Named Queries
If we are using named queries, our implementation gets a list of TodoDocument objects by calling the findByNamedQuery() method of the TodoDocumentRepository interface and returns that list.

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
import org.springframework.stereotype.Service;
import javax.annotation.Resource;
import java.util.List;
 
@Service
public class RepositoryTodoIndexService implements TodoIndexService {
 
    @Resource
    private TodoDocumentRepository repository;
 
    @Override
    public List<TodoDocument> search(String searchTerm) {
        return repository.findByNamedQuery(searchTerm);
    }
}
@Query Annotation
When we are using the @Query annotation, our implementation obtains a list of TodoDocument objects by calling the findByQueryAnnotation() method of the TodoDocumentRepository interface and returns that list.

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
import org.springframework.stereotype.Service;
import javax.annotation.Resource;
import java.util.List;
 
@Service
public class RepositoryTodoIndexService implements TodoIndexService {
 
    @Resource
    private TodoDocumentRepository repository;
 
    @Override
    public List<TodoDocument> search(String searchTerm) {
        return repository.findByQueryAnnotation(searchTerm);
    }
}
Selecting the Correct Query Creation Technique

The obvious question is:

What is the best way to add query methods to our Spring Data Solr repositories?
We can use the following guidelines when we are deciding the correct query creation technique for our query method:

If the created query is very simple, query generation from the method name is often the best choice. The problem of this approach is that implementing “complex” queries with this approach leads into long and ugly method names.
It is a good idea to keep our queries near our query methods. The benefit of using the @Query annotation is that we can see the executed query and our query method by reading the source code of our repository interface.
If we want to separate the executed queries from our repository interface, we should use named queries. The problem of this approach is that we have to check the executed query from the properties file which is quite cumbersome.
Summary

We have now learned how we can create static queries with Spring Data Solr. This blog entry has taught us two things:

We know what query method methods are and how we can create them.
We are familiar with the different query creation techniques and we know when to use them.
The example application which demonstrates the concepts described in this blog entry is available at Github. In the next part of this tutorial we will learn how we can add custom functionality to a single repository.

If you want to learn how to use Spring Data Solr, you should read my Spring Data Solr tutorial.
