---
title: User Info CSV
date: 2024-05-31
tags:
  - kotlin
---
In the Kotlin file, write a program to perform a GET request on the route https://coderbyte.com/api/challenges/logs/user-info-csv and then sort the CSV data by the second column.

Finally, print it as comma-separated values.

Example Input:
```
name,email,phone
Jane Smith,janesmith@example.com,555-5678
John Doe,johndoe@example.com,555-1234
```


Example Output: 
```
Jane Smith, janesmith@example.com, 555-5678, John Doe, johndoe@example.com, 555-1234
```

```kotlin
import java.io.BufferedReader
import java.io.InputStreamReader
import java.net.HttpURLConnection
import java.net.URL

fun main() {
  val targetURL = "https://coderbyte.com/api/challenges/logs/user-info-csv"
  val urlConnection = URL(targetURL).openConnection() as HttpURLConnection
  val responseCode = urlConnection.responseCode
  
  if (responseCode != HttpURLConnection.HTTP_OK) {
      println("Failed Request $responseCode")
      return
  } 
  val reader = BufferedReader(InputStreamReader(urlConnection.inputStream))
  var emailHashMap = HashMap<String, String>()    
  
  // skipping headers -> "name,email,phone"
  reader.readLine()
  var line = reader.readLine()

  do {
    val array = line.split(",")
    emailHashMap[array[1]] = line.replace(",", ", ")
    line = reader.readLine()
  } while(line != null)

  reader.close()

  // Sorting 
  var csvSorted = ""
  emailHashMap.keys.sorted().forEachIndexed { index: Int, key: String ->
    if (index == emailHashMap.size - 1) {
      csvSorted += emailHashMap[key]
    } else {
      csvSorted += emailHashMap[key]+", "
    }
  }
  print(csvSorted)
}
```