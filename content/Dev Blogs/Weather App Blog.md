## 07/09/21

### Project Goal
In this project, my goal was to make API calls to a weather server and populate the UI with several weather attributes.

### Getting Started
I started this program by first adding the dependencies required for OkHttp and Retrofit.

### Learning Resources
I searched up a YouTube tutorial that would teach me the basics of Retrofit and OkHttp.

While learning the basics, I wanted to apply it to the project I wanted to make, so I looked for a weather API to use in my program.

### Key Steps
- Learn Retrofit
- Learn OkHttp
- Get an API to apply what I learned

### Initial API Call
Once I had written the API endpoint managers, I tried right away to make the API call. Once I ran the application, it crashed.

### Thread Management Discovery
After a couple of Google searches, I realized that I was making the call on the main thread, which was the reason for the application crashing.

I started to get into the discussion of thread management. I didn’t know what a thread in this context even really was, so I decided to Google it. I was bombarded with 100 different answers. For sanity's sake, I boiled it down to the ability to run multiple sequences of code at the same time using different threads (single sequential flow of control within a program).

### Adjusting Code for Background Thread
After that discovery, I had to make an adjustment to the code to allow it to run on the background thread as opposed to the main thread. Running the API call on the main thread causes the application to crash because the UI also runs on it. The reason for the application crashing is that the API call might have to wait for a response, which causes the UI to also wait, which in turn causes it to freeze and crash.

### Solution with Enqueue
The solution to that was using `enqueue`, which asynchronously sends a request and gets a callback of the result, executed on the background thread.

Perfect, the application was up and running!

### Final Steps
After that, I changed the architecture to MVVM, added test cases, and added GitHub actions. Finally, the application was a wrap!

### Lessons Learned
- Do NOT be intimidated by what you don’t know
- Do NOT worry about all the little details
- Get the application up and running, then worry about making it better
