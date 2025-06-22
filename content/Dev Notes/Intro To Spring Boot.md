---
title: Intro To Spring Boot
author: Ramzi Eljabali
date: 2025-06-14
tags:
  - article
  - spring-boot
  - kotlin
  - api
---

# Intro To Spring Boot

## Introduction

I want to create an endpoint to get random quotes or quotes depending on specific tags. With my limited knowledge I know of Spring Boot. So let's get into how we create a spring boot project and then how we create an API.

## Key Points
<!-- List the main points of the article. You can create bullet points or brief descriptions here. -->

- **Point 1:** Create Spring Boot Project 
- **Point 2:** Create Quotes Data Base
- **Point 3:**

## Section 1: Create Spring Boot Project 

Thankfully there is a GUI we can use to create the spring boot project. 
Using [Spring Initializer](https://start.spring.io) we can pick:
- `project`
	- `Gradle - Kotlin`
- `Spring Boot` - The version of it you want to use.
	- `3.5.0`
- `Project Metadata`
	- `Group`
	- `Artifact`
	- `Name`
	- `Description`
- `Packaging`
	- `Jar`
- `Java`
	- `17`
- `Dependencies`
	- Spring Web (for REST)
	- Spring Boot DevTools (for hot reload)
	- Spring Data JPA (for DB access, optional)
	- H2 or PostgreSQL (DB, optional)

![Spring Boot Initializer Configuration](Add)

Now that we have the configuration we click `generate` and we have our spring boot project.
## Section 2: Creating the quotes data base

Now that we have the project generated, we want to create the `PostgreSQL` database

- We're going to be using PostgreSQL
	1. We want to install `PostgreSQL`
		1. `brew install postgresql`
	2. `psql postgres`
		1. `CREATE USER quotesapp WITH PASSWORD 'secret123';`
		2. `CREATE DATABASE quotesdb OWNER quotesapp;`
		3. `GRANT ALL PRIVILEGES ON DATABASE quotesdb TO quotesapp;`			

### 1. **Creating a database user (`quotesapp`)**
- You create a database user **quotesapp** with a password.
- This user is like an account that can connect to PostgreSQL.
- Our Spring Boot app will use this user’s credentials to log in and interact with the database securely.

---

### 2. **Creating a database (`quotesdb`)**

- You create a database named **quotesdb** where our application’s data will be stored (tables, rows, etc.).
- This is the container for all our data.

---

### 3. **Granting privileges**

- You give the user **quotesapp** full rights on the **quotesdb** database.
- This means our app can create tables, insert, update, delete, and query data inside this database.

---
The SQL commands **prepare your database environment**, and then our Spring Boot app uses that environment to store and retrieve your data.

## Section 3: Creating  The Endpoint
### 1. Let's define our `Quote` & `Tag` `entity` in our project:

These entities are tables in our database.

```kotlinr
@Entity  
data class Quote(  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    val id: Long = 0,  
    val text: String,  
    val author: String,  
  
    @ManyToMany(cascade = [  
    CascadeType.PERSIST,  
    CascadeType.MERGE,  
    CascadeType.REFRESH,  
	])
    @JoinTable(  
        name = "quote_tags",  
        joinColumns = [JoinColumn(name = "quote_id")],  
        inverseJoinColumns = [JoinColumn(name = "tag_id")]  
    )  
    val tags: Set<Tag> = setOf()  
)

@Entity  
data class Tag(  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    val id: Long = 0,  
    val name: String  
)
```

---
### 2. Creating Repositories

We want to define our repository which we will use to expose to our service to make the endpoint work.

The **repository** is our app’s direct line to the **database**. It’s where we define methods to:
- Find, save, update, and delete records
- Query the database
- Handle relationships between entities

```kotlin
interface QuoteRepository : JpaRepository<Quote, Long> {  
    fun findByTags_Name(name: String): List<Quote>  
}  
  
interface TagRepository : JpaRepository<Tag, Long> {  
    fun findByName(name: String): Tag?  
}
```

### 3. Creating Our Service

The **service** is where we write our app’s **business logic** - what runs when someone asks for something:

```kotlin
@Service  
class QuoteService(  
    private val quoteRepository: QuoteRepository,  
    private val tagRepository: TagRepository  
) {  
    @Transactional  
    fun createQuote(request: QuoteRequest): Quote {  
        val tagEntities = request.tags.map { tagName ->  
            tagRepository.findByName(tagName) ?: Tag(name = tagName)  
        }.toSet()  
  
        val quote = Quote(  
            text = request.text,  
            author = request.author,  
            tags = tagEntities  
        )  
  
        return quoteRepository.save(quote)  
    }  
  
    @Transactional(readOnly = true)  
    fun getQuotesByTag(tagName: String): List<Quote>{  
        return quoteRepository.findByTags_Name(tagName)  
    }  
}
```

### 4. Creating Our Controller


```kotlin
@RestController  
@RequestMapping("/quotes")  
class QuotesController(private val quoteService: QuoteService) {  
    @PostMapping  
    fun addQuote(@RequestBody request: QuoteRequest) = quoteService.createQuote(request)  
  
    @GetMapping("/tag/{tagName}")  
    fun getQuotesTag(@PathVariable tagName: String): List<Quote> {  
        return quoteService.getQuotesByTag(tagName)  
    }  
}
```

| Layer          | Responsibility                               | Talks to      |
| -------------- | -------------------------------------------- | ------------- |
| **Controller** | Receives HTTP request                        | Users/Clients |
| **Service**    | Business logic: what to do with that request | Repositories  |
| **Repository** | Fetch/store data in the database             | PostgreSQL    |
## Section 3: Testing Our Endpoint

Using `Post Man` I am able to test our end point.

- `{{base_url}}/quotes/tag/{{tagName}}`
![]()

- `{{base_url}}/quotes`
![]()

## Conclusion
