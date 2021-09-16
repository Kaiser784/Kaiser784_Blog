---
description: 'Aug 31, 2021 - Present'
---

# Logs

### Day - 1

Read "First Bug-Bounty" Writeups to get different perspectives on how people tackled at initial points, dealt with dupes and rejections.

Started practicing Golang on [CodeWars](https://www.codewars.com). I used CodeWars because I think it's the only website which matched my vibe of not being Competitive Programming but at the same time exponentially increasing my understanding of a language with each kata. I learnt python the same way from here a year back.

**Some References**

* [https://medium.com/@soufianehabti/my-story-with-xss-ed017bdc44c4](https://medium.com/@soufianehabti/my-story-with-xss-ed017bdc44c4)
* [https://infosecwriteups.com/bug-bounty-broken-api-authorization-d30c940ccb42](https://infosecwriteups.com/bug-bounty-broken-api-authorization-d30c940ccb42)
* [https://infosecwriteups.com/idor-in-jwt-and-the-shortest-token-you-will-ever-see-uid-1234567890-4e02377ea03a](https://infosecwriteups.com/idor-in-jwt-and-the-shortest-token-you-will-ever-see-uid-1234567890-4e02377ea03a)

### Day - 2

Only read some writeups about API and Recon couldn't do more due to academic deadlines. Will try to wrap up academic stuff faster so they don't pile up and takeaway time from me.

Also decided to take the Dante Pro Lab from HTB which upon completing can be used to bypass synack waitlist, will start working on it from tomorrow.

**Some References**

[https://orwaatyat.medium.com/your-full-map-to-github-recon-and-leaks-exposure-860c37ca2c82](https://orwaatyat.medium.com/your-full-map-to-github-recon-and-leaks-exposure-860c37ca2c82)

### Day - 3

Did Initial Recon on Dante and found 2 flags. Learnt different approaches in Recon and credential bruteforcing.

* Can't disclose more because of HTB policy of not disclosing writeups of Active machines.

Read some recommended writeups on medium.

**Some References**

[https://medium.com/techiepedia/my-easiest-critical-bug-81c341a0d6d4](https://medium.com/techiepedia/my-easiest-critical-bug-81c341a0d6d4)

### Day - 4

Further solved the Dante Labs learnt more about pivoting and different ways we can do it using different tools. Learnt about `sshuttle` and `chisel` had to cycle through other methods to check which one is allowing me to scan the network without a hitch.

Did a bit of Wreath Network till the pivoting part to get some basic understanding of this concept. Planning to do Wreath parallely with Dante.

Chose a program with large scope on [hackerone](https://hackerone.com/hacktivity), gonna start recon and start understanding the working of application and read the previously disclosed reports.

**Some References**

[https://tryhackme.com/room/wreath](https://tryhackme.com/room/wreath) [https://medium.com/techiepedia/nuclei-the-best-tool-for-automating-vulnerability-testing-4bf2a9ba5189](https://medium.com/techiepedia/nuclei-the-best-tool-for-automating-vulnerability-testing-4bf2a9ba5189)

### Day - 5

Properly pivoted to the first internal network in Dante spent some time fine tuning my approach and scans. Did recon on a bunch of ip's I found on the network, Organized it and got initial foothold on one of them.

Tried to get used to nuclei by performing basic scans

**Some References**

### Day - 6

Took a break from solving Dante and explored more on the program I chose for the Bug Bounty and learn different and effective ways to use nuclei.

Read about a bug called Dependency Confusion which looked really interesting gonna try to hunt for this.

**Some References**

* [https://medium.com/infosec/nuclei-a-bug-bounty-tool-502fd4d1e098](https://medium.com/infosec/nuclei-a-bug-bounty-tool-502fd4d1e098)
* [https://dhiyaneshgeek.github.io/web/security/2021/09/04/dependency-confusion/](https://dhiyaneshgeek.github.io/web/security/2021/09/04/dependency-confusion/)

### Day - 7

Moved forward in Dante but got stuck at horizontal priv esc, will see if I can resolve it tmrw or move on to other IP's.

Started doing proper Recon on the program instead of just reading different ways to do it.

**Some References**

### Day - 8

Got the horizontal priv esc did some search for root but got fed up with that box and moved on to the next IP and rooted it.

I realized I didn't have a proper way of documenting my Bug Bounty recon it was just a ton of Markdown in Obsidian, Obsidian is great but didn't feel right for this task. Sot went on a search and found this [bbrf](https://github.com/honoki/bbrf-client).

Found this 100 Red team Projects repo, without code just titles to try out might give it a try.

**Some References**

[https://github.com/kurogai/100-redteam-projects](https://github.com/kurogai/100-redteam-projects)

### Day - 9

Pwned a simple machine on the Dante Network. Decided to start looking to reading some Security Research/Conference papers. Didn't do much other than that due to academics.

**Some References**

### Day - 10

Got foothold of a windows machine, I don't have much experience with windows machines was a good chance to learn more decided to do priv esc later and focus on Bug Hunting setup more for today.

**Some References**

* [https://sankethsharath.medium.com/the-need-for-note-making-and-an-organized-methodology-in-bug-bounty-hunting-f4d23c7db4bf](https://sankethsharath.medium.com/the-need-for-note-making-and-an-organized-methodology-in-bug-bounty-hunting-f4d23c7db4bf)
* [https://www.fourzerothree.in/learning-bugbounty-hunting/](https://www.fourzerothree.in/learning-bugbounty-hunting/)

### Day - 11

Didn't progress much in the Dante Lab, Read a lot of Medium articles regarding security research internships and some Bug Bounties.

**Some References**

* [https://towardsdatascience.com/guide-to-reading-academic-research-papers-c69c21619de6](https://towardsdatascience.com/guide-to-reading-academic-research-papers-c69c21619de6)

### Day - 12

There were too many windows boxes left in the Network, struggled a lot in moving forward but made progress, have to do something with my weakness in windows boxes. Searched and looked out for a better bug bounty program with large scope and less restrictions found one did some basic recon but have to explore the application.

**Some References**

### Day - 13

Took a break from Dante and read up some medium articles about some dorks I could use.

**Some References**

* [https://infosecwriteups.com/first-bug-bounty-ever-sql-injection-da4e64e30851](https://infosecwriteups.com/first-bug-bounty-ever-sql-injection-da4e64e30851)
* [https://medium.com/@corraldev/how-did-i-earned-6000-from-tokens-and-scopes-in-one-day-12f95c6bf8aa](https://medium.com/@corraldev/how-did-i-earned-6000-from-tokens-and-scopes-in-one-day-12f95c6bf8aa)

### Day - 14

Learnt about KiteRunner for API testing and tested it on the program, gonna go through Tib3ius windows priv esc to get some ground in the Dante. But probably not gonna do much progress in next 2 days have to cram for academic exams. But will still try to read some articles.

**Some References**

### Day - 15

Got started with using Postman to test API's was checking some of the endpoints I discovered from KiteRunner, gonna do some proper testing soon a bit occupied with exams. Gonna take a break from Dante until I have a clear mind. Read medium articles on Bug Bounties using Postman.

**Some References**

* [https://asfiyashaikh.medium.com/web-services-api-pentesting-part-1-464313481720](https://asfiyashaikh.medium.com/web-services-api-pentesting-part-1-464313481720)
* [https://infosecwriteups.com/exploiting-apis-with-postman-and-google-chrome-ade13ce74e2b](https://infosecwriteups.com/exploiting-apis-with-postman-and-google-chrome-ade13ce74e2b)

### Day - 16

Was trying out more features of Postman with the target in the program scope, but couldn't do much other than that.

**Some References**

