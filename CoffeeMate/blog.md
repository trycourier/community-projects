# Coffee Mate

## Background

CoffeeMate helps users start their day reading about latest developments in areas of their interest without getting distracted by social media or drowning themselves in a sea of emails, notifications or even endless scrolling on a news app. CoffeeMate does this by sending registered users an email every morning with news articles from categories that they selected.

We used are HTML, CSS and Vanilla JS for the frontend and Node.js and Express.js for the backend of our website. 

## Instructions

### Part 1: [Replace with Subtitle]

[What are you building in this project]

1. [describe step]
2. [describe step]
   ```javascript
   //code
   ```
3. [describe step]

### Part 2: Frontend

To create the frontend of this project you need to create a ```public``` directory which will contain all the code to create the user interface.

1. Inside the ```public``` directory we create an [assets](https://github.com/kirtan-desai/coffee-mate/tree/main/public/assets) folder that contains the images our website displays. We have two background images -- one for the desktop web screen and the other for the mobile web screen -- and one logo image in this folder.
2. Now we create our [index.html](https://github.com/kirtan-desai/coffee-mate/blob/main/public/index.html) file inside the ```public``` directory. 

   In our ```<head>``` tag we have our title - CoffeeMate, our [fontawesome](https://fontawesome.com/) import for the social media logos in the footer, our [Google fonts](https://developers.google.com/fonts) import and our css stylesheet (which we will create in the next step) imports.

   Now our ```<body>``` consists of :
      - The header which consists of the title and the logo.
      - The content of the website i.e. the heading and the form. 
         
         Our form contains two input fields to take the user's name and email, checkboxes that consist of the different news categories the users can subscribe to and the sign up button that allows the user to submit the form and register for the daily news notifications.

      - The footer

   Our index.html file also contains code to handle our form data. In the ```<script>``` tag we create two functions. 
   
   Our first function handleData, makes sure that if the user tries to sign up without selecting a news category the form won't submit and the user's screen will display an error asking them to select atlease one category. If the user tries to sign up with all fields filled appropriately, we submit our form by making a POST request to the url "/" (by giving the method attribute and the action attribute in the form tag the value "POST" and "/" respectively), reset our form to clear all the input fields and display a success message on the screen to inform the user the form has been submitted successfully. 

   ```javascript
       function handleData(event){
        const form_data = new FormData(document.querySelector("form"));
        const form = document.getElementById("myForm")
        if(!form_data.has("checkbox-categories[]")){
            document.getElementById("error").style.visibility = "visible"
            return false
        }
        else {
            form.submit()
            form.reset()
            document.getElementById("submitted").style.visibility = "visible"
            return false
        }
        
      }
   ```

   Our second function checkError makes sure that if the user selects a category checkbox, the error message (if present, because the user tried to sign up without selecting a category) should disappear. We do this by setting an onclick listener on every checkbox. When a user clicks on a checkbox the checkError function is invoked for that instance of the checkbox. If the checkbox is checked, the function ensures that the error message is not displayed.

   ```javascript
      function checkError(box) {
        if (box.checked == true){
            document.getElementById("error").style.visibility = "hidden"
        }
    }
   ```


3. Lastly, we create two css files [index.css](https://github.com/kirtan-desai/coffee-mate/blob/main/public/index.css) and [reset.css](https://github.com/kirtan-desai/coffee-mate/blob/main/public/reset.css). 

   We use the reset.css to prevent the default styling of different broswers to affect the styling of our web page. Make sure to import this stylesheet prior to the index.css stylesheet in the index.html file.

## Conclusions

[Overview of the project. What's next?]

[Call to action: e.g. try building this project and tag me @shreythecray when you do!]

## About the Author

[Introduce yourself, talk about your interests, types of projects you like building, skills, etc.]

## Quick Links

🔗 [link all resources you use to build this project]
🔗 [e.g. documentation, stackoverflow pages, youtube videos, etc.]