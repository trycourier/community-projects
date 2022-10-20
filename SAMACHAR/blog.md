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
   from PIL import Image
   from streamlit_lottie import st_lottie
   import requests
   ```

### Part 1: Firebase Authentication

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

3. [describe step]

### Part 2: [Replace with Subtitle]

[What are you building in this project]

1. [describe step]
2. [describe step]
3. [describe step]
   ```go
   //code
   ```

## Conclusions

[Overview of the project. What's next?]

[Call to action: e.g. try building this project and tag me @shreythecray when you do!]

## About the Author

[Introduce yourself, talk about your interests, types of projects you like building, skills, etc.]

## Quick Links

🔗 [link all resources you use to build this project]
🔗 [e.g. documentation, stackoverflow pages, youtube videos, etc.]
