---
title: Pokedex App
date: 2021-08-17
tags:
  - android
---
This project was a combination of everything that I have learned thus far, with extra specs.

### Technologies Used
- Retrofit
- OkHttp
- RxJava
- Koin Dependency Injection
- Picasso

### Project Overview
This project is a Pokedex application that shows a set number of Pokémon at first and allows you to click on a Pokémon of your choosing to learn more about that character.

The application is broken down into two main pieces:
1. **Pokedex:** A set of Pokémon.
2. **Pokemon:** A specified Pokémon from the Pokedex page.

It also allows you to scroll down until you reach the last Pokémon available in the PokeAPI.

### Steps Taken

#### 1. API Selection
The first step was to pick out an API that has adequate information about Pokémon. Lucky for me, there is a very well-documented Pokémon API, PokeAPI, made by Paul Hallett, which was perfect for this project.

#### 2. Dependency Injection
I started by creating Koin dependency injections for Retrofit, OkHttp, and GsonConverterFactory so that I could run a quick test to see a sample of the Pokedex API.

#### 3. Building the UI
After I got the network calls working, I started to build the first UI component: a screen full of Pokémon characters with their names and pictures. 

I decided to go with the GridView, which would allow me to show a good number of Pokémon on one page. Implementing and customizing the GridView was straightforward.

#### 4. Handling Images
I found a different API that contained only images of the Pokémon that corresponded with PokeAPI Pokémon ID numbers. I used Picasso by Square to seamlessly incorporate the Pokémon images into the Pokedex UI.

#### 5. Fragment vs Activity
I initially tried making the two UI elements into fragments, but since I had already made progress using Activities, I decided to recreate the fragment into an Activity to maintain the progress of the application.

#### 6. Dedicated Pokémon Page
Next, I implemented a dedicated Pokémon page with multiple stats. I needed to pass the Pokémon ID from the Pokedex page to the Pokémon page. After some research, I created a `newInstance` static function that takes in a Context and Pokémon ID integer, returning a new instance of the Pokémon Activity to launch it.

#### 7. UI Enhancements
I upgraded the Pokémon UI by adding horizontal progress bars corresponding to the Pokémon stats and fixing text locations. I also added animation to the launch and exit of the Pokémon Activity.

### Challenges and Solutions
- **API Issues:** The Pokémon image API was incomplete. I replaced it with a better one.
- **Endpoint Issues:** I was using the wrong endpoint for the Pokedex, causing excessive calls. A simple change fixed this.
- **Use Cases:** No use cases were created for the Pokedex and Pokémon pages initially. Creating use cases reduced the burden on the ViewModel and improved the code quality.

### Takeaways
- Get your app up and running, then start cleaning up and adding features.
- Don't be scared to delete your code in favor of better code.
- Don't get cold feet; tackle challenges head-on even if you're unsure of your capabilities.
