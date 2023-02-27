# Boost Your Productivity with PowerNap - A Time Tracker Web App Integrated with Courier API for Effortless Notification Management.

## Background

PowerNap is a simple and elegant time tracker web application that allows users to set up a timer for their power nap sessions. The application was built to help individuals improve their productivity by incorporating short naps into their work routine.For this project, we utilized Node.js and Express.js for the backend and vanilla JS, HTML and CSS for frontend development respectively. We also integrated Courier API to enable the app to send push notifications to users.

I chose to use Node.js for its efficient and lightweight nature, which makes it perfect for building scalable applications.For the frontend, I chose to use vanilla JavaScript because it's lightweight, fast, and compatible with all major browsers along with CSS Finally, I incorporated the Courier API to send notifications to users in real-time, which enhances the user experience and improves productivity.


## Instructions

### **Part 1: Building the Frontend**

Building the frontend of PowerNap is a simple process that involves defining the HTML structure, styling the timer using CSS, and implementing the JavaScript code. We will go through each step in detail below.

1. **Defining the HTML structure:**
   
   To build the frontend of PowerNap, we first need to define the HTML structure. We'll create a div to hold the timer, a button to start the timer, another div to hold the music player and an audio element to play a sound when the timer is up.
   ```html
   <div class="timer-container">
   <div class="clock">25:00</div>
    <div class="buttons">
    <button class="start">Start</button>
    </div>
   </div>
   <div class="music-container">
   <audio id="audio" src="lofi-music.mp3"></audio>
   </div>
   ```

   #### The UI consists of two main sections:

   #### **Timer container:**
   This section contains the timer display and a start button. It has a class of "timer-container" for styling purposes.
   div tag with class clock: This is where the countdown timer will be displayed. It starts with a default value of "25:00".
   div tag with class buttons: This is where the start button is located.
   #### **Music container:** 
   This section contains an audio player to play background music during the Pomodoro. It has a class of "music-container" for styling purposes.
   audio tag with id audio: This is where the audio file is linked. The audio file is called "lofi-music.mp3" and it is located in the same directory as the HTML file.

