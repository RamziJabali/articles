---
title: "First Android App: Tic-Tac-Toe"
date: 2021-07-21
tags:
  - android
---
### Introduction
In an attempt to learn Android development, I decided to create a simple tic-tac-toe game.

### First Realization
I couldn't just open my computer and start coding the application, like I would usually be able to when coding a terminal-based application.

### The UI Question
With that realization came a question: 

**"How do you want the UI to look like?"**

I wanted to have a simple grid that the user could click on to mark the board with their player.

### The Implementation Question
Upon answering that question, another came up in protest:

**"How will you implement the UI?"**

With my basic knowledge of Android, I listed all the different views I knew:
- TextView
- ButtonView
- EditTextView
- ImageView
- GridView

### View Comparison Table
| View         | Pros                               | Cons                                            |
|--------------|------------------------------------|-------------------------------------------------|
| TextView     | Easy to implement                  | Hard to customize                               |
|              |                                    | Rows and Columns are not programmatically updated|
|              |                                    | Looks ugly                                      |
| ButtonView   | Easy to implement                  | Hard to customize                               |
|              |                                    | Rows and Columns are not programmatically updated|
|              |                                    | Extremely ugly                                  |
| EditTextView | Easy to implement                  | Unneeded feature                                |
|              |                                    | Hard to customize                               |
|              |                                    | Rows and Columns are not programmatically updated|
|              |                                    | Will look extremely ugly                        |
| ImageView    | Looks good                         | Hard to make look good                          |
|              |                                    | Hard to customize                               |
|              |                                    | Rows and Columns are not programmatically updated|
| GridView     | Looks good                         | Hard to make                                    |
|              | Rows/columns programmatically set  |                                                 |
|              | Helps implement UI in a neater fashion |                                                 |
|              | Easy to maintain                   |                                                 |

### Setting Priorities
My priorities for the application:
- UI
- Programmatically set/updated attributes

With these priorities in mind, the clear winner was the GridView.

### Implementation Strategy
I decided to create this application using the GridView and the MVVM architecture.

### First Struggle
I didn't know how to create a GridView, so as any person would do, I went to YouTube and StackOverflow for answers.

I was bombarded with the same bad example over and over again and left with a bit of working code and a bit of knowledge of what I was doing.

### First Success
My first success came when I used GitHub's search feature to narrow down on similar code.

### Second Success
My second success was from reading articles about GridView, which shed light on what I had to do.

I was then able to construct a working piece of code that allowed me to see what I had done.

### Learning and Searching Techniques
Combining the learning and searching techniques I found, I was able to maximize my outcome and productivity.

### Unit Testing
Towards the end of the application, I wanted to add unit testing, so I went with JUnit for testing, but I realized it was not going to be as straightforward as unit testing in a terminal-based application.

With that in mind, I had to find a library that would allow me to test my code in isolation without having to make a copy of the code to test separately.

### Discovering Mockk.io
After googling for a few minutes, I saw Mockk.io pop up as a great library to use for testing code.

I tinkered with it for an hour or more in an attempt to understand how to use it, but that ended up being a hassle. All I had to do was open their documentation and use their listed example code.

### Final Polishing and Deployment
After unit testing was added, the application was working, and all it needed was a bit of UI and code polishing in highlighted areas.

Once I finished polishing what needed polishing, I shipped it to GitHub, marking the end of the program.

### Lessons Learned
- Search GitHub for example code
- Read articles pertaining to the topic at hand
- Read the documentation of a library you’re going to use
- Spend contiguous hours working on a project to maximize output
- Just because it works doesn’t mean it’s finished
