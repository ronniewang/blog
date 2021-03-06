本篇大概介绍一下Solr，分为如下三个部分：

1. 第一部分对Solr和它的数据模型有个大概的了解
2. 第二部分为我们的Solr instance创建一个Schema
3. 第三部分介绍通过介绍Solr提供的REST-like HTTP API

## Solr概述

Let’s start by getting a brief introduction to the Solr search server. This introduction is very thin and it provides only the information we need to know in order to understand the implementation of our search function.

Also, even though [Solr](http://lucene.apache.org/solr/) is heavily dependent from [Lucene](http://lucene.apache.org/core/), this blog entry makes no difference between them.

This section describes

* Solr的Data Model
* 新文档加入Solr时发生了什么
* 在索引上索引执行查询时发生了什么

### Solr的Data Model

一个索引An index consists of documents which are essentially a collection of fields. 如果我们拿Solr的data model和关系数据库进行对比的化，会发现如下的相似性：

* 一个索引类似与一个数据库表
* 一个文档类似于数据库表的一行
* 一个field类似于一个数据库表的一列

Each field of a document can be either indexed, stored or both. The meaning of these terms is described in the following:

* 一个被索引的field是可搜索和可排序的，被索引的field不在搜索结果中体现Indexed fields are not returned in the search results.
* 被存储的field会在搜索结果中返回
* 如果一个field既是被索引的又是被存储的，那么这个field就是既是可搜索和可排序的，又会在搜索结果中返回

### 在索引中增加信息

当一个新文档被加入Solr时，被索引的和被存储的field的处理方式是不同的，区别如下：

* Indexed fields undergo an analysis phase which typically breaks the text into words and apply different transformations to it. The results of this analysis phase are saved to Solr index.
* The values of stored fields are saved as is.

### 从索引中搜索信息

搜索可以通过下面三个步骤来描述：

* Typically the search query undergo a similar analysis phase than the indexed fields. The goal of this is to ensure that the search query matches with the content of the index.
* Solr uses its index to perform the search.
* The matching documents are returned in the requested format. Each document contains the values of its stored fields.

### 建立Schema

schema用于配置下面的内容：

* The fields of a document.
* How the fields of document are processed when a new document is added to the index.
* How the fields are dealt with when a search is executed against the index.

The schema is configured in a file called schema.xml, and we can create a schema for our application by following these steps:

1. Configure the used field types.
2. Configure the fields of our document.
3. Configure the copy fields.
4. Configure the unique key field of our document.

These steps are described with more details in the following subsections. The skeleton schema of our Solr instance looks as follows:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<schema name="todo" version="1.5">
    <fields>
        <!-- Configure fields here -->
    </fields>
 
    <!-- Configure unique key -->
    <!-- Configure copy fields here -->
     
    <types>
        <!-- Configure field types here -->
    </types>
</schema>
```

Note: This section describes the schema of the example application of my blog entry called [Running Solr with Maven待修改](http://www.petrikainulainen.net/programming/maven/running-solr-with-maven/).

### Configuring the Field Types

A field type specifies the following things:

* The data type of the field.
* How the information is analyzed when it is added to the index.
* How the information is processed when information is searched from the index.

We can configure a field type by using the fieldType element. Its attributes which are used in our schema are described in the following:

* The name attribute states the name of the field type. It is basically an alias which is used declare the type of a field.
* The class attribute declares the class that implements the field type in question.
* The sortMissingLast attribute specifies how the sorting works when the value of this field is missing. If the value of this attribute is set to ‘true’, the documents which don’t have value in the field in question are returned last.
* The positionIncrementGap attribute declares the amount of empty space which is put between multiple fields of the same document. The value of this attribute is used in multivalued fields and its idea is to prevent false matches across different fields.
* The precisionStep attribute is used in ranged queries of numeric fields. We can get more information about this by reading the [API documentation of the NumericRangeQuery class](http://lucene.apache.org/core/4_1_0/core/org/apache/lucene/search/NumericRangeQuery.html).

We can configure the field types of our schema by following these steps:

1. Configure the Field Type for Long Fields
2. Configure the Field Type for String Fields
3. Configure the Field Type for Fields Containing Text

These steps are described with more details in the following.

#### Configuring the Field Type for Long Fields

Let’s start by configuring a simple field type for long fields which are not analyzed on the index or search phase. We can configure this field type by following these steps:

1. Set the name of the field type to ‘long’.
2. Set the name of the implementing class to ‘solr.TrieLongField’.
3. Set the value of the precisionStep attribute to zero.
4. Set the value of the positionIncrementGap attribute to zero.

Our field type declaration looks as follows:

```xml
<fieldType name="long" class="solr.TrieLongField" precisionStep="0" positionIncrementGap="0"/>
```

#### Configuring the Field Type for String Fields

The next step is to configure a simple field type for string fields which are not analyzed on the index or search phase. We can configure this field type by following these steps:

1. Set the name of the field type to ‘string’.
2. Set the name of the implementing class to ‘solr.StrField’.
3. Set the value of the sortMissingLast attribute to ‘true’.

The declaration of our string field looks as follows:

```xml
<fieldType name="string" class="solr.StrField" sortMissingLast="true" />
```

#### Configuring the Field Type for Fields Containing Text

The last step is to configure the text_general field type. We can do this by following these steps:

1. Create a new field type. Set the name of the field type to ‘text_general’. Set the name of the implementing class to ‘solr.TextField’. Set the value of the positionIncrementGap to 100.
2. Create a new analyzer which is run at the index time. Configure that the text is splitted into words by using the word break rules of [the Unicode Text Segmentation algorithm](http://unicode.org/reports/tr29/). Create a filter which removes words found from stopwords.txt file from the text. Transform text to lower case.
3. Create a new analyzer which is run at the query phase. Configure that the search query is splitted into words by using the word break rules of [the Unicode Text Segmentation algorithm](http://unicode.org/reports/tr29/). Create a filter which removes words found from stopwords.txt file from the search query. Ensure that the synonyms found from synonyms.txt are applied. Transform the search query to lower case.

The declaration of the text_general field type looks as follows:

```xml
<fieldType name="text_general" class="solr.TextField" positionIncrementGap="100">
    <!-- Configures the analysis done at the index phase -->
    <analyzer type="index">
        <!-- Uses word break rules of the Unicode Text Segmentation algorith when splitting text into words. -->
        <tokenizer class="solr.StandardTokenizerFactory"/>
        <!-- Removes words found from stopwords.txt file. This filter is case insensitive. -->
        <filter class="solr.StopFilterFactory" ignoreCase="true" words="stopwords.txt" enablePositionIncrements="true" />
        <!-- Transforms text to lower case -->
        <filter class="solr.LowerCaseFilterFactory"/>
    </analyzer>
    <!-- Configures the analysis done at the query time -->
    <analyzer type="query">
        <!-- Uses word break rules of the Unicode Text Segmentation algorith when splitting text into words. -->
        <tokenizer class="solr.StandardTokenizerFactory"/>
        <!-- Removes words found from stopwords.txt file. This filter is case insensitive. -->
        <filter class="solr.StopFilterFactory" ignoreCase="true" words="stopwords.txt" enablePositionIncrements="true" />
        <!-- Applies synonyms found from the synonyms.txt file. -->
        <filter class="solr.SynonymFilterFactory" synonyms="synonyms.txt" ignoreCase="true" expand="true"/>
        <!-- Transforms text to lower case -->
        <filter class="solr.LowerCaseFilterFactory"/>
    </analyzer>
</fieldType>
```

We can get more information about the analysis phase by reading the following documents:

* [Analyzers, Tokenizers and Token Filters.](http://wiki.apache.org/solr/AnalyzersTokenizersTokenFilters)

### Configuring the Fields of Our Document

We can add new fields to our document by adding field elements to the schema.xml file. [The field element has many attributes](http://wiki.apache.org/solr/SchemaXml#Common_field_options) but at this point we need to understand the meaning of following attributes:

* The name attribute specifies the name of the field.
* The indexed attribute (true/false) specifies if the field is added to the search index. Only indexed fields are searchable and sortable.
* The stored attribute(true/false) specifies if the field should be returned in the search results.
* The multiValued (true/false) specifies if the field can appear in the document more than once.
* The type type attribute specifies the type of the field.
* The required (true/false) specifies whether the field is required or not.

In order to make maximize the performance of our Solr instance, we must follow the following guidelines:

* We should not store fields which are not required in the search results.
* We should not index the fields which are not used by our search function.

We can get more information about the optimal field configuration by reading the following documents:

* [Field Options by Use Case](http://wiki.apache.org/solr/FieldOptionsByUseCase)
* [Solr Performance Factors](http://wiki.apache.org/solr/FieldOptionsByUseCase)

We are now ready to configure the actual fields of our schema. Let’s first talk a bit about the information which we need show on the search result page. This information is described in the following:

* We need the id of the todo entry which is used to create a link to the view todo entry page.
* We need the title of the todo entry which is used as an anchor text of the created link.

When we know that our application must be able to search the contents of the title and description of a todo entry, we can add the required fields to our schema by following these steps:

1. Add a required field called ‘id’ to the schema and set its type to ‘string’. Ensure that this field is both indexed and stored. Set the value of the multiValued attribute to false.
2. Add a required field called ‘title’ to the schema and its type to ‘text_general’. Configure this field to be both indexed and stored. Set the value of the multivalued attribute to false.
3. Add a field called ‘description’ the schema sets its type to ‘text_general’. Configure this field to be indexed but not stored. Set the value of the multiValued attribute to false.
4. Add a field called ‘text’ and set its type ‘text_general’. Configure this field to be indexed but not stored. Set the value of its multiValue attribute to true. This field is a field which stores the content of all other indexed text fields.
5. Add a field called ‘_version_’ and sets its type ‘long’. Configure this field to be indexed and stored.
Our field declarations looks as follows:

```xml
<field name="id" type="string" indexed="true" stored="true" required="true" multiValued="false" />
<field name="title" type="text_general" indexed="true" stored="true" required="true" multiValued="false"/>
<field name="description" type="text_general" indexed="true" stored="false" multiValued="false"/>
<field name="text" type="text_general" indexed="true" stored="false" multiValued="true"/>
<field name="_version_" type="long" indexed="true" stored="true"/>
```

### Configuring the Copy Fields

We use [copy fields](http://wiki.apache.org/solr/SchemaXml#Copy_Fields) to copy the contents of title and description fields to the text field. We can configure the copy fields by using the copyField element. We can create the required configuration by following these steps:

1. Create a copy field which copies the value of the ‘title’ field to the ‘text’ field.
2. Create a copy field which copies the value of the ‘description’ field to the ‘text’ field.

The declaration of our copy fields looks as follows:

```xml
<copyField source="title" dest="text"/>
<copyField source="description" dest="text"/>
```

### Configuring the Unique Key Field of Our Document

The [unique key](http://wiki.apache.org/solr/SchemaXml#The_Unique_Key_Field) is a field which should be unique for all documents. It is not mandatory to specify the unique key of a document but if we decide to do so, it means that the index cannot contain two documents which both have same value in the field configured as the unique key.

In our case, we use the field ‘id’ as the unique key of our document. We can make this configuration by adding the following XML to the `schema.xml` file:

```xml
<uniqueKey>id</uniqueKey>
```

## Using the REST-like HTTP API

Solr offers a REST-like HTTP API which we can use to add information to Solr and execute search queries against its index. Both of these use cases are described in the following.

> Note: This section assumes that we are using the example application of my blog entry called [Running Solr with Maven](http://www.petrikainulainen.net/programming/maven/running-solr-with-maven/).

### Adding Information to Solr

We can add new information to Solr by following these steps:

1. Send a POST request to the url ‘http://localhost:8983/solr/update/json?commit=true’.
2. Set content type of the request to ‘application/json’.
3. Sending the added information in the body of the request as JSON.

The contents of our request body looks as follows:

```
[
    {
        "id":"1",
        "title":"Write introduction to Solr",
        "description":"This blog entry provides an introduction to Solr search server"
    },
    {
        "id":"2",
        "title":"Implement example application",
        "description":"This application demonstrates the usage of spring-data-solr."
    }
]
```

We have now added two documents to our Solr index by using the REST-like API provided by Solr.

However, it is good to know that there are other options which we can use to add information to the Solr index. These options are described in the following documents:

* [POST JSON documents](http://wiki.apache.org/solr/UpdateJSON)
* [Import records from a database](http://wiki.apache.org/solr/DataImportHandler)
* [Load data from a CSV file](http://wiki.apache.org/solr/UpdateCSV)
* [Index binary documents](http://wiki.apache.org/solr/ExtractingRequestHandler)
* [Use SolrJ](http://wiki.apache.org/solr/Solrj)

### Searching Information from Solr Index

We are now ready to search the information stored in the index of our Solr instance. We can execute search queries against the Solr index by following these guidelines:

* The search query is executed by sending a GET request to the url ‘http://localhost:8983/solr/todo/select’.
* The query string must be set as the value of the [q request parameter](http://wiki.apache.org/solr/CommonQueryParameters#q).
* The format of the query results must be set as the value of the [wt request parameter](http://wiki.apache.org/solr/CoreQueryParameters#wt).

Let’s move on and find out how we can list all documents found from the index and execute a simple search query against the indexed data.

#### Finding All Documents of the Index

We can list all documents in JSON format by following these steps:

1. Send a GET request to the url ‘http://localhost:8983/solr/todo/select’.
2. Set the value of q request parameter to ‘*.*’
3. Set the value of the wt request parameter to ‘json’.

When we send a GET request to the url ‘http://localhost:8983/solr/todo/select?q=*%3A*&wt=json’, we should receive the following JSON:

```
{
    "responseHeader": {
        "status":0,
        "QTime":1,
        "params":{
            "wt":"json",
            "q":"*:*"
        }
    },
    "response":{
        "numFound":2,
        "start":0,
        "docs":[
            {
                "id":"1",
                "title":"Write introduction to Solr",
                "_version_":1425949176574771200
            },
            {
                "id":"2",
                "title":"Implement example application",
                "_version_":1425949176662851584
            }
        ]
    }
}
```

#### Searching Information from the Index

We can search all documents which title or description contains the word ‘application’ by following these steps:

1. Send a GET request to the url ‘http://localhost:8983/solr/todo/select’.
2. Set the value of q request parameter to ‘application’
3. Set the value of the wt request parameter to ‘json’.

When we send a GET request tot he url ‘http://localhost:8983/solr/todo/select?q=application&wt=json’, we should receive the following JSON:

```
{
    "responseHeader":{
        "status":0,
        "QTime":7,
        "params":{
            "wt":"json",
            "q":"application"
        }
    },
    "response":{
        "numFound":1,
        "start":0,
        "docs":[
            {
                "id":"2",
                "title":"Implement example application",
                "_version_":1425949176662851584
            }
        ]
    }
}
```

## The End

We have now obtained the information which is required to understand the concepts described in the next parts of my Spring Data Solr tutorial. This blog entry has teached us three things:

* We know the basics of the Solr data model.
* We know how we can configure the schema of our Solr instance.
* We know how we can add documents to the Solr index and search information from it by using the HTTP API of Solr.

The next part of this tutorial describes how we can [get the required dependencies with Maven and configure Spring Data Solr](http://www.petrikainulainen.net/programming/solr/spring-data-solr-tutorial-configuration/).

> If you want to learn how to use Spring Data Solr, you should read my [Spring Data Solr tutorial](http://www.petrikainulainen.net/spring-data-solr-tutorial/).
