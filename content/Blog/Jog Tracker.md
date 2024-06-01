---
title: Jog Tracker
date: 2022-01-20
tags:
  - android
---
### Introduction
This blog documents my journey in creating a jog tracker Android application, combining various concepts and libraries I've learned.

### Libraries Used
- Caldroid
- Google Map
- Gson
- Koin
- LiveData
- MPAndroidChart
- OkHttp
- Retrofit
- WorkManager

### APIs Used
- Quotable

### Design Phase
Before diving into coding, I started by designing the application's look and flow. Inspired by existing jogging applications, I initially planned to include all statistics on one tab and a Google Maps view on another tab for specific runs. Realizing this could overcrowd the interface, I opted for a simpler approach, focusing on essential statistics.

- The first look:

![Original Imagination](https://raw.githubusercontent.com/RamziJabali/blog/main/screenshots/original_imagination.png)


#### Redesigned Statistics Page
I streamlined the statistics page to display only essential information, improving user experience and interface clarity.

![First Look](https://raw.githubusercontent.com/RamziJabali/blog/main/screenshots/first%20look.png)


### Calendar Integration
The calendar was designated its own page for users to view their runs and routes, with minor formatting adjustments for enhanced readability.

![Calendar Look](https://raw.githubusercontent.com/RamziJabali/blog/main/screenshots/calendar_look.png)


### Project Development
After extensive design work, I transitioned to coding, dividing the project into key components:

1. **Database Design:** Ensured a one-to-many relationship for jog entries to optimize querying.
2. **Setting up Database:** Established database setup.
3. **Calendar Page:** Implemented the calendar page for visualizing runs and routes.
4. **User Location:** Incorporated user location functionality, including permissions and foreground service.
5. **Motivational Quotes:** Integrated motivational quotes feature.
6. **Worker Manager:** Utilized WorkManager for background tasks.

### Database Design
To efficiently manage jog entries, I designed the database with a one-to-many relationship, summarizing multiple jog entries into a single record for streamlined querying.

![Database](https://raw.githubusercontent.com/RamziJabali/blog/main/screenshots/DataBase.png)

### Conclusion
Through meticulous design and development, the jog tracker application blends functionality and simplicity, providing users with essential statistics and a user-friendly interface for tracking their runs.
