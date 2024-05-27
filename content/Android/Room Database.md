## Creating Room Database

### Where to start?

First you need to add the required dependencies!

```
val room_version = "2.6.1"

    implementation("androidx.room:room-runtime:$room_version")
    annotationProcessor("androidx.room:room-compiler:$room_version")

    // Kotlin Extensions and Coroutines support for Room
    implementation("androidx.room:room-ktx:$room_version")

    // To use Kotlin Symbol Processing (KSP)
    ksp("androidx.room:room-compiler:$room_version")

```

There are three components to creating a Room database

1. Database class: holds the database and contains refrences of all DAO's
2. DAO(Data Access Object): Get's entites from the database and pushes changes back to the database
3. Entites: Get/Set field values

### Now that we have the basic understanding down, let's get started:

1. Create an entity for your table:
    * You have to annotate your data class with the `@Entity or @Entity("jog_entries")`
        * You are defining this data class as an entity for your DAO, and you can decide to label your table
        * You have to define a primary key for your entity via the `@PrimaryKey()` annotation, you can also have it so the keys are autogenrated.
        * You can optionally have different names for your columns by using `@ColumnInfo(name = "id")` which sets a custom names for a table and it's columns
           
```kotlin
@Entity("jog_entries")
data class JogEntry(
    @PrimaryKey(autoGenerate = true) @ColumnInfo(name = "id")
    val id: Int,
    @ColumnInfo("jog_id")
    val jogId: Int,
    @ColumnInfo("date_time")
    val dateTime: String,
    @ColumnInfo("latitude")
    val latitude: Double,
    @ColumnInfo("longitude")
    val longitude: Double,
)
```

2. Create a DAO for your table

    * The DAO(data access object) provides methods that the rest of your application uses to interact with your table.
        * So in this case my `JogEntryDAO` has multiple methods that will be used to interact with my `jog_entries` table previously defined.
     
```kotlin
@Dao
interface JogEntryDAO {
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    fun addUpdateWorkout(jogEntries: JogEntry)

    @Query("SELECT * FROM jog_entries")
    fun getAll(): Flow<List<JogEntry>>

    @Query("SELECT * FROM jog_entries WHERE date_time LIKE (:stringDate)")
    fun getByDate(stringDate: String): Flow<List<JogEntry>?>

    @Query("SELECT * FROM jog_entries WHERE date_time BETWEEN (:startDate) AND (:endDate)")
    fun getByRangeOfDates(startDate: String, endDate: String): Flow<List<JogEntry>>

    @Query("SELECT * FROM jog_entries WHERE jog_summary_id IS :jogId")
    fun getByID(jogId: Int): Flow<List<JogEntry?>>

    @Query("DELETE FROM jog_entries")
    fun deleteAll()
}
```

* You could have a usecase class that transforms values returned from your DAO and can transorm values required for your DAO.

3. Create your Database

    1. Class creation
        * Create an abstract class that extends `RoomDatabase`, ex: `abstract class AppDatabase : RoomDatabase()`
        * Annotate your class with the `@Database()` annotation and provide it with all your entities and a * version number: `@Database(entities = [JogEntryDAO::class], version = 1)`
    
    2. Within the class
        * Define abstract functions for all your available DAO's: `abstract fun jogEntryDao(): JogEntryDAO`
    
    3. Refrence your database for usage
        * Use Rooms `databaseBuilder()` to create your database object
        * ```
          val db = Room.databaseBuilder(
            applicationContext,
            AppDatabase::class.java, "just-jog-database")
          .build()
          ```
        * I prefer to do this with dependency injection framework like Koin.

```kotlin
@Database(entities = [JogEntryDAO::class], version = 1)
abstract class AppDatabase : RoomDatabase() {
    abstract fun jogEntryDao(): JogEntryDAO
}
```

4. Use your database

```kotlin
val db = Room.databaseBuilder(
            applicationContext,
            AppDatabase::class.java, "just-jog-database")
          .build()

val jogEntryDao = db.jogEntryDao()
val users: List<User> = jogEntryDao.getAll()

```

5. Example:

   * Mock View Model to simulate an actual application
   * Pass it a dao refrence or a usecase refrence to be able to make calls to and from the database
   * Using a `CompletableDeferred` makes it so I know that the process completed successfully or failed exceptionally

```kotlin
     class MockVM(private val dao: JogEntryDAO): ViewModel {
   
     fun addJog(jogEntry: JogEntry) {
        val deferred = CompletableDeferred<Unit>()

        val job = viewModelScope.launch(Dispatchers.IO)  {
            try {
                dao.addUpdateWorkout(jogEntry)
                deferred.complete(Unit)
                Log.d("MockVM::Class.java", "added Jog: $deferred")
            } catch (e: CancellationException) {
                Log.d("MockVM::Class.java", "Coroutine canceled: ${e.message}")
                // Handle cancellation if needed
            }
        }

        deferred.invokeOnCompletion { cause ->
            if (cause != null) {
                if (cause is CancellationException) {
                    Log.d("MockVM::Class.java", "Coroutine canceled: ${cause.message}")
                } else {
                    Log.e("MockVM::Class.java", "Coroutine failed with: $cause")
                }
                job.cancel() // Cancel the job explicitly
                }
            }
         }

      fun getAllJogs() {
          viewModelScope.launch(Dispatchers.IO){ 
                val list = viewModelScope.async { dao.getAll() }.await()
                list.collect {
                    Log.d("MockVM::Class.java", "getAllJogs: $it")
                    }
                }
            }
        }
     }
   
```

* Within your `MainActivity` you can have a refrence to your `MockVM` and call the `getAllJogs()` function
   
```kotlin
override fun onCreate(){
vm.getAllJogs()
     override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        val db = Room.databaseBuilder(
            applicationContext,
            JustJogDataBase::class.java, "just-jog-database"
        )
            .build()
        val vm = MockVM(db.jogEntryDao())
        vm.addJog(
            JogEntry(
                id = 0, jogSummaryId = 0, dateTime = "", latitude = 0.0, longitude = 0.0
            )
        )

        vm.getAllJogs()
}
```
  

### Result:

LogCat
```
2024-05-13 13:42:03.554 17739-17763 MockVM::Class.java      ramzi.eljabali.justjog               D  addJog: true
2024-05-13 13:42:03.850 17739-17763 MockVM::Class.java      ramzi.eljabali.justjog               D  getAllJogs: [JogEntry(id=1, jogSummaryId=0, dateTime=, latitude=0.0, longitude=0.0)]
```
