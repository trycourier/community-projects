# Node.js Tutorial - Sending Secret Messages with the Courier API

### A demo of how it looks like in action:

[![Thumbnail](https://user-images.githubusercontent.com/60550481/202363166-61dfee1f-4ba0-43c4-ba95-e4a4ed5d84f9.png)](https://youtu.be/JFSrnBoGox4)


## What will we be making
One of the best applications of programming is to automate our daily tasks. How repetitive and boring it is to check a database of invoices for a business, comb through them and manually send reminder emails one by one! In this project we are going to make a tool which will automatically parse the spreadsheet, cross reference the reminder date with the current date and send beautiful, professional emails with Courier.

## Tools we will be using
- Python
- [Zocrypt](https://pypi.org/project/zocrypt/)
- Pandas
- Courier


## Instructions

### Part 1: Authorize Courier to send messages using Gmail and design the template for the same

In this first chapter, we will need to authorize our program to send the reminders by email. Let’s get started by integrating the Gmail API, which will enable Courier to send emails with a single API call.

[Log into your Courier account](https://app.courier.com/signup) and create a new secret workspace. For the onboarding process, select the email channel and let Courier and build with Node.js. Start with the Gmail API since it only takes seconds to setup. All we need to do to authorize is login via Gmail. Now the API is ready to send messages.
Copy the starter code, which is a basic API call using cURL, and paste it in the new terminal. It has your API key saved already, knows which email address you want to send to, and has a message already built in.
Once you can see the dancing pigeon, you are ready to use Courier to send more notifications.

Design your notification as shown in this picture

![Image](https://d112y698adiu2z.cloudfront.net/photos/production/software_photos/002/271/144/datas/gallery.jpg)




### Part 2: Make a dummy invoice sheet and read the data through Pandas

Now we use google sheets to make a spreadsheet and load up our data. Instead of authenticating and using google's API to scrape this (a painful process) we will be using pandas to read the spreadsheet. Make a new sheet on google sheets and name it whatever you want. Add the fields and values according to your needs. Here is an example of the workbook.

![Image](https://d112y698adiu2z.cloudfront.net/photos/production/software_photos/002/271/146/datas/gallery.jpg)


Go to share and copy sheet ID and sheet name. This is the url format we are going to be using:
```
https://docs.google.com/spreadsheets/d/{SHEET_ID}/gviz/tq?tqx=out:csv&sheet={SHEET_NAME}

```
Now we simply replace `SHEET_ID` with the sheet ID and `SHEET_NAME` with the sheet name. 


### Part 3: Piecing it all together and writing the code for this

First things first, we must install all the requirements for this project. 
Simply run:

```pip install pandas zocrypt trycourier```


Now we make a new python file and open it in any preferred code editor. Let's start coding. First let's import the required modules into our program:
```py
import os
import pandas as pd
from datetime import date
from zocrypt import decrypter
from trycourier import Courier
```
now while we could store the auth as it is, that would be dangerous and could result in misuse of the Courier API by an unknown third party so we are going to encrypt it with a few lines of code. To do this, we are going to import the encrypter module from zocrypt. Then we are going to generate a key and encrypt the auth using the encrypter module. You can do this in a seperate file or from the command prompt itself:

```py
import os
from zocrypt import encrypter,key
key = key.generate()
to_encrypt=input("Enter your auth token to encrypt: ")
print(f"Your encrypted auth token is: {encrypter.encrypt_text(to_encrypt,key)}")
```

Now we use the encrypted auth and incorporate it in our code: 
```py
key = os.environ["sidkey"]

auth = decrypter.decrypt_text('my encrypted auth token made using encrypt_auth.py', key)

SHEET_ID="1UWT83GYsDvtabgTPZH7L9M-eT4fStvavwmPbVngAc5c"
SHEET_NAME="Sheet1"
URL=f"https://docs.google.com/spreadsheets/d/{SHEET_ID}/gviz/tq?tqx=out:csv&sheet={SHEET_NAME}"

client = Courier(auth_token=auth)
```

After this we write functions for reading the data, querying it and sending emails. We are going to compare the reminder date to the present date and if the present date is greater than the reminder date and the has_paid variable is false

```py
def read_data(url):
    parse_dates = ["due_date","reminder_date"]
    df = pd.read_csv(url, parse_dates=parse_dates)
    return df

def query_data_and_send_emails(df):
    present = date.today()
    email_counter = 0
    for _, row in df.iterrows():
        if (present >= row["reminder_date"].date()) and (row["has_paid"] == "no"): 
            receiver_email=row["Email"],
            name=row["Name"],
            due_date=row["due_date"].strftime("%d, %b %Y"),  
            invoice_no=row["invoice_no"],
            amount=row["Amount"],


            resp=client.send_message(
            message={
                "brand_id": "69MRB0BK80MPBCNXW919N1KRBBNN",
                "template": "4JPR3NPZK84AWWQ2RK5P4CM54KD2",
                "to": {
                "email": receiver_email[0],
                },
                "data": {
                "Invoice_ID": invoice_no[0],
                "name": name[0],
                "amount": amount[0],
                "invoice_no": invoice_no[0],
                "due_date": due_date[0],
                },
            }
            )
            print(resp['requestId'])
            email_counter += 1
    return f"Total Emails Sent: {email_counter}"

df = read_data(URL)
result = query_data_and_send_emails(df)
print(result)
```  
Now we test it. On hitting run if you have reminder dates that are due and the has_paid variable is set to false, it should send the required email. To troubleshoot, you can check [logs](https://app.courier.com/logs/messages) in Courier.
