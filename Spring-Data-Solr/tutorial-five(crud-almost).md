In the [previous part](http://www.petrikainulainen.net/programming/solr/spring-data-solr-tutorial-configuration/) of my Spring Data Solr tutorial, we learned how we can configure Spring Data Solr. Now it is time to take a step forward and learn how we can manage the information stored in our Solr instance. This blog entry describes how we add new documents to the Solr index, update the information of existing documents and delete documents from the index.

We can make the necessary modifications to our [example application](https://github.com/pkainulainen/spring-data-solr-examples/tree/master/query-methods) by following these steps:

1. Create a document class which contains the information stored in the Solr index.
2. Create a repository interface for our Spring Data Solr repository.
3. Create a service which uses the created repository.
4. Use the created service.

These steps are described with more details in the following sections.

## Creating the Document Class

The first step is to create a document class which contains the information added to the Solr index. A document class is basically just a POJO which is implemented by following these rules:

* The @Field annotation is used to create a link between the fields of the POJO and the fields of the Solr document.
* If the name of the bean’s field is not equal to the name of the document’s field, the name of the document’s field must be given as a value of the @Field annotation.
* The @Field annotation can be applied either to a field or setter method.
* Spring Data Solr assumes by default that the name of the document’s id field is ‘id’. We can override this setting by annotating the id field with the @Id annotation.
* Spring Data Solr (version 1.0.0.RC1) requires that the type of the document’s id is String.

_More information:_

* [Solrj @ Solr Wiki](http://wiki.apache.org/solr/Solrj#Directly_adding_POJOs_to_Solr)

Let’s move on and create our document class.

In the [first part](http://www.petrikainulainen.net/programming/solr/spring-data-solr-tutorial-introduction-to-solr/) of my Spring Data Solr tutorial, we learned that we have to store the id, description and title of each todo entry to the Solr index.

Thus, we can create a document class for todo entries by following these steps:

1. Create a class called TodoDocument.
2. Add the id field to the TodoDocument class and annotate the field with the @Field annotation. Annotate the field with the @Id annotation (This is not required since the name of the id field is ‘id’ but I wanted to demonstrate its usage here).
3. Add the description field to the TodoDocument class and annotate this field with the @Field annotation.
4. Add the title field to the TodoDocument and annotate this field with the @Field annotation.
5. Create getter methods to the fields of the TodoDocument class.
6. Create a static inner class which is used to build new TodoDocument objects.
7. Add a static getBuilder() method to the TodoDocument class. The implementation of this method returns a new TodoDocument.Builder object.

The source code of the TodoDocument class looks as follows:

```java
import org.apache.solr.client.solrj.beans.Field;
import org.springframework.data.annotation.Id;
 
public class TodoDocument {
 
    @Id
    @Field
    private String id;
 
    @Field
    private String description;
 
    @Field
    private String title;
 
    public TodoDocument() {
 
    }
 
    public static Builder getBuilder(Long id, String title) {
        return new Builder(id, title);
    }
 
    //Getters are omitted
 
    public static class Builder {
        private TodoDocument build;
 
        public Builder(Long id, String title) {
            build = new TodoDocument();
            build.id = id.toString();
            build.title = title;
        }
 
        public Builder description(String description) {
            build.description = description;
            return this;
        }
 
        public TodoDocument build() {
            return build;
        }
    }
}
```

## Creating the Repository Interface

The base interface of Spring Data Solr repositories is the SolrCrudRepository<T, ID> interface and each repository interface must extend this interface.

When we extend the SolrCrudRepository<T, ID> interface, we must give two type parameters which are described in the following:

* The T type parameter means the type of our document class.
* The ID type parameter means the type of the document’s id. Spring Data Solr (version 1.0.0.RC1) requires that the id of a document is String.

We can create the repository interface by following these steps:

1. Create an interface called TodoDocumentRepository.
2. Extend the SolrCrudRepository interface and give the type of our document class and its id as type parameters.

The source code of the TodoDocumentRepository interface looks as follows:

```java
import org.springframework.data.solr.repository.SolrCrudRepository;
 
    public interface TodoDocumentRepository extends SolrCrudRepository<TodoDocument, String> {
}
```

## Creating the Service

Our next step is to create the service which uses the created Solr repository. We can create this service by following these steps:

1. Create a service interface.
2. Implement the created interface.

These steps are described with more details in the following.

### Creating the Service Interface

Our service interface declares two methods which are described in the following:

* The void addToIndex(Todo todoEntry) method adds a todo entry to the index.
* The void deleteFromIndex(Long id) method deletes a todo entry from the index.
 
> Note: We can use the addToIndex() method for adding new todo entries to the Solr index and updating the information of existing todo entries. If an existing document has the same id than the new one, the old document is deleted and the information of the new document is saved to the Solr index (See [SchemaXML @ Solr Wiki](http://wiki.apache.org/solr/SchemaXml#The_Unique_Key_Field) for more details).

The source code of the TodoIndexService interface looks as follows:

```java
public interface TodoIndexService {
 
    public void addToIndex(Todo todoEntry);
 
    public void deleteFromIndex(Long id);
}
```

### Implementing the Created Interface

We cam implement the service interface by following these steps:

1. Create a skeleton implementation of our service class.
2. Implement the method used to add documents to the Solr index.
3. Implement the method used to delete documents from the Solr index.

These steps are described with more details in the following.

#### Creating a Skeleton Implementation of the Service Class

We can create a skeleton implementation of our service interface by following these steps:

1. Create a class called RepositoryTodoIndexService and annotate this class with the @Service annotation. This annotation marks this class as a service and ensures that the class will be detected during the classpath scanning.
2. Add a TodoDocumentRepository field to the RepositoryTodoIndexService class and annotate that field with the @Resource annotation. This annotation instructs the Spring IoC container to inject the actual repository implementation the service’s repository field.

The source code of our dummy service implementation looks as follows:

```java
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
 
import javax.annotation.Resource;
 
@Service
public class RepositoryTodoIndexService implements TodoIndexService {
 
    @Resource
    private TodoDocumentRepository repository;
 
    //Add methods here
}
```

#### Adding Documents to the Solr Index

We can create the method which adds new documents to the Solr index by following these steps:

1. Add the addToIndex() method to the RepositoryTodoIndexService class and annotate this method with the @Transactional annotation. This ensures that our [Spring Data Solr repository will participate in Spring managed transactions](http://static.springsource.org/spring-data/data-solr/docs/1.0.0.RC1/reference/htmlsingle/#solr.transactions).
2. Create a new TodoDocument object by using the builder pattern. Set the id, title and description of the created document.
3. Add the document to the Solr index by calling the save() method of the TodoDocumentRepository interface.

The source code of the created method looks as follows:

```java
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
 
import javax.annotation.Resource;
 
@Service
public class RepositoryTodoIndexService implements TodoIndexService {
 
    @Resource
    private TodoDocumentRepository repository;
 
    @Transactional
    @Override
    public void addToIndex(Todo todoEntry) {
        TodoDocument document = TodoDocument.getBuilder(todoEntry.getId(), todoEntry.getTitle())
                .description(todoEntry.getDescription())
                .build();
         
        repository.save(document);
    }
 
    //Add deleteFromIndex() method here
}
```

#### Deleting Documents from the Solr Index

We can create a method which deletes documents from the Solr index by following these steps:

1. Add the deleteFromIndex() method to the RepositoryTodoDocumentService class and annotate this method with the @Transactional annotation. This ensures that our [Spring Data Solr repository will participate in Spring managed transactions](http://static.springsource.org/spring-data/data-solr/docs/1.0.0.RC1/reference/htmlsingle/#solr.transactions).
2. Delete document from the Solr index by calling the delete() method of the TodoDocumentRepository interface.

The source code of the created method looks as follows:

```java
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
 
import javax.annotation.Resource;
 
@Service
public class RepositoryTodoIndexService implements TodoIndexService {
 
    @Resource
    private TodoDocumentRepository repository;
 
    //Add addToIndex() method here
 
    @Transactional
    @Override
    public void deleteFromIndex(Long id) {
        repository.delete(id.toString());
    }
}
```

## Using the Created Service

Our last step is to use the service which we created earlier. We can do this by making the following modifications to the RepositoryTodoService class:

1. Add the TodoIndexService field to the the RepositoryTodoService class and annotate this field with the @Resource annotation. This annotation instructs the Spring IoC container to inject the created RepositoryTodoIndexService object to the service’s indexService field.
2. Call the addToIndex() method of the TodoIndexService interface in the add() method of the RepositoryTodoService class.
3. Call the deleteFromIndex() method of the TodoIndexService interface in the deleteById() method of the RepositoryTodoService class.
4. Call the addToIndex() method of the TodoIndexService interface in the update() method of the RepositoryTodoService class.

The source code of the RepositoryTodoService looks as follows:

```java
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
 
import javax.annotation.Resource;
import java.util.List;
 
@Service
public class RepositoryTodoService implements TodoService {
 
    @Resource
    private TodoIndexService indexService;
 
    @Resource
    private TodoRepository repository;
 
    @PreAuthorize("hasPermission('Todo', 'add')")
    @Transactional
    @Override
    public Todo add(TodoDTO added) {
        Todo model = Todo.getBuilder(added.getTitle())
                .description(added.getDescription())
                .build();
 
        Todo persisted = repository.save(model);
        indexService.addToIndex(persisted);
 
        return persisted;
    }
 
    @PreAuthorize("hasPermission('Todo', 'delete')")
    @Transactional(rollbackFor = {TodoNotFoundException.class})
    @Override
    public Todo deleteById(Long id) throws TodoNotFoundException {
        Todo deleted = findById(id);
 
        repository.delete(deleted);
        indexService.deleteFromIndex(id);
 
        return deleted;
    }
 
    @PreAuthorize("hasPermission('Todo', 'update')")
    @Transactional(rollbackFor = {TodoNotFoundException.class})
    @Override
    public Todo update(TodoDTO updated) throws TodoNotFoundException {
        Todo model = findById(updated.getId());
 
        model.update(updated.getDescription(), updated.getTitle());
        indexService.addToIndex(model);
 
        return model;
    }
}
```

## Summary

We have successfully created an application which adds documents to the Solr index and deletes documents from it. This blog entry has taught us the following things:

* We learned how we can create document classes.
* We learned that we can create Spring Data Solr repositories by extending the SolrCrudRepository interface.
* We learned that Spring Data Solr assumes by default that the name of the document’s id field is ‘id’. However, we can override this setting by annotating the id field with the @Id annotation.
* We learned that at the moment Spring Data Solr (version 1.0.0.RC1) expects that the id of a document is String.
* We learned how we can add documents to the Solr index and delete documents from it.
* We learned that Spring Data Solr repositories can participate in Spring managed transactions.

The next part of my Spring Data Solr tutorial describes how we can [search information from the Solr index by using query methods](http://www.petrikainulainen.net/programming/solr/spring-data-solr-tutorial-query-methods/).

P.S. The example application of this blog entry is available at [Github](https://github.com/pkainulainen/spring-data-solr-examples/tree/master/query-methods).

If you want to learn how to use Spring Data Solr, you should read my [Spring Data Solr tutorial](http://www.petrikainulainen.net/spring-data-solr-tutorial/).

* [Running Solr with Maven](https://github.com/ronniewang/blog/blob/master/Spring-Data-Solr/tutorial-two(running-solr-with-maven).md) 
* [Spring Data Solr Tutorial: Introduction to Solr](https://github.com/ronniewang/blog/blob/master/Spring-Data-Solr/tutorial-three(introduction-to-solr).md) 
* [Spring Data Solr Tutorial: Configuration](https://github.com/ronniewang/blog/blob/master/Spring-Data-Solr/tutorial-four(configuration).md) 
* [Spring Data Solr Tutorial: CRUD (Almost)](https://github.com/ronniewang/blog/blob/master/Spring-Data-Solr/tutorial-five(crud-almost).md) 
* [Spring Data Solr Tutorial: Query Methods](https://github.com/ronniewang/blog/blob/master/Spring-Data-Solr/tutorial-six(query-methods).md) 
* [Spring Data Solr Tutorial: Adding Custom Methods to a Single Repository](https://github.com/ronniewang/blog/blob/master/Spring-Data-Solr/tutorial-seven(adding-custom-methods-to-a-single-repository).md) 
* [Spring Data Solr Tutorial: Dynamic Queries](https://github.com/ronniewang/blog/blob/master/Spring-Data-Solr/tutorial-eight(dynamic).md) 
* [Spring Data Solr Tutorial: Sorting](https://github.com/ronniewang/blog/blob/master/Spring-Data-Solr/tutorial-nine(sorting).md) 
* [Spring Data Solr Tutorial: Pagination](https://github.com/ronniewang/blog/blob/master/Spring-Data-Solr/tutorial-ten(pagination).md) 
* [Spring Data Solr Tutorial: Adding Custom Methods to All Repositories](https://github.com/ronniewang/blog/blob/master/Spring-Data-Solr/tutorial-eleven(adding-custom-methods-to-all-repositories).md) 
