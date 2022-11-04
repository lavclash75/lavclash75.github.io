---
title: Discord Open Redirect (Unpatched) 
date: 2022-11-02
categories: [bug bounty, discord]
tags: [bug bounty, 0 day] #minúsculas
---
# Discord Open Redirect (Unpatched) 

First of all I didn't find this vulnerability, I only saw that was exploited in the wild 

How i realized about the vulnerability, a friend receives the typical message of free nitro that we have all received at some point and he talks to me in the DM to tell me if it is real or spam message. I warn you in advance that it is spam, but he makes me curious and I decide to investigate it, to receive the nitro you have to add a bot to your server, I decide to add it to another account that I have to test and on another server where only the account is, when I add the bot I realize that the url You are automatically redirected to another malicious website, which looks like the typical Discord phishing website. This is where I realize the vulnerability and I said "oh how strange" it has been automatically directed to the phishing website. Next we are going to analyze the url to see the open redirect.



## Analysis :

```javascript
https://discord.com/api/oauth2/authorize?client_id=1022148663725801502&permissions=8&redirect_uri=https%3A%2F%2Ffnomm.com%2Fnekotina&response_type=code&scope=identify%20guilds.join%20bot
```
First we have the client_id which is not important
```
client_id=1022148663725801502
```
Then the permissions of the bot here we can see that it is an 8 which is Administrator permissions on the Discord server
```
permissions=8
```
Open Redirect:
Then in the url we have the following redirect uri parameter that has the encoded url of the malicious web
```
redirect_uri=https%3A%2F%2Ffnomm.com
```
URL decoded would look like this
```
https://fnomm.com
```
The rest of the link is to add the bot and send an automatic message to all the people on the server with the same free nitro link.

## Report to the program bug bounty:
Once the URL was analyzed, I proceeded to send the report to the Discord bug bounty program, explaining what was mentioned above.

In the report I also attached a proof of concept video.

Positive point of the Discord bug bounty program is that they responded to me the same day I reported it.

Negative point the program considered that it is social engineering and they closed the report as informative since they considered that it is not a vulnerability and that it was outside the scope of the program. Lastly, I would like to add that it is good that they have bug bounty programs but for example, CSRF is out of scope, and that social engineering is totally out of scope. I am not saying that it does not have to be because in the end phishing could be done of everything but here you are taking advantage of an error to do phishing and be more realistic and let's be honest users are not going to notice that the url changes to another malicious one, because first they already saw that it is a discord.com url, in the end, I think it's not a question of whether or not it's a vulnerability, just making the end user more secure by applying a measure. Thx for reading happy hunting 😎
## The program bug bounty of  Discord 
[https://discord.com/security](https://discord.com/security)
