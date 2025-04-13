---
title: Automating Work Flows
author: Ramzi Eljabali
date: 2025-04-12
tags:
  - article
  - git-hook
  - automating
  - formatting
  - ktlint
  - android
  - git
---

# Automating Work Flows

## Introduction

When we're in the zone coding and we are consistently adding and testing, we do tend to forget to format our code. This now results in pushing up unformatted dirty code. In an effort to avoid this I want to automate this part of my work flow to allow peace of mind.

## Key Points

- **Part 1:** Adding Format Tool Gradle Task 
- **Part 2:** Creating Pre-commit Git Hook
- **Part 3:** Testing

## Section 1: Adding Format Tool Gradle Task 

1. I picked [Ktlint](https://github.com/pinterest/ktlint) as my linter for my project.
2. Added it to my `libs.toml`
3. `ktlint = { id = "org.jlleitschuh.gradle.ktlint", version.ref = "ktlint" }
4. Added it to my root `build.gradle.kts`
5. Tested that it was a task I could run by running the following command
	- `./gradlew ktlintTest`
	- Observe the outcome
```
./gradlew ktlintCheck
Calculating task graph as no cached configuration is available for tasks: ktlintCheck
Type-safe project accessors is an incubating feature.

> Configure project :shared
w: file:///Users/ramzijabali/Documents/Code/just-jog-kmm/shared/build.gradle.kts:12:13: 'kotlinOptions(KotlinJvmOptions.() -> Unit): Unit' is deprecated. Please migrate to the compilerOptions DSL. More details are here: https://kotl.in/u1r8ln

> Task :ktlintKotlinScriptCheck FAILED
/Users/ramzijabali/Documents/Code/just-jog-kmm/build.gradle.kts:2:5 Missing space after //
/Users/ramzijabali/Documents/Code/just-jog-kmm/settings.gradle.kts:1:1 File must end with a newline (\n

FAILURE: Build failed with an exception.
```
6. That's about it
## Section 2: Creating Pre-commit Git Hook
1. Create a hook script 
	- Create the file in the following file structure `.git/hooks/pre-commit`
		- `touch .git/hooks/pre-commit` / `nano .git/hooks/pre-commit`
		- `chmod +x .git/hooks/pre-commit`
2. Add the following script to the file
```
#!/bin/sh

echo "Running ktlintFormat..."
./gradlew ktlintFormat

# If it fails, prevent the commit
if [ $? -ne 0 ]; then
  echo "ktlintFormat failed. Commit aborted."
  exit 1
fi
```

- ###  `#!/bin/sh`
	This is the **shebang** line. It tells the system to use the `sh` shell (Bash-compatible) to run this script

- ### `echo "Running ktlintFormat..."`
	Prints a message to the console so you know what the script is doing

- ### `./gradlew ktlintFormat`
	This line actually **runs the formatting** using Gradle Wrapper

- ### `if [ $? -ne 0 ]; then`
	`$?` is a special variable in shell scripting: it holds the **exit code** of the last command
	`-ne` means "not equal"
	So this line checks: **Did `ktlintFormat` fail? (non-zero exit code)**

- ### `echo "ktlintFormat failed. Commit aborted."`
	If it failed, we print a message to let you know why the commit isn’t going through
	
- ### `exit 1` 
	This tells Git: “Do **not** allow the commit.”  
	Any non-zero exit code from a Git hook will stop the commit

If it failed, we print a message to let you know why the commit isn’t going through.
## Section 3: Testing

- Make a commit and check out the results!

## Conclusion

Automating workflow is a lot easier than you would think and should be added as a part of your project to make code production faster and easier.

