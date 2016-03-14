The [previous part](http://www.petrikainulainen.net/programming/solr/spring-data-solr-tutorial-query-methods/) of my Spring Data Solr tutorial taught us how we can create static queries by using query methods. The natural next step would be describe how we can create dynamic queries with Spring Data Solr. However, before we can move on to that subject, we have to understand how we can add custom methods to a single repository.

This blog entry will help us to understand how that is done.

During this blog entry we will modify our [example application](https://github.com/pkainulainen/spring-data-solr-examples/tree/master/query-methods) to update the information of a todo entry to the Solr index by using a technique called partial update.

Let’s start by taking a closer look at Solr’s partial update feature.

## What is Partial Update?

The partial update feature was introduced in Solr 4.0 and it gives us the possibility to select the fields which are updated. This can be very useful if it is slow to index the content of the whole document.

However, the partial update function has its limitations. If we want to use the partial update function, we have to store all fields of the document which increases the size of the Solr index. The reason for this is that it is not possible to do a partial update to Lucene index. Lucene always deletes the old document before indexing the new one. This means that if the fields which are not updated are not stored, the values of these fields are lost when a partial update is done to a document.

It is our job to decide which one is more important to us: speed or the size of the index.

We can get more information about partial update by checking out the following resources:

* [Solr 4.0: Partial documents update](http://solr.pl/en/2012/07/09/solr-4-0-partial-documents-update/)
* [Solr – Partial Update of Documents @ StackOverflow](http://stackoverflow.com/questions/15161903/partial-update-of-documents)
* [Solr – update a new field to existing document @ StackOverflow](http://stackoverflow.com/questions/11791803/update-a-new-field-to-existing-document)

Let’s move and learn how we can add custom methods to a single Spring Data Solr repository.

## Adding Custom Methods a Single Repository

We can add custom methods to a single repository by following these steps:

1. Create a custom interface which declares the custom methods.
2. Implement the custom interface.
3. Modify the repository interface to extend the custom interface.

These steps are described with more details in the following subsections.

### Creating the Custom Interface

First, we have to create an interface and declare the custom methods in it. We can do this by following these steps:

1. Create an interface called PartialUpdateRepository.
2. Declare the custom methods.

Because we have to declare only one custom method which is used to update the information of a todo entry, the source code of the PartialUpdateRepository interface looks as follows:

```java
public interface PartialUpdateRepository {
 
    public void update(Todo todoEntry);
}
```

### Implementing the Custom Interface

Second, we have to implement the PartialUpdateRepository interface. The repository infrastructure tries to auto detect the classes which implements the custom repository interfaces by using the following rules:

1. The implementation of a custom repository interface must be found from the same package than the custom interface.
2. The name of the class which implements a custom repository interface must be created by using the following formula: [The name of the actual repository interface][The repository implementation postfix].

The default value of the repository implementation postfix is ‘Impl’. We can overwrite the default value by using one of the following methods:

* If we are using Java configuration, we can configure the used postfix by setting the preferred postfix as the value of the repositoryImplementationPostfix attribute of the [@EnableSolrRepositories](http://static.springsource.org/spring-data/data-solr/docs/1.0.0.RC1/api/org/springframework/data/solr/repository/config/EnableSolrRepositories.html) annotation.
* If we are using XML configuration, we can configure the used postfix by setting the preferred postfix as the value of the repository-impl-postfix attribute of the [repositories namespace element](http://static.springsource.org/spring-data/data-solr/docs/1.0.0.RC1/reference/htmlsingle/#namespace-dao-config)

The example application of this blog entry uses the default configuration. Thus, we can implement the PartialUpdateRepository interface by following these steps:

1. Create a class called TodoDocumentRepositoryImpl.
2. Annotate the class with the @Repository annotation.
3. Add SolrTemplate field to the class and annotate this field with the @Resource annotation.
4. Implement the update() method.

Let’s take a closer look at the implementation of the update() method. We can implement this method by following these steps:

1. Create a new PartialUpdate object. Set the name of the document’s id field and its value as constructor arguments.
2. Set the names and values of updated fields to the created object.
3. Do a partial update by calling the saveBean() method of the SolrTemplate class.
4. Commit changes by calling the commit() method of the SolrTemplate class.

The source code of the TodoRepositoryImpl class looks as follows:

```java
import org.springframework.data.solr.core.SolrTemplate;
import org.springframework.data.solr.core.query.PartialUpdate;
import org.springframework.stereotype.Repository;
 
import javax.annotation.Resource;
 
@Repository
public class TodoDocumentRepositoryImpl implements PartialUpdateRepository {
 
    @Resource
    private SolrTemplate solrTemplate;
 
    @Override
    public void update(Todo todoEntry) {
        PartialUpdate update = new PartialUpdate("id", todoEntry.getId().toString());
 
        update.add("description", todoEntry.getDescription());
        update.add("title", todoEntry.getTitle());
 
        solrTemplate.saveBean(update);
        solrTemplate.commit();
    }
}
```

### Modifying the Repository Interface
We can make the custom update() method visible to the users of our repository by extending the PartialUpdateRepository interface. The source code of TodoDocumentRepository interface looks as follows:

```java
import org.springframework.data.solr.repository.SolrCrudRepository;
 
public interface TodoDocumentRepository extends PartialUpdateRepository, SolrCrudRepository<TodoDocument, String> {
 
    //Query methods are omitted.
}
```

Let’s move on and find out how we can use our new repository method.

## Using the Custom Method

We can use the custom update() method by making the following changes to our example application:

1. Add update() method to the TodoIndexService interface.
2. Implement the update() method.
3. Modify the update() method of the RepositoryTodoService class to use the new method.

These steps are described with more details in the following subsections.

### Adding New Method to TodoIndexService Interface

As we remember, the TodoIndexRepository interface declares methods which are used to add information to the Solr index, search information from it and remove documents from the index.

We have to add a new method to this interface. This method is called update() and it takes the updated Todo object as a method parameter. The source code of the TodoIndexRepository interface looks as follows:

```java
public interface TodoIndexService {
 
    //Other methods are omitted.
 
    public void update(Todo todoEntry);
}
```

### Implementing the Added Method

We can implement the update() method of the TodoIndexService interface by following these steps:

1. Add the update() method to the RepositoryIndexService class and annotate the method with the @Transactional annotation. This ensures that our [Spring Data Solr repository will participate in Spring managed transactions](http://static.springsource.org/spring-data/data-solr/docs/1.0.0.RC1/reference/htmlsingle/#solr.transactions).
2. Call the update() repository method and pass the updated Todo object as a method parameter.

The source code of the RepositoryTodoIndexService class looks as follows:

```java
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
 
import javax.annotation.Resource;
 
@Service
public class RepositoryTodoIndexService implements TodoIndexService {
 
    @Resource
    private TodoDocumentRepository repository;
 
    //Other fields and methods are omitted.
 
    @Transactional
    @Override
    public void update(Todo todoEntry) {
        repository.update(todoEntry);
    }
}
```

### Modifying the RepositoryTodoService Class

Our last step is to modify the update() method of the RepositoryTodoService class to use the new update() method which is declared in the TodoIndexService interface. The relevant parts of the RepositoryTodoService class looks as follows:

```java
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
 
import javax.annotation.Resource;
 
@Service
public class RepositoryTodoService implements TodoService {
 
    @Resource
    private TodoIndexService indexService;
 
    @Resource
    private TodoRepository repository;
 
    //Other methods are omitted.
 
    @PreAuthorize("hasPermission('Todo', 'update')")
    @Transactional(rollbackFor = {TodoNotFoundException.class})
    @Override
    public Todo update(TodoDTO updated) throws TodoNotFoundException {
        Todo model = findById(updated.getId());
 
        model.update(updated.getDescription(), updated.getTitle());
 
        indexService.update(model);
 
        return model;
    }
}
```

## Summary

We have now added a custom method to a single Spring Data Solr repository and implemented an update function which uses the partial update feature of Solr. This tutorial has taught us two things:

* We know how we can add custom methods to a single Spring Data Solr repository.
* We know that we can use partial update only if all fields of our document are stored (The value of the stored attribute is true).

The next part of my Spring Data Solr tutorial describes how we can use the skills learned from this blog entry for [creating dynamic queries with Spring Data Solr](http://www.petrikainulainen.net/programming/solr/spring-data-solr-tutorial-dynamic-queries/).

P.S. The example application of this blog entry is available at [Github](https://github.com/pkainulainen/spring-data-solr-examples/tree/master/query-methods).

If you want to learn how to use Spring Data Solr, you should read my [Spring Data Solr tutorial](http://www.petrikainulainen.net/spring-data-solr-tutorial/).
