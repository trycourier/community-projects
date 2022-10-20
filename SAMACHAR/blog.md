# SAMACHAR (E-Newspaper)

## Background

SAMACHAR helps its user to get a personalized newspaper mailed to their email id along with an SMS on their phone number notifying them about the delivery of their E-Newspaper. As the newspaper is mailed, the news links are getting stored with the user for future reference.
To make newspapers tremendous amount of trees are cut daily. In India itself, more than 240 million copies of newspapers are produced daily. So SAMACHAR(i.e. News in the Hindi language) aims to make this planet greener by switching people from physical newspapers to E-Newspaper so that no more trees have to be cut down to make Newspapers.Moreover in today's fast-running world, people don't get time to hogg from one website to another to check for news related to a particular topic, so SAMACHAR is here to solve this problem!

The tech-stack used in this projects are: Pythons's Streamlit Framework,bits of html/css,Firebase,Google OAuth,Lottie Animations,Courier Platform.


## Instructions



### Part 0: Setting Up

1. Open a python compiler like Pycharm and create a new Project.
2. Create a file main.py.
3. Paste the code given below and install these libraries.
   ```python
   import streamlit as st
   from pyrebase import initialize_app
   import sms   #not a library, a file which we will create and import.
   import emaill  #not a library, a file which we will create and import.
   import authorization  #not a library, a file which we will create and import.
   from PIL import Image
   from streamlit_lottie import st_lottie
   import requests
   ```

### Part 1: 

Newspaper Body Generation
1. Go to https://newsapi.org/ and create an account to get an API-KEY.
2. Create a new file with name news.py and paste the code given below and enter your API-KEY.
   ```python
   import requests
   class Newsfeed:
       base_url='https://newsapi.org/v2/everything?'
       api_key= "[API-KEY]"
       def __init__(self,interest,from_date,to_date,language='en'):
           self.interest=interest
           self.from_date=from_date
           self.to_date=to_date
           self.language=language
       def get(self):
           url=f'{self.base_url}' \
           f'qInTitle={self.interest}&' \
           f'from={self.from_date}&' \
           f'to={self.to_date}&' \
           f'language={self.language}&' \
           f'apiKey={self.api_key}'

           response=requests.get(url)
           content=response.json()
           x=content['articles']

           email_body=''
           for i in x:
             email_body= email_body + i['title']+"\n"+ i['url']+"\n\n"
           return email_body
   ```
   This class gets the news based upon parameters-interest,from date,to date and language, then creates a url to request news and finally forms the body of the email with news Title and news Urls.

In this first Chapter, we will need to authorize our API to send the secret messages. Let’s get started by integrating the Gmail and Twilio APIs, which will enable Courier to send emails and messages from a single API call.

1. Log into your Courier account and create a new secret workspace
2. For the onboarding process, select the email channel and let Courier and build with Node.js. Start with the Gmail API since it only takes seconds to setup. All we need to do to authorization is login via Gmail. Now the API is ready to send messages.
3. Copy the starter code, which is a basic API call using cURL, and paste it in the a new terminal. It has your API key saved already, knows which email address you want to send to, and has a message already built in.
Once you can see the dancing pigeon, you are ready to use Courier to send more notifications. Before we build out our application, we just need to set up the Twilio provider to enable text messages.

4. Head over to “Channels" in the left menu and search for Twilio. You will need an Account SID, Auth Token, and a Messaging Service SID to authorize Twilio.
5. Open twilio.com, login and open the Console, and find the first two tokens on that page. Save the Account SID and Auth Token in Courier.
You lastly just need to locate the Messaging Service SID, which can be created in the Messaging tab on the left menu. Checkout Twilio’s docs on how to create a Messaging Service SID, linked in the description.

6. Once we have all three pieces of information, install the provider and now your Courier account is authorized to send any email or SMS within one API call.
### Part 2: Firebase Authentication

Making Authentication system for our project using Firebase

1. Go to https://firebase.google.com/ and click on Get Started.
2. Click on Add Project and create a new project by giving a name to your project.
3. In Project Overview , go to Project Settings, scroll down and choose Web App(third option).
4. Give a name to your app and click on continue.
5. Copy the below code in your python compiler and replace the fields with the values given to you by firebase and click on continue to console
   ```python
   firebaseConfig = {
       'apiKey': [API-KEY],
       'authDomain': "[AUTH-DOMAIN]",
       'projectId': "[PROJECT ID]",
       'storageBucket': "[STORAGE BUCKET]",
       'messagingSenderId': "[MSI]",
       'appId': "[APP ID]",
       'measurementId': "[MEASUREMENT-ID]"
    }
    firebase = initialize_app(firebaseConfig)
    auth = firebase.auth()
    db = firebase.database()
    storage = firebase.storage()
   ```
6.Now go to: Project Overview-> Build -> Authentication -> Get Started and enable Email/Password.




## Conclusions

[Overview of the project. What's next?]

[Call to action: e.g. try building this project and tag me @shreythecray when you do!]

## About the Author

[Introduce yourself, talk about your interests, types of projects you like building, skills, etc.]

## Quick Links

🔗 [link all resources you use to build this project]
🔗 [e.g. documentation, stackoverflow pages, youtube videos, etc.]
