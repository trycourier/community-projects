# Build the CoffeeMate Website with Express.js and Courier for Email Notifications

## Background

CoffeeMate helps users start their day reading about latest developments in areas of their interest without getting distracted by social media or drowning themselves in a sea of emails, notifications or even endless scrolling on a news app. CoffeeMate does this by sending registered users an email every morning with news articles from categories that they selected.

We used HTML, CSS and Vanilla JS for the frontend and Node.js, Express.js and Firebase for the backend of our website. 

## Instructions

### Part 1: Backend

In this part, we will be setting up the backend of our app which involves fetching news data and sending news data through Courier.

We will be using ```node.js``` to run our javascript code for our backend. If you don't have node installed, install it from [here](https://nodejs.org/en/). You can download the current latest version.

1. Initializing project

   Make a project folder with name of your choice. 

   Open the folder in the code editor of your choice and using the terminal in the project directory run ```npm init --y```. 

   You should have a package.json file now.

   In the terminal, run the command ```git init``` to initiate git for the project. If you don't have ```git``` installed on your local machine, you download it from [here](https://git-scm.com/downloads)

2. Adding dependencies

   In you package.json file, add the following field which are the dependencies we'll be installing for our project

   ```json
   "dependencies": {
      "dotenv": "^16.0.2",
      "express": "^4.18.1",
      "express-validator": "^6.14.2",
      "firebase-admin": "^11.0.1",
      "node-fetch": "^2.6.7"
   }
   ```
   Make sure you're in the project folder in the terminal and run ```npm i``` to install the dependencies we just added.

   Once the command is finished running, we'll have a ```node_modules``` folder in our project directory which contains the code for all the packages we installed. We don't need to add it to git since we can always install these packages from our ```package.json``` file using ```npm i```

   To remove node_modules from git, make a ```.gitignore``` file in the project directory and write ```node_modules``` in it.


