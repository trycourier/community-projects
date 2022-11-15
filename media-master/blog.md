# Media Master

## Background

Currently, to process any image or video, the server’s need to process them while they are being uploaded. This takes a lot of processing power and memory and eventually slows down the uploading process. I created this project to apply processing on the videos and images after they are uploaded to the temporary folder and then transfer the processed files to permanent storage. The project uses Courier and Azure Functions.

Any website that accepts media content (photos, videos, etc) does some or the other form of background processing on said media. If this processing runs in the main thread of the web server, it could lead to horrible response times and also increase susceptibility to a Denial of Service attack. Operations like compression, validation, re-encoding and watermarking are often essential to services of many popular web apps. Even more important is the surety of being notified whenever something goes wrong in crucial operations like these.

I was motivated to create this programme after learning about the flow of video processing on YouTube. So, what exactly does it do? When a user uploads a video, the clip is kept temporarily for processing reasons, and an acknowledgement is delivered to the user on the fly. The user is not obliged to be online during the processing. The video will be processed in stages and layers, and the relevant outcome will be released publicly upon completion of each phase. For example, if a video has been processed and finalised at 144p, it will be immediately accessible for streaming at that resolution without waiting for 240p or 480p render versions.

**<u>Tools used</u>** 

- Azure Functions: Azure Functions is the ideal tool for asynchronous background tasks like ours. Azure Functions is a serverless solution that allows to write less code and maintain less infrastructure. Azure Functions provide "compute on-demand" in two ways:

        1. Allows to implement system's logic into readily available blocks of code. These code blocks are called "functions". Different functions can run anytime when required to respond to critical events.
        2. On increase in requests, Azure Functions meets the demand with as many resources and function instances as necessary - but only while needed. As requests fall, any extra resources and application instances drop off automatically.

- Courier: Courier provides an amazing suite of notification integrations. Courier is an API and web studio for development teams to manage all product-triggered communications (email, chat, in-app, SMS, push, etc.) in one place. This is how Courier works:

        1. Application events can be sent to Courier via their API or SDK
        2. Courier receives and processes events that provide information about the notification content and receiver.
        3. Courier creates a notice template and sends it to the appropriate provider (supports over 60 providers across all channels).

### Roadmap

1. Storing the files

2. Detect uploads on the storage

3. Converting Media files

4. Processing the files

5. Sending a feedback to the users on successful or failed uploading of files

## Instructions

### Part 1: Storing Files

The first step was to store files for processing. I found Azure Blob Storage to be an ideal choice as it flexibly scales up for high-performance computing and is highly secured using authentication with Azure Active Directory and RBAC along with rest encryption.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/y47nbgu0scin79br1id8.png)

Imported BlobServiceClient and ContentSettings from azure.storage.blob to employ Azure Storage resources and blob containers. I have set the container name(CONTAINER_NAME) as “devmrfitz”. Then I called get_container_client to get a reference to a BlobClient object and upload the tempfile.

```

import azure.functions as func

from azure.storage.blob import BlobServiceClient

import os 

AZURE_CONNECTION_STRING = os.getenv('AzureWebJobsStorage')

CONTAINER_NAME = "devmrfitz"

COURIER_API_KEY = os.getenv("COURIER_API_KEY") 

def main(myblob: func.blob.InputStream):

        blob_service_client: BlobServiceClient = BlobServiceClient.from_connection_string(AZURE_CONNECTION_STRING)

```

### Part 2: Detecting uploads on the storage

To detect uploads on Azure Blob Storage, we used Azure Functions. Basically, it is a serverless compute service to run event-triggered code where event, in our case, is upload. It runs a script or piece of code in response to a variety of events. Imported azure.functions and used to azure.functions.InputStream to detect file upload.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/x0rz9q13hrvewutuks7p.png)

### Part 3: Converting Media files

Now, the tedious task was to process video files which was made extremely easy using FFmpeg which uses demuxers to read input files and get packets containing encoded data from. In case of video files, ffmpeg tries to keep them synchronized by tracking lowest timestamp on any active input stream. It uses libavfilter library under the hood to enable the use of filters to process raw audio and video.

### Part 4: Processing the files

To support inversion, resizing, watermarking and trimming and compressing of videos and images, I used PIL(Pillow) library which is a Python Imaging Library which contains all the methods which supported my usecase.

1. Image inversion: ImageOps.invert(image)

2. Image compression: image.save(destination_path, optimize=True, quality=quality)

3. Image resizing: image.resize((width, height))

