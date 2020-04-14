---
layout: post
title: 《Elasticsearch技术解析与实战》Chapter 1.4 Spring Boot整合Elasticsearch
date: 2019-04-16 11:34:52
categories: Java
tags:
- Elasticsearch
- 读书笔记
description: 《Elasticsearch技术解析与实战》Chapter 1.4 Spring Boot整合Elasticsearch
---

# 1. spring-boot-starter-data-elasticsearch
## 1.1 pom.xml和application.yml
```java
<!-- Spring Boot Elasticsearch 依赖 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
</dependency>
spring:
  data:
    elasticsearch:
      repositories:
        enabled: true
      cluster-name: docker-cluster
      cluster-nodes: lujiahao.ml:9300
```

## 1.2 创建Repository
```java
@Repository
public interface PersonEsRepository extends ElasticsearchRepository<Person,Long> {
    List<Person> findPersonByName(String name);
}
```

## 1.3 文档实体类
```java
@Data
@Document(indexName = "person", type = "chinese")
public class Person implements Serializable{
    private static final long serialVersionUID = -6804453833406105286L;
    @Id
    private Long id;
    private String name;
    private Integer age;
    private String address;
}
```

## 1.4 增删改查
```java
@Service
public class EsStarterService {
    @Autowired
    private PersonEsRepository repository;
    /**
     * 增
     */
    public Person save(Person person) {
        return repository.save(person);
    }
    /**
     * 删
     */
    public void delete(Person person) {
        repository.delete(person);
    }
    /**
     * 改
     */
    public Person update(Person person) {
        return repository.save(person);
    }
    /**
     * 查
     */
    public Iterable<Person> findAll() {
        return repository.findAll();
    }
}
```

## 1.5 单元测试
```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class EsStarterServiceTest {
    @Autowired
    private EsStarterService esStarterService;
    @Test
    public void save() {
        Person person = new Person();
        person.setId(new Random().nextLong());
        person.setName("lujiahao");
        esStarterService.save(person);
    }
    @Test
    public void delete() {
        Person person = new Person();
        person.setId(-5264182431891613084L);
        person.setName("lujiahao123456");
        esStarterService.delete(person);
    }
    @Test
    public void update() {
        Person person = new Person();
        person.setId(542136934419565287L);
        person.setName("lujiahao123456");
        esStarterService.update(person);
    }
    @Test
    public void findAll() {
        Iterable<Person> all = esStarterService.findAll();
        all.forEach(System.out::println);
    }
}
```

# 2. ElasticsearchTemplate
## 2.1 pom.xml和application.ym
```java
<!--elasticsearch-->
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-elasticsearch</artifactId>
</dependency>
spring:
  data:
    elasticsearch:
      repositories:
        enabled: true
      cluster-name: docker-cluster
      cluster-nodes: lujiahao.ml:9300
```

## 2.2 文档实体类
同上

## 2.3 增删改查
```java
@Service
public class ElasticsearchTemplateService {

    @Autowired
    public ElasticsearchTemplate elasticsearchTemplate;


    private static final String INDEX_NAME = "person";
    private static final String TYPE_NAME = "chinese";
    /**
     * 增
     */
    public String save(Person person) {
        IndexQuery indexQuery = new IndexQueryBuilder()
                .withIndexName(INDEX_NAME)
                .withType(TYPE_NAME)
                .withId(String.valueOf(person.getId()))
                .withObject(person)
                .build();
        String index = elasticsearchTemplate.index(indexQuery);
        System.out.println("xxxxxxxxxxxx " + index);
        return index;
    }

    /**
     * 删
     */
    public void deleteByName(String name) {
        DeleteQuery deleteQuery = new DeleteQuery();
        deleteQuery.setQuery(QueryBuilders.matchQuery("name", name));
        deleteQuery.setIndex(INDEX_NAME);
        deleteQuery.setType(TYPE_NAME);
        elasticsearchTemplate.delete(deleteQuery);
    }

    /**
     * 改
     */
    public UpdateResponse update(Person person) {
        try {
            UpdateRequest updateRequest = new UpdateRequest()
                    .index(INDEX_NAME)
                    .type(TYPE_NAME)
                    .id(String.valueOf(person.getId()))
                    .doc(XContentFactory.jsonBuilder()
                            .startObject()
                            .field("name", person.getName())
                            .endObject());
            UpdateQuery updateQuery = new UpdateQueryBuilder()
                    .withIndexName(INDEX_NAME)
                    .withType(TYPE_NAME)
                    .withId(String.valueOf(person.getId()))
                    .withClass(Person.class)
                    .withUpdateRequest(updateRequest)
                    .build();
            UpdateResponse update = elasticsearchTemplate.update(updateQuery);
            return update;
        } catch (Exception e) {
            return null;
        }
    }

    /**
     * 查
     */
    public List<Person> getAll() {
        SearchQuery searchQuery = new NativeSearchQueryBuilder()
                .withQuery(QueryBuilders.matchAllQuery())
                .build();
        return elasticsearchTemplate.queryForList(searchQuery, Person.class);
    }
}
```

## 2.4 单元测试
```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class ElasticsearchTemplateServiceTest {
    @Autowired
    private ElasticsearchTemplateService elasticsearchTemplateService;
    @Test
    public void save() {
        Person person = new Person();
        person.setId(new Random().nextLong());
        person.setName("haha");
        String save = elasticsearchTemplateService.save(person);
        System.out.println(save);
    }
    @Test
    public void deleteByName() {
        elasticsearchTemplateService.deleteByName("lujiahao");
    }
    @Test
    public void update() {
        Person person = new Person();
        person.setId(-5264182431891613084L);
        person.setName("hahaaaaaaaaa");
        UpdateResponse update = elasticsearchTemplateService.update(person);
        System.out.println(update);
    }
    @Test
    public void getAll() {
        List<Person> all = elasticsearchTemplateService.getAll();
        System.out.println(all);
    }
}
```

# 3. 代码示例
```java
https://github.com/lujiahao0708/LearnSeries/tree/master/LearnElasticSerach
```


## Tips
本文同步发表在公众号，欢迎大家关注！😁 
后续笔记欢迎关注获取第一时间更新！
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/%E5%85%AC%E4%BC%97%E5%8F%B7%E4%BA%8C%E7%BB%B4%E7%A0%81.jpg)