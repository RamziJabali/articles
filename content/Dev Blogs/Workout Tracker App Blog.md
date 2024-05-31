## 08/30/21

### Technologies Used
- Room Database
- RxJava
- Caldroid
- Koin Dependency Injection

### Project Overview
This project was undertaken to learn database management for a bigger project I am planning.

### Project Goals
- Allow users to open the app and see dates they have or haven't worked out.
- Enable users to click on a date to mark their workout status.
- Use a database to record the dates a user has worked out.

### Project Approach
- Divide the project into smaller pieces.
- Use existing tools and libraries to enhance the project.
- Decide on a database and stick with it.
- Stay flexible with ideas and technologies.

### Development Process

#### Calendar View Challenges
I started with the default `CalendarView` but struggled to implement the main features. I spent an unreasonable amount of time trying to add colors to specific dates, which proved fruitless.

**Learning Moment:** After three hours of frustration, I decided to abandon `CalendarView` and search for a better alternative.

#### Finding Caldroid
I found Caldroid on GitHub, which perfectly fit my requirements. Implementing Caldroid, I added click listeners to mark workout status on selected dates.

This was my first time working with Dates in a project, so I had to learn how to convert to and from the `Long` data type.

#### MVVM Architecture
I used the MVVM architecture for better readability and easier code management.

#### Database Management with Room
I decided to use Room for database management due to its simplicity and required features.

##### Steps:
1. **Creating an Entity:** I created an entity called `user_workout_schedule` with fields `date` (Long) and `did_user_attend_date` (Boolean).
2. **Room Database and DAO:** I created a Room database and a DAO. The DAO creation took some time, but Google searches and articles helped me get through it.
3. **Repository:** I created a repository for the database.

![Database](https://raw.githubusercontent.com/RamziJabali/blog/main/screenshots/DataBase.png)


#### Connecting the Pieces
With the calendar widget sorted and the database in place, I connected all the components and added final touches.

#### Use Cases
I added a UseCase to facilitate smooth conversion between the entity schema and the app schema. This step was crucial for abstraction and code maintainability.

#### Final Touches
After a few tweaks and a solid code review, the application was complete.

### Takeaways
- Do NOT get too attached to an idea.
- Be open to swapping out technologies for better ones.
- Break your project into manageable pieces for better progress and breathing room.




