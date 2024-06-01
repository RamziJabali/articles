---
title: Intro to Android
date: 2021-03-01
tags:
  - java
  - android
  - xml
  - article
---
<p align="center">
 
<h2 align="center">onClick</h2>

We made a button in XML and defined it's onClick property which when we go to Java will allows us to make it 

into a function that when the button is clicked the function clickFunction is then called upon.

```java
 public void clickFunction(View view){
 
	CODE

 }

```

We had to import the View.

```java
import android.view.View;
```

<h4 align="center">Button</h4>

```xml
    <Button
        android:id="@+id/button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:onClick="clickFunction" // We now defined an onClick function called clickFunction
        android:text="button"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />
```

![1](https://github.com/RamziJabali/Learning_Android/blob/main/screenshots/Screen%20Shot%202021-04-07%20at%2012.23.56%20AM.png)

we're putting a line of code into Log, the log tells us what is going on behind the scenes when the app is running.

So we're putting a type of log called "i" for info, which tells you certaion information about your application.

We add a tag that describes the log, and we can display that message in the Logcat.

<h4 align="center">Log</h4>

```java
public void clickFunction(View view){
    Log.i("info", "Button is pressed")
}
```

OUTPUT:
![2](https://github.com/RamziJabali/Learning_Android/blob/main/screenshots/Screen%20Shot%202021-03-06%20at%2010.44.28%20PM.png)


<h4 align="center">EditText</h4>

```xml
<EditText
        android:id="@+id/nameEditText" //Edit text ID
        android:layout_width="406dp"
        android:layout_height="67dp"
        android:layout_marginBottom="51dp"
        android:ems="10"
        android:hint="Your Name?"
        android:inputType="textPersonName"
        android:text="Name"
        app:layout_constraintBottom_toTopOf="@+id/button"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintHorizontal_bias="0.4"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintVertical_bias="1.0" />
```

![3](https://github.com/RamziJabali/Learning_Android/blob/main/screenshots/Screen%20Shot%202021-04-07%20at%2012.24.07%20AM.png)

To see what the user enters in the EditText box the log when the button is pressed we need to include it in the onCLick function we made

We first need to make a EditText variable which has a type of `EditText` we will name it `nameEditText` and we'll set it equal to 

the EditText box view and we'll need to use `findViewById()` to find that view. We need to use `R` to be able to acess resources

and then we specify `id` and then the name of the EditText box id we made which is `nameEditText`.

```java
EditText nameEditText = (EditText) findViewById(R.id.nameEditText);//it will return a view so we want to convert the View into an 
                                                                     EditText
```

```java
 public void clickFunction(View view){
        EditText nameEditText = (EditText) findViewById(R.id.nameEditText);
        Log.i("info", "Button is pressed");
        Log.i("Values" , nameEditText.getText().toString()); //gets the value of the variable nameEditText, then we convert it to a string!

    }
```
![4](https://github.com/RamziJabali/Learning_Android/blob/main/screenshots/Screen%20Shot%202021-04-07%20at%2012.38.36%20AM.png)
![5](https://github.com/RamziJabali/Learning_Android/blob/main/screenshots/Screen%20Shot%202021-04-07%20at%2012.06.06%20AM.png)

<h2 align="center">Example</h2>

<h4 align="center">XML</h4>


```xml
<EditText
        android:id="@+id/editTextTextUserName"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:ems="10"
        android:inputType="textPersonName"
        android:hint="Enter Username"
        app:layout_constraintBottom_toTopOf="@+id/editTextPassword"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <EditText
        android:id="@+id/editTextPassword"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:ems="10"
        android:hint="Enter Password"
        android:inputType="textPassword"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintHorizontal_bias="0.502"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <Button
        android:id="@+id/button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Button"
        android:onClick="loginButtonOnClick"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/editTextPassword" />
```

<h4 align="center">Java</h4>

```java
    public void loginButtonOnClick(View view){
        EditText userNameEditText = (EditText)(findViewById(R.id.editTextTextUserName));
        EditText passwordEditText = (EditText)(findViewById(R.id.editTextPassword));

        Log.i("info", "Button Clicked");

        Log.i("UserName", userNameEditText.getText().toString());
        Log.i("Password", passwordEditText.getText().toString());
    }
```
<h4 align="center">User Interface</h4>

![6](https://github.com/RamziJabali/Learning_Android/blob/main/screenshots/Screen%20Shot%202021-04-07%20at%201.41.29%20PM.png)

<h4 align="center">Log Cat</h4>

![7](https://github.com/RamziJabali/Learning_Android/blob/main/screenshots/Screen%20Shot%202021-04-07%20at%201.41.51%20PM.png)


<h4 align="center">Toast</h4>


To use the Toast features you have to import the Toast class first
```java
import android.widget.Toast;
```
You can display a pop up using toast by using the following
```java
Toast.makeText(<context>,<display message>, <Toast.LENGTH_SHORT>).show();
```
Displays the text typed in the ```EditText``` box.
```java
Toast.makeText(this,"Hello "+userNameEditText.getText().toString(), Toast.LENGTH_SHORT).show();
```

![8](https://github.com/RamziJabali/Learning_Android/blob/main/screenshots/Screen%20Shot%202021-04-14%20at%2012.38.06%20PM.png)

<h4 align="center">ImageView</h4>

```xml
 <ImageView
        android:id="@+id/imageView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:scaleType="fitCenter"
        android:src="@drawable/aot"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintVertical_bias="0.551" />
```

![9](https://github.com/RamziJabali/Learning_Android/blob/main/screenshots/ImageView.png)

We can use the onClick function from the button to change the image of the ImageView.

We use `setImageResource(R.id.<filename>);` to change the image of the `ImageView`
Example
```java
public void buttonOnClick(View view){
        ImageView image = (ImageView) (findViewById(R.id.imageView));
        image.setImageResource(R.drawable.eren);
    }
```
<h2 align="center">Example of changing an image onClick</h2>

![10](https://github.com/RamziJabali/Learning_Android/blob/main/screenshots/ImageChange.gif)

<h1 align="center">Currency Changing Application</h1>

<h2 align="center">XML</h2>

```xml
<ImageView
        android:id="@+id/imageViewMoney"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:src="@drawable/pounds"
        app:layout_constraintBottom_toTopOf="@id/textViewCurrency"
        app:layout_constraintHorizontal_bias="0.495"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <TextView
        android:id="@+id/textViewCurrency"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/ENTER_CURRENCY"
        app:layout_constraintBottom_toTopOf="@id/buttonToConvert"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/imageViewMoney" />

    <EditText
        android:id="@+id/userEditTextView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:hint="@string/ENTER_CURRENCY_VALUE"
        app:layout_constraintTop_toBottomOf="@+id/textViewCurrency"
        app:layout_constraintBottom_toTopOf="@id/buttonToConvert"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        />

    <Button
        android:id="@+id/buttonToConvert"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/CONVERT"
        android:onClick="convertToDollarsOnButtonClick"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toBottomOf="@id/textViewCurrency" />

```
<h2 align="center">UI</h2>

![11](https://github.com/RamziJabali/Learning_Android/blob/main/screenshots/Screen%20Shot%202021-04-16%20at%2011.03.14%20PM.png)

<h2 align="center">Java</h2>

This is the onClick function we covered

```java
 public void convertToDollarsOnButtonClick(View view) {
        EditText userText = (EditText) (findViewById(R.id.userEditTextView));
        ImageView imageView = (ImageView) (findViewById(R.id.imageViewMoney));

        Log.i("button clicked", "User Entry");

        Toast.makeText(this, userText.getText().toString() + "£ is " + convertPoundsToDollars(Double.parseDouble(userText.getText().toString())) + "$", Toast.LENGTH_LONG).show();

        imageView.setImageResource(R.drawable.money);

    }
```

The function that's doing the converting 

```java
private double convertPoundsToDollars(Double pounds) {
        double dollarInPounds = 1.38;
        return pounds * dollarInPounds;
    }
```

<h2 align="center">OUTPUT</h2>

![12](https://github.com/RamziJabali/Learning_Android/blob/main/screenshots/currencyConverter.gif)