2. **Implementing the JavaScript code**
   
   First, we need to select the HTML elements that we will be working with using the querySelector method. We will also create some variables that we will be using later on:

   ```javascript
   const element = document.querySelector('.clock');
   const startBtn = document.querySelector('.start');
   const audio = document.querySelector('#audio');

   const timer = {
   pomodoro: 1500,
   shortBreak: 300,
   longBreak: 900
   };
   let countdown;
   let paused = false;
   ```

   Next, we will create a function updateClock that will take care of updating the clock display. We will convert the time in seconds to minutes and seconds and display it in the clock element.

   ```javascript
   function updateClock() {
   let min = Math.floor(time / 60);
   let sec = time - min * 60;
   min = min < 10 ? `0${min}` : min;
   sec = sec < 10 ? `0${sec}` : sec;
   element.textContent = `${min}:${sec}`;
   }
   ```
   We will also create a function playSound that will play a sound when the timer reaches zero.
   ```javascript
   function playSound() {
   audio.play();
   }
   ```
   Next, we will create a function updateTimer that will take care of updating the timer. We will use the setInterval method to update the clock every second.

   ```javascript
   const updateTimer = () => {
   timer[mode] = timeRemaining--;
   if (timer[mode] === 0) {
      handleTimerEnd();
      return;
   }
   updateClock(timeRemaining);
   updatePomodoroProgress();
   };
   ```

   Now, we will create the main function startTimer that will take care of starting the timer. We will use the setInterval method to update the clock every second.

   ```javascript
   const startTimer = () => {
   timeRemaining = timer[mode];
   if (pause) {
      updateTimer();
      pause = false;
   }
   pomodoroInterval = setInterval(updateTimer, 1000);
   controlBtn.dataset.action = 'pause';
   controlBtn.textContent = 'pause';
   };
   ```
   Finally, we will add an event listener to the start button that will call the startTimer function when clicked.
   ```javascript
   controlBtn.addEventListener('click', e => {
   const action = e.target.dataset.action;
   if (action === 'start') {
      startTimer();
      
   } else {
      stopTimer();
      pause = true;
   }
   });
   ```

   And the basic functionality of the timer is complete! You can check out the final code [here](https://github.com/arncv/PowerNap/blob/main/public/script.js).

3. **Styling the timer using CSS**
   
   ![PowerNap](https://i.imgur.com/O4njIL2_d.webp?maxwidth=1520&fidelity=grand)


   Using CSS, we can build beautiful and functional UI design for a Pomodoro timer and a music player. The design features a calming blue gradient background, and the typography is set in the Lato font family.
   PowerNap includes a menu with buttons for setting the timer length and selecting the type of session, as well as a progress bar that displays the remaining time. The timer is contained within a circular container, and the countdown clock is prominently displayed in the center.

   The music player has a sleek and modern design, with an album cover image that rotates on hover. The player includes buttons for play, pause, and skip, as well as a progress bar that displays the current position within the song. Additional information about the currently playing song is displayed in a hidden overlay that appears when the player is active.

   

### **Part 2: Backend using Node.js and Express.js**

In order to create a full-fledged web application, we need a backend that can handle server-side logic, database operations, and external API integrations. For this purpose, we will be using Node.js and Express.js.

**Why?**

+ Node.js is a runtime environment for executing JavaScript code outside of a web browser. It is built on top of the Google Chrome V8 engine, which makes it highly performant and scalable.
+  Express.js is a popular web framework for Node.js that provides a simple and flexible API for building web applications.


 **Setting up the project**

   First, we need to create a new directory for our project and initialize it as a Node.js project. We will also install the `Express.js` framework,`body-parser` middleware that parses incoming request bodies, `node-fetch` library to fetch HTTP requests and `dotenv` to store environment variables(very important for privacy.. shh!).
   ```bash
   mkdir PowerNap
   cd PowerNap
   npm init -y
   npm install express
   npm install body-parser node-fetch dotenv 
   ```

   Next, we will create a new file called server.js and add the following code to it:

   ```javascript
   const express = require("express");
   const bodyParser = require("body-parser");
   const app = express();

   app.use(bodyParser.json());

   ```

   In the code above, we imported express and body-parser and created an instance of the express application. We then used app.use() to configure body-parser middleware to parse incoming requests with JSON payloads. This will allow us to easily access the JSON data sent in the request body.


   **Static Files:**

   Next, we need to serve the static files for our webapp. To do this, we will create a public directory in the root directory of our project and place all the static files inside it. Then, we can use the express.static middleware to serve the static files:

   ```javascript
   app.use(express.static("public"));
   ```

   **Routes:**

   Now, we need to define the routes for our webapp. We will define a single route that returns the index.html file when a GET request is made to the root URL:

   ```javascript
   const path = require("path");

   app.get("/", (req, res) => {
   res.sendFile(path.join(__dirname, "public", "index.html"));
   });
   ```
   In the code above, we imported the path module and used the path.join() method to join the current directory name, public, and index.html to form the path to the index.html file.

   **Starting the Server:**

   Finally, we need to start the server by listening on a port. We can do this by calling the listen method on the app object:
   
   ```javascript
   app.listen(3000, () => {
   console.log('Server is listening on port 3000')
   })
   ```
   In this part, we created the backend for our PowerNap webapp using Node.js and Express.js. We defined routes to serve the static files and added rate limiting to prevent abuse of our webapp. In the next part, we will add the Courier API to send notifications to the user.

### **Part 3: Integrating the Courier API**

Here comes the juicy part. We will be using the Courier API to send notifications to the user. The Courier API is a powerful and flexible API that allows you to send notifications to your users via SMS, email, push notifications, and more. 

**First things first:**

1. Create a free account on [Courier](https://app.courier.com/signup).
2. Create a new workspace. You can name it anything you want.
3. For the onboarding process, select the email channel aand let Courier build with Node.js.
4. Copy the starter code, which is a basic API call using cURL, and paste it in the a new terminal. It has your API key saved already, knows which email address you want to send to, and has a message already built in.
5. Once you can see the dancing pigeon on the website, you are ready to use Courier.

Now that you've set up your account, let's get started with the integration.

  Create a new file named .env in the root directory of your project. Add your Courier API key, email, phone number, and the message you want to send as environment variables in this file. Your .env file should look something like this:

```bash
APIKEY=<your_courier_api_key>
EMAIL=<your_email>
PHONENUMBER=<your_phone_number>
MESSAGE=<your_message>
```
Use the dotenv package to load the environment variables from the .env file.

```javascript
require("dotenv").config();
```

Create a function named send_message that will send a notification to your email and phone number using the Courier API.

```javascript
async function send_message() {
      const message = process.env.MESSAGE;
      console.log("Notification Successfully sent!")

    const courier_options = {
        method: 'POST',
        headers: {
          Accept: 'application/json',
          'Content-Type': 'application/json',
          Authorization: 'Bearer ' + process.env.APIKEY
        },
        body: JSON.stringify({
          "message": {
            "to": {
              "email": process.env.EMAIL,
              "phone_number": process.env.PHONENUMBER
            },
            "content": {
              "title": "Time to Wake Up!",
              "body": message
            },
            "routing": {
              "method": "all",
              "channels": ["sms","email"]
            },
          }
        })
      };
      
      fetch('https://api.courier.com/send', courier_options)
        .then(response => response.json())
        .then(response => console.log(response))
        .catch(err => console.error(err));

}
```

This is the send_message function which uses the Courier API to send a message to the specified email and phone number with the message defined in the .env file. Here's how it works:

1. The function gets the message from the `.env` file using `process.env.MESSAGE` and sends it via Courier API to the specified email and phone number.
2. The `courier_options` object is defined, which includes the necessary headers for the API call and the JSON body with the recipient email and phone number, message content, and routing information.
3. The `fetch` function is called with the API URL and `courier_options`.

Now that we have defined the send_message function, we need to call it when the timer ends.

In the `script.js` file, add the following code to the `handleTimerEnd` function:

```javascript
fetch('/send-message', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ data: 'message' })
  }).then(response => response.json())
  .then(data => {
    console.log(data);
  })
  .catch(error => {
    console.error('Error:', error);
  });
   send_message();

```
This code sends a POST request to the /send-message endpoint with a JSON body containing a single property data with the value 'message'. We then parse the JSON response and log it to the console.

In the `server.js` file, add the following code to the bottom of the file:

```javascript
app.post('/send-message', (req, res) => {
  send_message();
  res.json({ message: 'message sent' });
});
```
This code defines a POST request handler for the /send-message endpoint. It calls the send_message function and sends a JSON response with a message property.

Now, when the timer ends, the send_message function will be called and a notification will be sent to the user.

### **Starting the Server**

Finally, we need to start the server by listening on a port. 

```javascript
node server.js
```

When the timer ends, the send_message function will be called and a notification will be sent to the user. The notification will look something like this:

![image](https://i.imgur.com/QXDZfMm.png)

and we will get the confirmation message in the terminal:

![image](https://i.imgur.com/95mOdUz.png)

Sweet! We have successfully built a webapp that sends a notification to the user when the timer ends.

![yay](https://i.imgur.com/drPdZY4.gif)

## Conclusions

In this project, we have built a web application called "PowerNap" using HTML, CSS, JavaScript, Node.js, and Express.js. We have also integrated the Courier API to send notification messages to the user. The application allows the user to set a timer for a short nap, and when the time is up, they will receive a notification message reminding them to wake up.

In the future, I plan to add more features to this project. I want to add a feature that allows the user to set a timer of their choice.

I encourage you to try building this project on your own and see how it works. Don't forget to tag me at [@arnvgl](https://twitter.com/arnvgl/) when you do! I would love to see your version of this project.


## About the Author

Arnav Goel is a developer from New Delhi, India. Currently a sophomore at MSIT, Delhi, He is passionate about building products that make a positive impact on people's lives. All things technology fascinate him, and he loves to learn new things. He is also a huge hip-hop fan and loves to play chess.

## Quick Links

🔗 [Courier Docs](https://www.courier.com/docs/reference/send/message/)

🔗 [Courier Get Started with Node.js](https://www.courier.com/docs/guides/getting-started/nodejs/)

🔗 [Express.js](https://expressjs.com/)

🔗 [Node.js](https://nodejs.org/en/)



