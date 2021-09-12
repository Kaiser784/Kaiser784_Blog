---
description: 'Aug 31, 2021 - Present'
---

# Logs

### Day - 1

**Duration - 40 min**

**Summary**

Read "First Bug-Bounty" Writeups to get different perspectives on how people tackled at initial points, dealt with dupes and rejections.

Started practicing Golang on [CodeWars](https://www.codewars.com). I used CodeWars because I think it's the only website that matched my vibe of not being Competitive Programming but at the same time exponentially increasing my understanding of a language with each kata. I learned python the same way from here a year back.

**Some References**

* [https://medium.com/@soufianehabti/my-story-with-xss-ed017bdc44c4](https://medium.com/@soufianehabti/my-story-with-xss-ed017bdc44c4)
* [https://infosecwriteups.com/bug-bounty-broken-api-authorization-d30c940ccb42](https://infosecwriteups.com/bug-bounty-broken-api-authorization-d30c940ccb42)
* [https://infosecwriteups.com/idor-in-jwt-and-the-shortest-token-you-will-ever-see-uid-1234567890-4e02377ea03a](https://infosecwriteups.com/idor-in-jwt-and-the-shortest-token-you-will-ever-see-uid-1234567890-4e02377ea03a)

### Day - 2

**Duration - 25 min**

**Summary**

Only read some writeups about API and Recon couldn't do more due to academic deadlines. Will try to wrap up academic stuff faster so they don't pile up and take away time from me.

Also decided to take the Dante Pro Lab from HTB which upon completing can be used to bypass synack waitlist, will start working on it from tomorrow.

**Some References**

* [https://orwaatyat.medium.com/your-full-map-to-github-recon-and-leaks-exposure-860c37ca2c82](https://orwaatyat.medium.com/your-full-map-to-github-recon-and-leaks-exposure-860c37ca2c82)

### Day - 3

**Duration - 2 hrs**

**Summary**

Did Initial Recon on Dante and found 2 flags. Learned different approaches in Recon and credential brute-forcing.

* Can't disclose more because of HTB policy of not disclosing writeups of Active machines.

Read some recommended writeups on medium.

**Some References**

* [https://medium.com/techiepedia/my-easiest-critical-bug-81c341a0d6d4](https://medium.com/techiepedia/my-easiest-critical-bug-81c341a0d6d4)

### Day - 4

**Duration - 1hr 45 min**

**Summary**

Further solved the Dante Labs learned more about pivoting and different ways we can do it using different tools. Learned about `sshuttle` and `chisel` had to cycle through other methods to check which one is allowing me to scan the network without a hitch.

Did a bit of Wreath Network till the pivoting part to get some basic understanding of this concept. Planning to do Wreath parallelly with Dante.

Chose a program with a large scope on HackerOne, gonna start recon and start understanding the working of the application and read the previously disclosed reports.

**Some References**

* [https://tryhackme.com/room/wreath](https://tryhackme.com/room/wreath)
* [https://medium.com/techiepedia/nuclei-the-best-tool-for-automating-vulnerability-testing-4bf2a9ba5189](https://medium.com/techiepedia/nuclei-the-best-tool-for-automating-vulnerability-testing-4bf2a9ba5189)

### Day - 5

**Duration - 3hr 25 min**

**Summary**

Properly pivoted to the first internal network in Dante spent some time fine-tuning my approach and scans. Did recon on a bunch of IP's I found on the network, Organized it, and got an initial foothold on one of them.

Tried to get used to nuclei by performing basic scans.

**Some References**

### Day - 6

**Duration - 45 min**

**Summary**

Took a break from solving Dante and explored more on the program I chose for the Bug Bounty and learn different and effective ways to use nuclei.

Read about a bug called Dependency Confusion which looked really interesting gonna try to hunt for this.

**Some References**

* [https://medium.com/infosec/nuclei-a-bug-bounty-tool-502fd4d1e098](https://medium.com/infosec/nuclei-a-bug-bounty-tool-502fd4d1e098)
* [https://dhiyaneshgeek.github.io/web/security/2021/09/04/dependency-confusion/](https://dhiyaneshgeek.github.io/web/security/2021/09/04/dependency-confusion/)

### Day - 7

**Duration - 2hr 45 min**

**Summary**

Moved forward in Dante but got stuck at horizontal priv esc, will see if I can resolve it tmrw or move on to other IP's.

Started doing proper Recon on the program instead of just reading different ways to do it.

**Some References**

### Day - 8

**Duration - 1 hr 45 min**

**Summary**

Got the horizontal priv esc did some search for root but got fed up with that box and moved on to the next IP and rooted it.

I realized I didn't have a proper way of documenting my Bug Bounty recon it was just a ton of Markdown in Obsidian, Obsidian is great but didn't feel right for this task. Sot went on a search and found this [bbrf](https://github.com/honoki/bbrf-client).

Found this 100 Red team Projects repo, without code just titles to try out might give it a try.

**Some References**

[https://github.com/kurogai/100-redteam-projects](https://github.com/kurogai/100-redteam-projects)

### Day - 9

**Duration - 2 hr**

**Summary**

Pwned a simple machine on the Dante Network. Decided to start looking to reading some Security Research/Conference papers. Didn't do much other than that due to academics.

**Some References**

### Day - 10

**Duration - 1 hr 25 min**

**Summary**

Got foothold of a windows machine, I don't have much experience with windows machines was a good chance to learn more decided to do priv esc later and focus on Bug Hunting setup more for today.

**Some References**

* [https://sankethsharath.medium.com/the-need-for-note-making-and-an-organized-methodology-in-bug-bounty-hunting-f4d23c7db4bf](https://sankethsharath.medium.com/the-need-for-note-making-and-an-organized-methodology-in-bug-bounty-hunting-f4d23c7db4bf)
* [https://www.fourzerothree.in/learning-bugbounty-hunting/](https://www.fourzerothree.in/learning-bugbounty-hunting/)

### Day - 11

**Duration - 1 hr**

**Summary**

Didn't progress much in the Dante Lab, Read a lot of Medium articles regarding security research internships and some Bug Bounties.

**Some References**

* [https://towardsdatascience.com/guide-to-reading-academic-research-papers-c69c21619de6](https://towardsdatascience.com/guide-to-reading-academic-research-papers-c69c21619de6)

### Day - 12

**Duration - 3hr 25 min**

**Summary**

There were too many windows boxes left in the Network, struggled a lot in moving forward but made progress, have to do something with my weakness in windows boxes. Searched and looked out for a better bug bounty program with large scope and less restrictions found one did some basic recon but have to explore the application.

**Some References**