4. Image watermarking on an Image: image.paste(watermark, (0, 0), watermark) 

5. Text watermarking on an Image: image.paste(watermark, (px, py, px + wx, py + wy), watermark)

6. Trimming videos: subprocess.run([FFMPEG_PATH, ‘-i’, video_path, ‘-ss’, start_time, ‘-to’, end_time, ‘-c’, ‘copy’, output_path, ‘-accurate_seek’], capture_output=True, text=True)

7. Compressing videos: subprocess.run([FFMPEG_PATH, ‘-i’, video_path, ‘-c:v’, ‘libx265’, ‘-crf’, str(crf), ‘-c:a’, ‘copy’, output_path], capture_output=True, text=True)

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/r6yudz9cj5vis90d86i1.png)

### Part 5: Sending feedback

There needs to be a response mechanism that informs the user as well as the server about successful upload and the processing that follows it. To implement this, I used Courier service which is a multi-channel notification service that enables us to send notifications to users using emails(my choice for this project), Discord notification, Slack message, etc. 

Courier passes messages to Integrations via the [Send endpoint](https://www.courier.com/docs/reference/send/message/). We must send an Authorization header with each request. The Courier Send API also requires an event. The authorization token and event values are the "Auth Token" and "Notification ID" we see in the detail view of our “Test Appointment Reminder” event. Click the gear icon next to the Notification's name to reveal them.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ftdtrqxwkgfre9dhjs14.png)

These variables can finally be fed into [Courier's Python SDK](https://pypi.org/project/trycourier/) to facilitate simple notification sending.

Courier works by taking in an event as input via an API, which is in our case, a successful upload. The event is accompanied with the data required for the feedback and details of the recipient. It then generates a notification and sends it through the channel specified. Here, I have chosen emails to be the channel for receiving notification.

Imported trycourier package from Courier service:
```
from trycourier import Courier
```

Added all the metadata:
```

try:

    ...

except UnidentifiedImageError as e:

    # Send courier notification

    if "email" in metadata:

        courier_client = Courier(auth_token=COURIER_API_KEY)

        courier_client.send_message(

        message={
```

-Receiver’s email
```
            "to": {
            "email": metadata["email"],

            },
```
-Subject and body content
```
            "content": {

            "title": "Media Master Warning! Unidentified Image detected",

            "body": "Hello {{emailPrefix}},\n\nAn unidentified file ({{name}}) was uploaded as an image and was not processed. The error generated was: \n\n\n {{error}} \n\nThanks,\nMedia Master",

            },

            "data": {

            "emailPrefix": metadata["email"].split("@")[0],

            "name": myblob.name,

            "error": str(e),

            },
```
-Specific channel
```
            "routing": {

            "method": "single",

            "channels": ["email"],

            },

        }

        )

    logging.warning(f"Unidentified image: {e}")

except UnidentifiedVideoError as e:

    # Send courier notification

    if "email" in metadata:

        courier_client = Courier(auth_token=COURIER_API_KEY)

        courier_client.send_message(

        message={

            "to": {

            "email": metadata["email"],

            },

            "content": {

            "title": "Media Master Warning! Unidentified Video detected",

            "body": "Hello {{emailPrefix}},\n\nAn unidentified file ({{name}}) was uploaded as a video and was not processed. The error generated was: \n\n\n {{error}} \n\nThanks,\nMedia Master",

            },

            "data": {

            "emailPrefix": metadata["email"].split("@")[0],

            "name": myblob.name,

            "error": str(e),                

            },

            "routing": {

            "method": "single",

            "channels": ["email"],

            },

        }

        )

    logging.warning(f"Unidentified video: {e}")        

```

## Conclusion

So, our fast, light-weight and easy-to-use media processing service is ready to used and can help a lot of students and professionals in their day-to-day hustle. 

What new features and improvements can you think of in Media Master? Pull requests and forks are always welcome at the Github repo.

## About the Author

I'm Aditya Pratap Singh, a full stack developer and a junior at IIIT Delhi, India. I have worked with various languages and frameworks all the way from Javascript to C++. Hit me up @devmrfitz on any popular social platform (https://linktr.ee/devmrfitz).

## Quick links

- https://www.courier.com/docs/guides/getting-started/send-message/

- https://azure.microsoft.com/en-us/products/storage/blobs/

- https://learn.microsoft.com/en-us/azure/developer/python/sdk/azure-sdk-overview

- https://azure.microsoft.com/en-in/products/functions/

- https://github.com/devmrfitz/media-master
