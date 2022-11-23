# How to use the Interest Rate Watcher?

## Background
The Interest Rate Watcher was built to monitor interest rate changes made by central banks and to notify end users about said changes. With both the COVID pandemic and the war in Ukraine, the global economy has been in a precarious situation which has made interest rates set by central banks more volatile. Interest rates set by central banks are an important policy tool and affect everything from the cost of a mortgage through to student loan financing. This program was created to ease the process of informing users of changes made by central banks. 

The Interest Rate Watcher was built in PHP and MySQL. PHP was chosen because of its popularity in the web development scene. MySQL was chosen for similar reasons, and because of its seamless integration with PHP. To use the program, it is highly recommended to use a cron job on Linux/UNIX-based systems or to use Windows Task Scheduler on Windows systems. 

## Instructions

### Part 1: Technical Architecture in a diagram 
Before beginning, it is helpful to explain how the Interest Rate Watcher works in diagrammatic form. As stated, the Interest Rate Watcher is built using 
PHP as well as MySQL.

![Courier API Technical Architecture](https://user-images.githubusercontent.com/36359216/203490709-16b2401d-d42f-4d15-9b92-8bde2dc4d28b.JPG)

As the diagram above shows, Interest Rate Watcher is designed to be as versatile as possible which is why it does not have a GUI. In order to even run the Interest Rate Watcher, you need to have:
* [Courier account with API credentials](https://www.courier.com/)
* Server running on environment that can run .php files and MySQL databases
* and perhaps most importantly...internet access!  

To run the Interest Rate Watcher, you need to run a cron job on send.php in Linux/UNIX-based environments or to use Windows Task Scheduler that runs send.php together with the server software (our recommendation is XAMPP which can be downloaded [here](https://www.apachefriends.org/download.html) as at 23rd November 2022. Our recommendation is that send.php should be set to run once every week (central banks typically change their interest rate once per month, so this time schedule should be sufficient to ensure that the latest change is recorded).

The lifecycle of the Interest Rate Watcher begins with send.php. Using the extensive scraping capabilities of send.php, it monitors the web to gather data on the interest rates set by central banks throughout the world, ranging from the Federal Reserve in the United States :us: through to the Bank of England in the United Kingdom 🇬🇧. The website currently used to monitor changes is http://www.worldgovernmentbonds.com/central-bank-rates/ as the website has a consistent and readable structure that can be gathered . Send.php periodically looks at this website for the interest rate set by a number of central banks around the world. It then compares the value stored in the database identified by interest_rate_watcher.sql. If this is the first time the Interest Rate Watcher has been installed or used on a system, then a database is created called *interest_rate_watcher* together with a number of other tables. If the value of the interest rate is the same, then nothing occurs. However, if the interest rate is determined as being different from that stored in the database, send.php uses the Courier API to notify the intended user (you or someone else or both) that there has been an interest rate change by a particular central bank. 

### Part 2: How to amend send.php for your project or website
The Interest Rate Watcher was designed with a "plug-and-play" in mind. This is one reason why it does not have a GUI. It is therefore versatile and can be used in a wide range of settings. Follow the instructions below to use the Interest Rate Watcher

1. Change the email address in the code below to that of your choosing
```
<?php

$emailAddress = "INCLUDE_YOUR_EMAIL_ADDRESS_HERE"; #You should include your email address here. This email address will be used to notify you when the interest rate of the relevant central bank changes.
```

2. Ensure that you have the necessary database credentials in order for the program to work.

```
$servername = "localhost"; #We use the details to log on to the local server. This script can also be used in production. You may need to change this if your authentication details differ.

$username = "root"; #This is the default username in most cases. You may need to change it if your authentication details differ.

$password = ""; #This is the default password in most cases (i.e. nothing). You may need to change it if your authentication details differ.

$conn = mysqli_connect($servername, $username, $password); #We use this line of code to connect to the database. Be sure that your login details are accurate otherwise an error could be thrown!
```

3.


4. [describe step]
5. [describe step]


## Conclusions

The Interest Rate Watcher is an important tool at a time when central banks are taking a more aggresive approach in dealing with the economic situation affecting many countries around the world. The Interest Rate Watcher is a vital tool for financial websites as well as individuals who would like to be alerted about interest rate changes by a central bank which might affect their mortgage and student loan obligations. It is possible that this tool will be expanded to include more important economic indicators. Although the Interest Rate Watcher can be an informative tool in regards to arranging your personal finances, it is not intended to be taken as financial advice; you should always consult a certified financial advisor for advice on how interest rates affect your own personal situation. 

Try to build this project and tag me on Twitter at [@BabatundeOnabaj](https://twitter.com/BabatundeOnabaj) or email me at babatunde.onabajo@churchmapped.com when you do!

## About the Author

Babatunde Onabajo is a programmer from and based in London, United Kingdom. He works for the search engine ChurchMapped Limited, a search engine and travel website which specialises in churches around the world. Babatunde primarily uses PHP and MySQL but has worked on a number of projects ranging from Java to C++. He was named as a Subject Matter Expert (SME) by NASA in 2022. He is a graduate from King's College, London (MSc Brazil in Global Perspective) and Cardiff University (BSc Economics) and briefly studied as an exchange student in International Law at the University of Geneva. Outside of programming, Babatunde is interested in politics, hip-hop and theology.

## Quick Links

🔗 Courier: https://www.courier.com/
🔗 YouTube explanation of Interest Rate Watcher: https://www.youtube.com/watch?v=nwztzPKm3bQ
🔗 PHP: https://www.php.net/
🔗 MySQL: https://www.mysql.com/
🔗 World Government Bonds: http://www.worldgovernmentbonds.com/central-bank-rates/

