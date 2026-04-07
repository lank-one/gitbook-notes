---
icon: badge-check
---

# My experience

<figure><img src="../../../.gitbook/assets/image (668).png" alt=""><figcaption></figcaption></figure>

Well, it took some effort, but we did it. Let me tell you about my experience...

## <mark style="color:$danger;">Exam</mark>

I first attempted it just before the exam changed its name; it used to be called CBBH (Certified Bug Bounty Hunter), and now it's CWES (Certified Web Exploitation Specialist). I can confirm that it's the same exam as before, with the same conditions and the same environment.

The first time I tried it, I got stuck. I couldn't get out of the mental loop I had entered and the various rabbit holes (dead ends) in the exam. I got quite frustrated and started to wonder if I had to go through the whole path again, etc.

I took a few weeks to figure out how to approach it, since the exam is so long (7 days for the exploitation + the report), I had to do it during my vacation, because I don't recommend taking this exam while you're working, as it requires quite a few hours a day.

I repeated some skills assessments without any guidance, which, based on the exam questions I saw during my first attempt, were the most important ones. I did the Vulnyx machines created by J4ckie0x17 for this certification (you can find the WriteUps uploaded) and I also made my own checklist to check during the exam and move through the different phases of web pentesting.

The day arrived and... I started off pretty well. On the first day, I got 3 flags (the 2 I got on my first attempt + 1 more). then I continued with my checklist, checking everything step by step. I managed to find many more things than on my first attempt and continued to write everything down. By the third or fourth day, I already had 7 flags out of a total of 10. Although you need 80 out of 100 total points to pass, some flags are worth more points than others, so that's something to keep in mind.

I got stuck trying to get those last 10 points to pass. I tried everything I could think of, and nothing seemed to work... I researched more and it seemed like nothing was going to work, that I wasn't going to be able to get it, because what I was trying was the only way to exploit what I had found... But then a light bulb went off in my head and I thought: What if I restart the victim machine to see if it's crashed? **BINGO**.

I restarted the instance, repeated the attack (changing some parameters but essentially doing the same thing), and it worked. I now had the 80/100 points I needed to pass.

I still had two days left before the deadline, so I decided to try to answer the two remaining questions. I don't know if it was because I was motivated by already having 80 points or if I just got into it, but I managed to get the other two flags and had 100 points.

It was late at night and I had to go back to work the next day (my vacation time to take the exam was running out), but I said, since I'm here, I'll write the report and turn it in now.

## <mark style="color:purple;">Report</mark>

Now it was time to write up the report with all my findings. Throughout the exam, I documented and took screenshots of everything I did, so half the work was already done.

Before taking the exam, I installed [SysReptor](https://sysreptor.com/) on my Kali machine. It's a report generator that has the necessary templates for HTB certifications built in and makes documenting everything much easier.

I put together the whole process of obtaining each of the flags and some other things I found, with some screenshots and other fields that SysReptor asked me to fill in. Between the hours it took and the days of little sleep due to putting in long hours on the exam, I was falling asleep on the keyboard, so I turned to our friend ChatGPT to finish polishing the text of the report.

When I had everything, I generated the report, checked that everything was correct (I ended up with a 53-page report), and uploaded it.

When you upload it, they give you 20 days to get feedback, so I had to wait... After three days, they had already replied with the following (I'll translate it for you, and only the part of the report):

> Here is some constructive feedback on content areas you could focus on to strengthen your skill-set:
>
> * It is always good to add descriptive captions to both command output and shell output so the technical people reading it (and possibly using it as a guide to reproduce + test their remediation efforts) can quickly identify what is being shown.
> * Where possible, it can be good to show a few more screenshots/command output to ensure that anyone reading the report can reproduce. While it's fine to show Burp, it can also be helpful to include the equivalent using cURL since developers may be more familiar with this. Not a hard requirement and can vary from client to client.
> * Even if you were unable to identify any specific issues on certain hosts, it is crucial to document your findings and the actions you took during the assessment. This documentation assists the customer in comprehending whether you conducted tests for specific vulnerabilities and demonstrates your thoroughness in the testing process. Furthermore, it reassures the customer that the tested environment exhibited a relatively secure posture against the identified attacks.
>
> Overall your report was excellent, well presented, precise, neat, and professional, and the tips above are just constructive feedback.

So, you know, don't leave anything out. Out of curiosity, I searched forums to see how many pages of reports other people had written, and some had written three times as many as me, so there's no problem with going into detail.

Once you receive the feedback and find out whether you've passed or not, you can order the merchandise that comes with passing the exam, which includes:

* A T-shirt with the exam logo and designs
* A physical certificate
* Stickers with the exam logo and designs

Of course, I ordered it and am waiting for it to arrive.

So, that's my experience. The certification is tough, but with good preparation and the right mindset, you can pass it without any problems (I know people who had passed and documented everything in 1 or 2 days), so good luck and go for it.

