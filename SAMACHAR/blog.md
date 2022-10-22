# SAMACHAR (Personalized E-Newspaper)
![ss](https://github.com/OjasMittal/community-projects/blob/74e75e76c03c49278372f238dee45b9e362daa16/ss1.png)
## Background
SAMACHAR helps its user to get a personalized newspaper mailed to their email id along with an SMS on their phone number notifying them about the delivery of their E-Newspaper. As the newspaper is mailed, the news links are getting stored with the user for future reference.
To make newspapers tremendous amount of trees are cut daily. In India itself, more than 240 million copies of newspapers are produced daily. So SAMACHAR(i.e. News in the Hindi language) aims to make this planet greener by switching people from physical newspapers to E-Newspaper so that no more trees have to be cut down to make Newspapers.Moreover in today's fast-running world, people don't get time to hogg from one website to another to check for news related to a particular topic, so SAMACHAR is here to solve this problem!

The tech-stack used in this project are: Pythons's Streamlit Framework,bits of html/css,Firebase,Google OAuth,Lottie Animations,Courier Platform.


## Instructions to make this Project



### Setup

1. Open a python compiler like Pycharm and create a new Project.
2. Create a file main.py.
3. Paste the code given below and install these libraries.
   ```python
   import streamlit as st
   from pyrebase import initialize_app
   import sms   #not a library, a file which we will create and import.
   import emaill  #not a library, a file which we will create and import.
   import authorization  #not a library, a file that we will create and import.
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

### Part 2: Authorize Courier to send messages using Gmail and Twilio APIs
In this first Part, we will need to authorize our API to send the secret messages. Let’s get started by integrating the Gmail and Twilio APIs, which will enable Courier to send emails and messages from a single API call.

1. Log into your Courier account and create a new secret workspace
2. For the onboarding process, select the email channel and let Courier build with Python. Start with the Gmail API since it only takes seconds to setup. All we need to do to authorize is login via Gmail. Now the API is ready to send messages.
3. Copy the starter code, which is a basic API call using cURL, and paste it in the a new terminal. It has your API key saved already, knows which email address you want to send to, and has a message already built in.
Once you can see the dancing pigeon, you are ready to use Courier to send more notifications. Before we build out our application, we just need to set up the Twilio provider to enable text messages.
4. Head over to “Channels" in the left menu and search for Twilio. You will need an Account SID, Auth Token, and a Messaging Service SID to authorize Twilio.
5. Open twilio.com, login and open the Console, and find the first two tokens on that page. Save the Account SID and Auth Token in Courier.
You lastly just need to locate the Messaging Service SID, which can be created in the Messaging tab on the left menu. Checkout Twilio’s docs on how to create a Messaging Service SID, linked in the description.

6. Once we have all three pieces of information, install the provider and now your Courier account is authorized to send any email or SMS within one API call.
7. Now in project directory create 2 files- sms.py and email.py, paste the respective codes in each one of them and enter your Authorization Token.
 ```python
 from trycourier import Courier
import os
from dotenv import load_dotenv
load_dotenv()
client = Courier(auth_token=os.getenv("AUTH_TOKEN"))
def send_sms(number):
    client.send_message(
            message={
                "to": {
                    "phone_number": f"91{number}"
                },
                "content": {
                    "title": "E-Newspaper(Samachar) mailed",
                    "body": "Your E-Newspaper(Samachar) for today has been mailed to you!"
                },
                "data": {
                    "news": "Your E-Newspaper(Samachar) for today has been mailed to you!"
                },
            }
        )

 ```
 
 ```python
 from trycourier import Courier
import datetime
import os
from dotenv import load_dotenv
load_dotenv()

client = Courier(auth_token=os.getenv("AUTH_TOKEN"))
from news import Newsfeed
def send_email(name,email,interestt):
                news_feed=Newsfeed(interest=interestt,
                                   from_date=(datetime.datetime.now()-datetime.timedelta(days=1)).strftime('%Y-%m-%d'),
                                   to_date=datetime.datetime.now().strftime('%Y-%m-%d'))
                client.send_message(
                        message={
                          "to": {
                            "email": f"{email}",
                          },
                          "content": {
                            "title": f"Your {interestt} Samachar for today",
                            "body": f"Hi {name}! \n Check out today's Samachar(News) on {interestt} \nDo not reply back to this email. \n\n {news_feed.get()}\nRegards\nSamachar",
                          },
                          "data": {"note": f"\nDo not reply back to this email. \n\n {news_feed.get()}\nRegards,\nSamachar",
                          },
                          "routing": {
                                "method": "single",
                                "channels": ["email"],
                            },
                        }
                      )
                return True
 ```
### Part 3: Google Oauth Authentication
1. Now create two files authorization.py and SessionState.py.
2. Follow https://towardsdatascience.com/implementing-google-oauth-in-streamlit-bb7c3be0082c to implement google authentication.
3. In the authorization.py file copy the code below in which I have made some backend and frontend changes to suit our project.Here we have made use of os library to store secrets which have been put in a .env file.
  ```python
  import streamlit as st
import asyncio
from SessionState import get
from httpx_oauth.clients.google import GoogleOAuth2
import os
from dotenv import load_dotenv
load_dotenv()

CLIENT_ID=os.getenv("CLIENT_ID")
CLIENT_SECRET=os.getenv("CLIENT_SECRET")
redirect_uri = os.getenv("REDIRECT")
client_id = CLIENT_ID
client_secret = CLIENT_SECRET


async def write_authorization_url(client,
                                  redirect_uri):
    authorization_url = await client.get_authorization_url(
        redirect_uri,
        scope=["profile","email"],
       # extras_params={"access_type": "offline"},
    )
    return authorization_url


async def write_access_token(client,
                             redirect_uri,
                             code):
    token = await client.get_access_token(code, redirect_uri)
    return token


async def get_email(client,
                    token):
    user_id, user_email = await client.get_id_email(token)
    return user_id, user_email


def main(user_id, user_email):
    st.sidebar.write(f"You're logged in as {user_email}")


def authorize():
    client = GoogleOAuth2(client_id, client_secret)
    authorization_url = asyncio.run(
        write_authorization_url(client=client,
                                redirect_uri=redirect_uri)
    )

    session_state = get(token=None)
    if session_state.token is None:
        try:
            code = st.experimental_get_query_params()['code']
        except:
            st.sidebar.write(f'''<h1>
                     <a target="_self"
                    href="{authorization_url}"><img src='data:image/png;base64,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'width="270"height="50"></a></h1>''',
                             unsafe_allow_html=True)
        else:
            # Verify token is correct:
            try:
                token = asyncio.run(
                    write_access_token(client=client,
                                       redirect_uri=redirect_uri,
                                       code=code))


            except:
                st.sidebar.write(f'''<h1>
                        Session Expired ReLogin <a target="_self"
                        href="{authorization_url}"><img src='data:image/png;base64,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'width="270"height="50"></a></h1>''',
                         unsafe_allow_html=True)

            else:
                # Check if token has expired:
                if token.is_expired():
                    if token.is_expired():
                        st.write(f'''<h1>
                            Login session has ended,
                            please <a target="_self" href="{authorization_url}">
                            login</a> again.</h1>
                            ''')
                else:
                    session_state.token = token
                    user_id, user_email = asyncio.run(
                        get_email(client=client,
                                  token=token['access_token'])
                    )
                    session_state.user_id = user_id
                    session_state.user_email = user_email
                    main(user_id=session_state.user_id,
                         user_email=session_state.user_email)
                    return 2

    else:
        main(user_id=session_state.user_id,
             user_email=session_state.user_email)
        return 2

  ```
 4.Make the .env file and put your keys here
  ```python

CLIENT_ID=[CLIENT_ID]
CLIENT_SECRET=[CLIENT_SECRET]
REDIRECT=[REDIRECT]
```

### Part 4: Firebase Authentication

Making Authentication system for our project using Firebase

1. Go to https://firebase.google.com/ and click on Get Started.
2. Click on Add Project and create a new project by giving a name to your project.
3. In Project Overview , go to Project Settings, scroll down and choose Web App(third option).
![webapp](https://github.com/OjasMittal/community-projects/blob/833566992dd7fd2252f6d99729ee338a05fb1f40/web%20app.png)
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

### Part 5: Making the Webapp

1.Make the icon and title for the tab of your webpage
```python
img = Image.open('icon.jpeg')
st.set_page_config(page_title="SAMACHAR", page_icon=img)
```
2.Set background image,sidebar's background image and hide streamlit footer & Main Menu.
```python
page_bg_img=f"""
<style>
[data-testid="stReportViewContainer"] > .main{{
background-image: url("https://img.freepik.com/free-vector/vector-modern-abstract-geometric-background_260559-237.jpg?size=626&ext=jpg&ga=GA1.2.1693190306.1663954344");
background-size: 110%;
background-position: top left;
background-repeat: no-repeat;
}}
[data-testid="stHeader"]{{
background-color: rgba(0,0,0,0);
}}

# [data-testid="stSidebar"]> div:first-child{{
# background-image: url("data:image/jpeg;base64,{img}");
# background-position: center;
# }}
</style>
 """

hide_menu_style="""
<style>
#MainMenu{visibility:hidden;}
footer{visibility:hidden;}
</style>
"""
st.markdown(hide_menu_style,unsafe_allow_html=True)
st.markdown(page_bg_img,unsafe_allow_html=True)
```
3. Define a function for calling animations whereever needed.
```python
#Animation
def load_lottie_url(url: str):
    r = requests.get(url)
    if r.status_code != 200:
        return None
    return r.json()
```
4. Feed Content into the website using streamlit commands and html tags.
```python
st.markdown("<h1 style='text-align: center; color: #F55F0E;'>SAMACHAR</h1>", unsafe_allow_html=True)
st.subheader('Get your personalised Newspaper mailed to you')
st.write(" ")
flag = 1
if flag == 1:
    st.info("Login through login option in the left drop down menu to use this service")
    st.warning('Make sure you are not logged in with multiple gmail accounts to use google authentication')
    st.write(" ")
col1, col2 = st.columns([5, 2])
with col1:
    st.write(" ")
    st.markdown(
        "<h3 style='text-align: center; color: #FFFFFF;'>SAMACHAR helps you to get a personalised newspaper \N{newspaper} emailed  to you instantly with links of the topic which interests you!</h3>",
        unsafe_allow_html=True)

    st.write(" ")
    st.write(" ")

with col2:
    lottie_animation_1 = "https://assets5.lottiefiles.com/datafiles/98a3d0add75fc3c86f6d6f9b148c111e/Newspaper animation.json"
    lottie_anime_json = load_lottie_url(lottie_animation_1)
    st_lottie(lottie_anime_json, key="news")
st.subheader("**Save time in todays fast running world by getting news of your interest without hogging through various webpages!**")
st.markdown("<h3 style='text-align: center; color: #FFFFFF;'>Get notified by SMS as soon as your newspaper gets mailed to you.</h3>", unsafe_allow_html=True)
st.write("")
```
5. Design the sidebar for Authentication.
```python
st.sidebar.markdown(
        "<h1 style='text-align: center; color: #FFFFFF;'>WELCOME !</h1>",
        unsafe_allow_html=True)
gauth = authorization.authorize()
if gauth != 2:
    st.sidebar.write("OR")
    choice = st.sidebar.radio('Login / SignUp to SAMACHAR', ['Login', 'SignUp'])
    st.write('<style>div.row-widget.stRadio > div {flex-direction:row;}</style>', unsafe_allow_html=True)
    email = st.sidebar.text_input("Enter your email address")
    password = st.sidebar.text_input("Enter your password", type="password")


if gauth == 2:
    flag = 0
    st.sidebar.write("")
    st.sidebar.write("")
    st.sidebar.write("")
    st.sidebar.write("")
    st.sidebar.write("")
    st.sidebar.write("")
    st.sidebar.write("")

    with st.sidebar:
        c1, c2, c3 = st.columns([1, 4, 1])
        with c2:
            lottie_animation_2 = "https://assets1.lottiefiles.com/packages/lf20_enlitdkl.json"
            lottie_anime_json = load_lottie_url(lottie_animation_2)
            st_lottie(lottie_anime_json, key="hello")
```
6. If Google Authentication is chosen display fields of mainpage
```python
    name = st.text_input("Enter your name")
    phone = st.text_input("Enter your contact number")
    email = st.text_input("Enter your email")
    interest = st.text_input("Topic you want news of: ")
    result = st.button("Get SAMACHAR")
    if result:
        success = emaill.send_email(name, email, interest)
        sms.send_sms(phone)
        if success:
            st.write("E-Newspaper Mailed Successfully!")
```
7. If firebase Autentication is chosen then check for SignUp/Login and display fields of mainpage
```python
else:
    if choice == "SignUp":
        handle = st.sidebar.text_input("Please enter your nickname", value="CoolPanda")
        submit = st.sidebar.button('Create my Account')
        if submit:
            try:
                user = auth.create_user_with_email_and_password(email, password)
                st.sidebar.success("Your account is created successfully!")
                st.sidebar.success("Click on Login to continue")
                st.balloons()
                user = auth.sign_in_with_email_and_password(email, password)
                db.child(user['localId']).child("Handle").set(handle)
                db.child(user['localId']).child("Id").set(user['localId'])

            except:
                st.info("This account already exists !")

    if choice == "Login":
        login = st.sidebar.checkbox('Login', key=2)
        if login:
            try:
                user = auth.sign_in_with_email_and_password(email, password)
                flag = 0
                st.sidebar.success("You have Logged in Successfully!")
                name = st.text_input("Enter your name")
                phone = st.text_input("Enter your contact number")
                email = st.text_input("Enter your email")
                interest = st.text_input("Enter your topic of interest")
                result = st.button("Get SAMACHAR")
                if result:
                    success = emaill.send_email(name,email,interest)
                    sms.send_sms(phone)
                    if success:
                        st.write("E-Newspaper Mailed Successfully!")
                        st.balloons()

            except:
                st.info("Enter a valid email/password !")

```
8. Now create a folder called .streamlit inside the project directory and make a config.toml file in it.Add the following code in it for background and text-colours.
```python
[theme]
backgroundColor="#122143"
secondaryBackgroundColor="#131314"
textColor="#f8fafd"
```
### Part 6: Deployment on Cloud
You can deploy the app on any Cloud server of your choice like streamlit.io, heroku,etc.

## Conclusions
Voila!! 
The App is now ready to provide people with their personalized E-Newspaper.
Implementing Machine Learning algorithms to provide news in sorted order from most to least trending news along with sentiment analysis to categorize news as positive news and negative news can be the future scope of this project.

Call to action: try building this project and tag me @OjasMittal when you do!

## About the Author
Ojas Mittal is a Tech. Enthusiast who likes to explore new things.He is pursuing Computer Science from Vellore Institute of Technology.He is an ML Engineer,Full-Stack Web Developer, and an Ethical Hacker.He is also a social worker and  member of Lions Club International. He likes to make projects which can be beneficial for society and can have a global impact.

## Quick Links
🔗  https://www.courier.com/ -> Courier Website.

🔗  https://towardsdatascience.com/implementing-google-oauth-in-streamlit-bb7c3be0082c -> Google OAuth Authentication in Streamlit.

🔗  https://www.youtube.com/watch?v=KLmSfHcOXlc -> Streamlit Login Page Firebase Authentication.

🔗  https://newsapi.org/ -> News API website.
