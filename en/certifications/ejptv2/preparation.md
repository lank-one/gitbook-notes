---
icon: user-doctor-message
---

# Preparation

In this section, I will share what I did to prepare for what I might encounter on the certification exam. I wanted to be very well prepared, and I will share the resources I used here.

## INE Course

When you purchase the certification, you are given a subscription that allows you to view INE content.

The eJPT-oriented course in this case is [Penetration Testing Student](https://my.ine.com/CyberSecurity/learning-paths/61f88d91-79ff-4d8f-af68-873883dbbd8c/penetration-testing-student), divided into the sections detailed in the [Courses -> eJPTv2 section](../../resources/courses/ejptv2.md).

I recommend watching all the sessions, even though some are very basic; it never hurts to refresh your knowledge. Alexis Hamed is a very good instructor. Take notes on what he explains and shows, as you can refer to them during the exam when you are unsure about a technique or vulnerability.

## INE Labs

In addition to the videos explaining the theory, the course includes more practical videos using techniques in laboratories. These laboratories are also included so that you can put into practice everything you see in the videos.

My recommendation is to do all the labs as you progress through the course, but above all, practice with the “black box” labs found in the Host & Network Penetration Testing: Exploitation section.

There is a lab for both Windows and Linux. These labs are the closest thing you will find to what you will encounter on the exam.

These sections focus on exploiting various services depending on the operating system. Different tools and exploitation techniques are used, so it is important to complete these labs and take note of all the alternatives presented to you.

## Extra Labs

In my case, I completed some machines on the following platforms:

### TryHackMe

In the case of TryHackMe, I completed two machines:

* **Blue**: Windows 7 machine, vulnerable to EternalBlue.
  * Link: [https://tryhackme.com/r/room/blue](https://tryhackme.com/r/room/blue)
* **Vulnversity**: Ubuntu machine running a web server
  * Link: [https://tryhackme.com/r/room/vulnversity](https://tryhackme.com/r/room/vulnversity)

### VulnHub

I also completed two machines on the VulnHub platform, which are quite similar to one or more that you may encounter in the exam itself:

* **DarkHole1**: A very comprehensive machine in which we will have to use different techniques and tools to progress.
  * Link: [https://www.vulnhub.com/entry/darkhole-1,724/](https://www.vulnhub.com/entry/darkhole-1,724/)
* **Symfonos 1**: A machine focused on the web section. I found a very similar one in my exam.
  * Link: [https://www.vulnhub.com/entry/symfonos-1,322/](https://www.vulnhub.com/entry/symfonos-1,322/)

## Content creators

### Xerosec

On the recommendation of a colleague who had recently passed the certification, I watched the video [eJPTv2 Preparation Lab | Exam Simulation by Xerosec](https://www.youtube.com/watch?v=v20IsEd5nUU\&t=18s).

The best thing you can do is watch the entire video (it's 4,5 hours long) and take notes on the techniques and methodology used throughout the video. You'll be very well prepared for the exam.

### J4ckie0x17

On this channel, you will find a playlist of 8 machines that are very suitable for eJPTv2 certification. Some of them are the ones I mentioned earlier.

[Playlist](https://youtube.com/playlist?list=PLOaY4Tlt3Xl6BcWHHYPMjkzbItKiKE42u\&si=6-kck8uBZYZKKZhr)

In the videos, you can see different techniques for exploiting different services that may be very useful during the exam. Try them out and take note of the commands you execute and the techniques you are using so that you know how to apply them during the exam.

## Before the exam

The total time to complete the exam is 48 hours.

My recommendation is that you start it when you have time (a weekend, for example) and take it easy, stop to eat, take a nap, or whatever you want, because if you manage your time well, you will have time to spare.

My main advice is to take it easy and organize the notes you have taken during the course and when doing the labs so that it is easier for you to find what you need.

It is also a good idea to create a structure for your exam notes, because even though it is an exam, you should take notes on everything you do to make your life easier. That way, if you have to stop at any point, you can pick up right where you left off without wasting so much time.

For example, I created the following structure for taking notes during the exam:<br>

<figure><img src="../../../.gitbook/assets/image (301).png" alt=""><figcaption></figcaption></figure>

I create the subnodes Recon, Targets, and Questions.

In Recon, I wrote down the commands I used to perform network reconnaissance and the results I obtained.

Once I identified all the hosts, I created the Targets subnode and within it a subnode for each identified machine. In turn, within each machine subnode, I created other subnodes for each phase I was in on that machine or the service I was compromising.

For example:

<figure><img src="../../../.gitbook/assets/image (302).png" alt=""><figcaption></figcaption></figure>

This allowed me to structure what I had in a more optimal way, and when I took a break, I could quickly pick up where I left off or know which techniques I had already tried on a target and had not worked.&#x20;

I also noted down any credentials obtained or other relevant information that might be useful to me.