3. Fetching News

   Make a file named ```.env``` in your project folder and add it to ```gitignore```. 

   We will be storing our secret keys as variables in this file so they do not get added to git and remain private to us. These variables are called environment variables.

   We will be using the news API provided by https://newsapi.org/. Register an account on their website and you'll recieve an API key.

   Add this api key to your .env file

   ```
   NEWS_API_KEY="your-api-key-here"
   ```
   We will be creating a file called ```apiHandler.js``` to manage our functions interacting with APIs.

   In ```apiHandlers.js```, add these two lines of code

   ```js
   const fetch = require('node-fetch') // used to make HTTP requests to our news API
   require('dotenv').config() // enables us to use our private environment variables
   ```

   The news API fetches news based on the category we provide. It can only fetch news for one category at a time. 

   So if we want news for multiple categories, we have to make an API call for each category.

   We will be defining a ```fetchSingleCategoryNews``` function to fetch news for a single category.

   Add the following function in ```apiHandlers.js```

   ```js
   const fetchSingleCategoryNews = async () => {
      const news_api = `https://newsapi.org/v2/top-headlines?country=us&category=general&apiKey=${process.env.NEWS_API_KEY}`

      const res = await fetch(news_api)

      if (!res.ok) throw new Error("News API is not working")

      const json = await res.json()

      console.log(json)
   }
   ```

   Add a function call too in ```apiHandlers.js``` so we can check if our news API is working
   ```js
   fetchSingleCategoryNews()
   ``` 

   Make sure you're in the project directory in the terminal and run `node apiHandlers.js`

   You should see an object containing a list of news articles.

   Now, we need to fetch news for multiple categories and combine them together into a news object containing 5 news.

   Remove the function call `fetchSingleCategoryNews()` and instead add this function to ```apiHandlers.js``` after our ```fetchSingleCategoryNews``` function.

   ```js
   const fetchNews = async (categories = ['general', 'business']) => {
      let news = []
      let newsToSend = 5

      try {
         // fetch n number of news for each category and put it in a news list such that we have 5 news in total
         newsSent = 0
         for (i in categories) {
               newsSent = Math.floor(newsToSend / (categories.length - i))
               newsToSend = newsToSend - newsSent
               news = news.concat(await fetchSingleCategoryNews(categories[i], newsSent))
         }
      } catch (error) {
         console.log(error)
         throw new Error(error)
      }

      // transform the news list to an object which we will be using to fill data in our email   
      let news_obj = {}
      for (i in news) {
         news_obj[i] = {}
         news_obj[i]["title"] = news[i]["title"]
         news_obj[i]["description"] = news[i]["description"]
         news_obj[i]["link"] = news[i]["url"]
      }

      console.log(news_obj)
   }
   ```

   Now, we will change our ```fetchSingleCategoryNews``` function to take a category and num of news parameters and return news

   ```js
   const fetchSingleCategoryNews = async (category, num) => {
      const news_api = `https://newsapi.org/v2/top-headlines?country=us&category=${category}&apiKey=${process.env.NEWS_API_KEY}`

      const res = await fetch(news_api)

      if (!res.ok) throw new Error("News API is not working")

      const json = await res.json()
      return json.articles.slice(0, Math.min(json.articles.length, num))
   }
   ```

   Add a function call for ```fetchNews``` 
   ```js
   fetchNews()
   ```

   Run `node apiHandlers.js`

   You can see our news object of 5 news now in the terminal.

4. Sending News using Courier

   The first thing we need to do is setup a Courier account with a Gmail channel.
   * Go to https://app.courier.com and create a new secret workspace
   * For the onboarding process, select the email channel and let Courier and build with Node.js. Start with the Gmail API since it only takes seconds to setup. All we need to do to authorization is login via Gmail. Now the API is ready to send messages.
   * Copy the starter code, which is a basic API call using cURL, and paste it in the a new terminal. It has your API key saved already, knows which email address you want to send to, and has a message already built in.
   * Once you can see the dancing pigeon on the website, you are ready to use Courier

   On the courier dashboard, go to the Designer section which you'll find on the left panel.

   Click on `Create a Notification`.

   Select `Email` channel and click on `Publish Changes` button on the top right corner.

   After publishing the changes, click on the email icon from the left sidebar to create a new email template.

   We need to make a template that looks like this

   <img src="https://i.imgur.com/TACVzwz.png" alt="drawing" width="400"/>

   Select the "T" sign from the bottom toolbar which will add a text block to the template.

   Click on the text block and change text style to H1 from the buttons above the text block. 

   Add this text to it

   ```
   Good Morning, {name}! Here's your daily dose of news.
   ```

   > Note: We will be using curly braces {} to use variables in the template

   Next, make a new text block and copy this in it

   ```
   1. {0.title}
      Description: {0.description}
      Link: {0.link}

   2. {1.title} 
      Description: {1.description}
      Link: {1.link}

   3. {2.title} 
      Description: {2.description}
      Link: {2.link}

   4. {3.title} 
      Description: {3.description}
      Link: {3.link}

   5. {4.title} 
      Description: {4.description}
      Link: {4.link}
   ```

   After this, make a new action block which is the second button after new text block button.

   This will be our unsubscribe button.

   Click twice on the button in the block and you'll see a modal with two fields.

   Write `Unsubscribe` in the first field and in the second field, add this line `https://yourdomainname.com/unsubscribe?email={email}`

   You can change the domain name once our app is hosted.

   Click on publish changes.

   Next, click on settings (gear icon) on the top left corner and click on Notification ID to copy it.

   Save this ID in our `.env` file as `TEMPLATE_ID`

   Next, head over to [Courier Send API documentation](https://www.courier.com/docs/reference/send/message/) and log in if you aren't logged in.

   Copy the Auth token from the right side of the page and add it to our `.env` file as `COURIER_API_KEY`

   Now we are all setup to use the Courier API!

   Add a variable called `TEST_EMAIL` in `.env`. We will be using this email to recieve notifications for testing.

   Remove the `fetchNews()` function call add this function to `apiHandlers.js`

   ```js
   const sendNews = async (news_obj, email = process.env.TEST_EMAIL, name = "John") => {

      // options for Courier API
      const options = {
         method: 'POST',
         headers: {
               Accept: 'application/json',
               'Content-Type': 'application/json',
               Authorization: 'Bearer ' + process.env.COURIER_API_KEY
         },
         body: JSON.stringify({
               "message": {
                  "to": {
                     email,
                     "data": { ...news_obj, email, name }
                  },
                  "template": process.env.TEMPLATE_ID
               }
         })
      }

      const res = await fetch('https://api.courier.com/send', options)

      if (!res.ok) throw new Error("Courier API is not working")
   }
   ```

   In ```fetchNews``` function, add the following line so we can get the news object as a return value
   ```js
   return news_obj
   ```

   Now, we will combine the two functionalities of fetching news and sending email using the function `fetchNewsAndSendEmail`. Add it to `apiHandlers.js`.

   ```js
   const fetchNewsAndSendEmail = async (email, name, categories) => {
      categories = [...new Set(categories)] // making sure there are no duplicate categories
      let news_obj
      // we will make 5 tries to fetch news and send news incase there is any error from api
      for (let i = 0; i < 5; i++) {
         try {
               news_obj = await fetchNews(categories)
               sendNews(news_obj, email, name.split(" ")[0])
               break
         } catch (err) {
               console.log(err)
         }
      }
   }
   ```

   Add a function call to ```fetchNewsAndSendEmail```
   ```js
   fetchNewsAndSendEmail(process.env.TEST_EMAIL, "John", ['general', 'health'])
   ```

   Run ```node apiHandler.js```

   You should receive an email on the test email which looks something like this

   <img src="https://i.imgur.com/hJ3dEH0.png" alt="drawing" width="400"/>

   We will now remove the function call for ```fetchNewsAndSendEmail``` and instead add this line of code to `apiHandlers.js` so we can use our ```fetchNewsAndSendEmail``` function in another file.

   ```js
   module.exports = fetchNewsAndSendEmail 
   ```

5. Adding users and user information

   We will be using Firebase as our cloud database to store user information

   To setup Firebase:
   * In [Firebase console](https://console.firebase.google.com/), click ```Add Project```, then follow the on-screen instructions to create a Firebase project.
   * Navigate to the Cloud Firestore section of the Firebase console. You'll be prompted to select an existing Firebase project. Follow the database creation workflow.
   * Select Test Mode and finish the setup.

   Getting Firebase service account key:
   * Go to [Google Cloud Console](console.cloud.google) and go to your project you just made.
   * Click on IAM & Admin
   * Click on Service Accounts from the left sidebar
   * Click on the field whose name is firebase-adminsdk
   * Go to keys section
   * Click on ADD KEY -> Create new key
   * Select JSON and click on CREATE

   A JSON file will be downloaded. 

   Copy this file to our project directory and rename it to `google-credentials.json`

   Add `google-credentials.json` to `.gitignore`

   Make a new file called ```db.js```

   Add this code to it. This is the configuration for our Firestore database.

   ```js
   const { initializeApp, cert } = require('firebase-admin/app')
   const { getFirestore } = require('firebase-admin/firestore')

   initializeApp({ credential: cert(require('./google-credentials.json')) })
   module.exports = getFirestore()
   ```

6. Setting up Express

   We will be using ExpressJS to manage our server and routes

   Create a file called ```index.js``` in the project folder and add the following lines to it

   ```js
   const express = require('express')
   const { body, validationResult } = require('express-validator')
   require('dotenv').config()
   const db = require('./db')
   const fetchNewsAndSendEmail = require('./apiHandlers')
   const port = process.env.PORT || 3000

   const app = express()

   app.use(express.static("public")) // this is where our html and css code will reside which we will add later
   app.use(express.urlencoded({ extended: true }))
   ```

   Next, we need to create a route which will handle incoming form data containing user information

   Add this code to index.js

   ```js
   app.post('/',
      // data validation
      body('email').trim().rtrim().notEmpty().withMessage('Email is empty').isEmail().withMessage('Invalid email'),
      body('name').trim().rtrim().notEmpty().withMessage('Name is empty').isLength({ max: 100 }),
      body('checkbox-categories').isArray({ min: 1, max: 5 }).withMessage('Categories should be an array with length between 1 and 5 inclusive'),
      body('checkbox-categories.*').isIn(['sports', 'technology', 'science', 'business', 'health']).withMessage('Invalid categories'),

      // handling the request
      async (req, res) => {
         const errors = validationResult(req);
         if (!errors.isEmpty()) return res.status(400).json({ errors: errors.array() });

         try {
               const data = req.body
               const doc = db.collection('users').doc(data.email);
               await doc.set(data)
               res.send('Record saved successfully')

               // send sample email
               fetchNewsAndSendEmail(data.email, data.name, data["checkbox-categories"])
         } catch (error) {
               res.status(400).send(error.message)
         }

      })

   app.listen(port)
   ```
   Run ```node index.js```

   To test if our code is working, we will use [Postman](https://www.postman.com/) to make a post request to our server.

   <img src="https://i.imgur.com/J4n1lay.png" alt="postman" height="250"/>

   Hit Send

   You should recieve a response saying ```Record saved successfully```

   You can go to your Firebase console and see the newly added data.

   Now, we will be adding the ```/unsubscribe``` route handler for managing requests to unsubscribe a user.

   Add the following code to index.js

   ```javascript
   app.get('/unsubscribe', async (req, res) => {
      // we will be using the email query parameter that we pass in the request as done during setting up the email template no courier
      const email = req.query.email ? req.query.email : "default"
      try {
         await db.collection('users').doc(email).delete()
         res.status(200).send(`Unsubscribed ${email} successfully!`)
      } catch (error) {
         res.status(400).send(error.message)
      }
   })
   ```
   Run `node index.js`

   Make a ```get request``` using Postman to http://localhost:3000/unsubscribe?email=registeredemail

   You should see that the registered user has been deleted from youre Firestore database

7. Cron Job to send daily emails 

   Now, to ensure that the user receives an email daily we need to set up a cron job to carry out this procedure. 

   We use [onrender](https://render.com/) to host our website and since it does not keep the hosted application active at all times, we used a [third party application](https://cron-job.org/en/) instead of our own server to run the cron job which hits a url on our server which calls ```fetchNewsAndSenEmail``` for all users.

   To create a cronjob: 
   * Create an account on [cron-job.org](https://cron-job.org/en/) and click on the "CREATE CRONJOB" button on the dashboard.

   * Fill in the details as displayed in the image below. When filling in the "URL" create your own cron-route-name that you enter in the latter part of the url and set its value in your ```.env``` file as well.

      <img src="https://i.imgur.com/tdovtQw.png" alt="cronjob-setup" width="550" height="400"/>
      
   * Now, we add a handler to our cron job route in ```index.js``` to get all the users in the database and then call the ```fetchNewsAndSendEmail``` function to send an email to all the users. 

      ```javascript
      app.get('/' + process.env.CRON_ROUTE, async (req, res) => {
         const snapshot = await db.collection('users').get() //gets list of all users
         snapshot.forEach(doc => fetchNewsAndSendEmail(doc.get('email'), doc.get('name'), doc.get('checkbox-categories')))
         res.status(200).send("Emails sent!")
      })
      ```
<br>

### Part 2: Frontend

To create the frontend of this project you need to create a ```public``` directory which will contain all the code to create the user interface.

1. Inside the ```public``` directory we create an [assets](https://github.com/kirtan-desai/coffee-mate/tree/main/public/assets) folder that contains the images our website displays. We have two background images -- one for the desktop web screen and the other for the mobile web screen -- and one logo image in this folder.

2. Copy the `index.html` file from [here](https://github.com/kirtan-desai/coffee-mate/blob/main/public/index.html) file and paste inside the ```public``` directory. 

   In the ```<head>``` tag you can add your project name to the title tag.

   Now our ```<body>``` consists of :
      - The header which consists of the title and the logo.
      - The content of the website i.e. the heading and the form. 
         
         Our form contains two input fields to take the user's name and email, checkboxes that consist of the different news categories the users can subscribe to and the sign up button that allows the user to submit the form and register for the daily news notifications.

      - The footer

   Our index.html file also contains code to handle our form submission. In the ```<script>``` tag we create two functions. 
   
   Our first function ```handleData()```, makes sure that if the user tries to sign up without selecting a news category the form won't submit and the user's screen will display an error asking them to select atlease one category. If the user tries to sign up with all fields filled appropriately, we submit our form by making a POST request to the route "/", reset our form to clear all the input fields and display a success message on the screen to inform the user the form has been submitted successfully. 

   ```javascript
      function handleData(event){
         const form_data = new FormData(document.querySelector("form"));
         const form = document.getElementById("myForm")
         //if no category has been selected
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

   Our second function ```checkError()``` makes sure that if the user selects a category checkbox, the error message (if present) should disappear. We do this by setting an onclick listener on every checkbox. When a user clicks on a checkbox the checkError function is invoked for that instance of the checkbox. If the checkbox is checked, the function ensures that the error message is not displayed.

   ```javascript
   function checkError(box) {
        if (box.checked == true){
            document.getElementById("error").style.visibility = "hidden"
        }
    }
   ```


3. Lastly, we create two css files [index.css](https://github.com/kirtan-desai/coffee-mate/blob/main/public/index.css) and [reset.css](https://github.com/kirtan-desai/coffee-mate/blob/main/public/reset.css). 

   We use the reset.css to prevent the default styling of different broswers to affect the styling of our web page.

Run the server using `node index.js` and you should see your homepage on https://localhost:3000 

> You can deploy this web app using any hosting service of your choice. Don't forget to add the ```.env``` and ```google-credentials``` files to your hosted project.
## Conclusions

The website is now ready and you can send daily news notifications to users following these steps!

## About the Authors

[Tanvi Khosla](https://www.linkedin.com/in/tanvi-khosla/) is a CS grad student at Virginia Tech seeking to pursue Software Engineering as a career. She currently works part time as a Software Developer/Graduate Assistant in the Network Infrastructure and Services team in the Division of IT at Virginia Tech. She enjoys working on frontend development and expanding her skillset by learning new tools and technologies as well as creating new projects. 

[Kirtan Desai](https://www.linkedin.com/in/desai-kirtan) is passionate about Software Development and Open Source Software. He works as a Developer at CGIT, Virginia Tech where he builds fullstack GIS web apps. He is currently pursuing his Masters in Computer Science and wishes to pursue a career in Software Development.

## Quick Links

[Courier](https://www.courier.com/)

[Node.js](https://nodejs.org/en/)

[Express.js](https://expressjs.com/)

[Firebase](https://firebase.google.com/)
