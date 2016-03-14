[前文](https://github.com/ronniewang/blog/blob/master/Spring-Data-Solr/tutorial-four(configuration).md)配置了Spring Data Solr，下面看一下怎样管理存储在Solr实例中的信息，本篇将会介绍索引文档的增加，修改，删除等

在[Github的例子](https://github.com/pkainulainen/spring-data-solr-examples/tree/master/query-methods)上面进行如下修改：

1. 新建document类
2. 新建repository接口
3. 新建一个使用repository的service
4. 使用service

具体步骤请往下看

## 新建Document类

Document类是符合下面约束的POJO类：

* The @Field annotation is used to create a link between the fields of the POJO and the fields of the Solr document.
* If the name of the bean’s field is not equal to the name of the document’s field, the name of the document’s field must be given as a value of the @Field annotation.
* `@Field`可以用在setter方法或者字段上
* Spring Data Solr assumes by default that the name of the document’s id field is ‘id’. We can override this setting by annotating the id field with the @Id annotation.
* Spring Data Solr (version 1.0.0.RC1)要求文档id是`String`类型的

*更多信息*

* [Solrj @ Solr Wiki](http://wiki.apache.org/solr/Solrj#Directly_adding_POJOs_to_Solr)

下面看看如何创建document类吧

[Solr简介](http://www.petrikainulainen.net/programming/solr/spring-data-solr-tutorial-introduction-to-solr/)一文中我们知道，我们需要在todo中存储id, description和title信息

通过下面步骤新建todo类：

1. 新建`TodoDocument`类
2. 添加`id`字段，加上`@Field`和`@Id`注解(如果字段名也是id，`@Id`并不是必须的).
3. 添加`description`字段，加上`@Field`注解
4. 添加`title`字段，加上`@Field`注解
5. 添加getter方法
6. Create a static inner class which is used to build new TodoDocument objects.
7. Add a static getBuilder() method to the TodoDocument class. The implementation of this method returns a new TodoDocument.Builder object.

`TodoDocument`代码如下：

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

## 新建Repository接口

Spring Data Solr的基础接口是SolrCrudRepository<T, ID>，每个repository都要继承它

SolrCrudRepository<T, ID>有两个类型参数：

* T表示document类
* ID表示document的id的类型，Spring Data Solr (version 1.0.0.RC1)要求为String

通过下面步骤新建repository：

1. 新建`TodoDocumentRepository`接口
2. 继承`SolrCrudRepository`接口，为类型参数赋值

`TodoDocumentRepository`代码如下：

```java
import org.springframework.data.solr.repository.SolrCrudRepository;
 
    public interface TodoDocumentRepository extends SolrCrudRepository<TodoDocument, String> {
}
```

## 新建Service

下面新建一个使用repository的service：

1. 新建service接口
2. 实现这个接口

具体步骤如下

### 新建service接口

我们接口有如下两个方法：

* `void addToIndex(Todo todoEntry)`方法向索引中增加一个todo
* `void deleteFromIndex(Long id)`方法从索引中删除一个todo
 
> Note: We can use the addToIndex() method for adding new todo entries to the Solr index and updating the information of existing todo entries. If an existing document has the same id than the new one, the old document is deleted and the information of the new document is saved to the Solr index (See [SchemaXML @ Solr Wiki](http://wiki.apache.org/solr/SchemaXml#The_Unique_Key_Field) for more details).

`TodoIndexService`代码如下：

```java
public interface TodoIndexService {
 
    public void addToIndex(Todo todoEntry);
 
    public void deleteFromIndex(Long id);
}
```

### 实现接口

步骤如下：

1. 建立service骨架代码
2. 实现`void addToIndex(Todo todoEntry)`方法
3. 实现`void deleteFromIndex(Long id)`方法

具体步骤如下：

#### 建立service骨架代码

步骤如下：

1. 新建`RepositoryTodoIndexService`类，加上`@Service`注解
2. 添加`TodoDocumentRepository`注解，加上`@Resource`注解

目前代码如下：

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

#### 实现`void addToIndex(Todo todoEntry)`方法

步骤如下：

1. Add the addToIndex() method to the RepositoryTodoIndexService class and annotate this method with the @Transactional annotation. This ensures that our [Spring Data Solr repository will participate in Spring managed transactions](http://static.springsource.org/spring-data/data-solr/docs/1.0.0.RC1/reference/htmlsingle/#solr.transactions).
2. Create a new TodoDocument object by using the builder pattern. Set the id, title and description of the created document.
3. Add the document to the Solr index by calling the save() method of the TodoDocumentRepository interface.

方法代码如下：

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

#### 实现`void deleteFromIndex(Long id)`方法

步骤如下：

1. Add the deleteFromIndex() method to the RepositoryTodoDocumentService class and annotate this method with the @Transactional annotation. This ensures that our [Spring Data Solr repository will participate in Spring managed transactions](http://static.springsource.org/spring-data/data-solr/docs/1.0.0.RC1/reference/htmlsingle/#solr.transactions).
2. Delete document from the Solr index by calling the delete() method of the TodoDocumentRepository interface.

方法代码如下：

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

`RepositoryTodoService`代码如下：

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

## 总结

本篇我们知道了如下事情：

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
