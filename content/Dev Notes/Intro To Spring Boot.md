---
title: Intro To Spring Boot
author: Ramzi Eljabali
date: 2025-06-14
tags:
  - article
  - spring-boot
  - kotlin
  - api
  - incomplete
---
# Intro To Spring Boot

## Introduction

I want to create an endpoint to get random quotes or quotes depending on specific tags. With my limited knowledge I know of Spring Boot. So let's get into how we create a spring boot project and then how we create an API.

## Key Points
- **Point 1:** Create Spring Boot Project 
- **Point 2:** Create Quotes Database
- **Point 3:** Testing End Point
- **Point 4:** Unit Testing Code
- **Point 5:** Adding Data

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
## Section 2: Creating the quotes database

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

![post_man_200_request_for_get_quotes_by_tag](https://raw.githubusercontent.com/RamziJabali/articles/refs/heads/v4/images/post_man_get_request.png)

- `{{base_url}}/quotes`

![post_man_200_request_for_posting_quote](https://raw.githubusercontent.com/RamziJabali/articles/refs/heads/v4/images/post_man_post_request.png)


## 4. Unit Testing Code

| Annotation        | Purpose                             | Usage Example               |
| ----------------- | ----------------------------------- | --------------------------- |
| `@SpringBootTest` | Full app context (integration test) | Service test with real repo |
| `@MockBean`       | Replace bean with mock              | Mock repo in service test   |
| `@WebMvcTest`     | Test controller layer only          | Controller unit tests       |
| `@Autowired`      | Inject beans                        | Inject service in test      |
| `@Test`           | Mark method as test                 | Any test method             |

### Service Testing

```kotlin
@Autowired  
lateinit var quoteService: QuoteService  
  
@MockitoBean  
lateinit var quoteRepository: QuoteRepository


@Test  
fun `getQuotesByTag returns a quote with that tag`() {  
    val strongTag = Tag(id = 1L, name = "strong")  
    val braveTag = Tag(id = 2L, name = "brave")  
    val motivationalTag = Tag(id = 2L, name = "motivational")  
  
    val expectedQuotes = listOf(  
        Quote(id = 2L, text = "Be brave!", author = "Someone", tags = setOf(braveTag)),  
        Quote(  
            id = 2L,  
            text = "Be really brave and inspiring!",  
            author = "Someone",  
            tags = setOf(braveTag, motivationalTag)  
        )  
    )  
    // Mock the repository to return test data  
    given(quoteRepository.count()).willReturn(4L)  
    given(quoteRepository.findByTags_Name(braveTag.name))  
        .willReturn(expectedQuotes)  
  
    // Call the service  
    val result = quoteService.getQuotesByTag(braveTag.name)  
  
    assertEquals(expectedQuotes.size, result.size)  
    assertEquals(expectedQuotes.map { it.toQuoteResponse() }, result)  
}
```

### Controller Testing

```kotlin
  
@Autowired  
lateinit var mockMvc: MockMvc  
  
@MockitoBean  
lateinit var quoteService: QuoteService  
  
private val objectMapper = jacksonObjectMapper()

  
@Test  
fun `POST addQuote returns created quote`() {  
    val quoteRequest = QuoteRequest(  
        text = "Be fearless",  
        author = "John Doe",  
        tags = listOf("motivational", "brave")  
    )  
  
    val quoteResponse = QuoteResponse(  
        id = 1L,  
        text = "Be fearless",  
        author = "John Doe",  
        tags = listOf("motivational", "brave")  
    )  
  
    // Mock the service call  
    given(quoteService.createQuote(quoteRequest)).willReturn(quoteResponse)  
  
    mockMvc.perform(  
        post("/quotes")  
            .contentType(MediaType.APPLICATION_JSON)  
            .content(objectMapper.writeValueAsString(quoteRequest))  
    )  
        .andExpect(status().isOk)  
        .andExpect(jsonPath("$.id").value(1L))  
        .andExpect(jsonPath("$.text").value("Be fearless"))  
        .andExpect(jsonPath("$.author").value("John Doe"))  
        .andExpect(jsonPath("$.tags", Matchers.hasSize<Any>(2)))  
        .andExpect(jsonPath("$.tags", Matchers.containsInAnyOrder("motivational", "brave")))  
}
```

## Conclusion

Using `Spring Boot` and `PostgreSQL` I was able to pretty easily able to make an endpoint to be able to post quotes and tags to our database.