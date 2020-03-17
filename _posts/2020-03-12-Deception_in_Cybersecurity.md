---
layout: post
date: 2020-03-16
title: 'Deception in Cybersecurity'
tags: [Cybersecurity, Deception, Defense]
featured_image_thumbnail:
featured_image: assets/images/posts/2020/deception.jpg
featured: true
hidden: true
comments: true
---

>All warfare is based on deception. Hence, when we are able to attack, we must seem unable; when using our forces, we must appear inactive; when we are near, we must make the enemy believe we are far away; when far away, we must make him believe we are near. <cite> Sun tzu, The Art of War <cite>


## What is deception technology...

Deception technology is essentially a Cybersecurity defense mechanism which is used in conjuncture with other defense measure such as Firewalls, EDR, IDS/IPS. It is based off the assumption that we know that an attacker is eventually going to find a way into the system and no environment is completely air-gapped. Deception technology is a way to make an attacker show himself/herself by presenting a **bait**, which is essentially an unpatched easy to hack system, and waiting for them to fall for it( which they usually do). 

Honeypots are the first thing that come to mind when I think about deceptive technology as I imagine would be the case for most people. It is a proactive, automated and easy to implement approach to improve the overall security of the organization.

## Why deception Technology...

The real question is **Why not?**. It is true that higher levels of implementation of deception technology are harder to implement but getting started with it is very simple and often underrated. It could be as simple as a file with a click-bait worthy name and a canary token!

"The perimeter is dead". Which means there are a lot more attack vectors than there was once and there are a lot more innovative attackers around which means that it is more crucial now than ever to make sure that if an attacker ever gains access into the network and dealt with. This may involve straight out blocking them access or even better observing their actions while restricting their access to block future attacks of such kind. This can be essential against new types of malware since this now gives us an environment to observe and interact with it without the whole building coming down!

## Why is it effective...

Well to put it bluntly... It's because attackers are lazy. They do not want to put more effort to break into something than they need to and this is a weakness that we can exploit really easily. Just imagine if an attacker sees a file named "password.txt" in a compromised user's system. That is literally the most obvious example of a bait there is but there is still a very high possibilty that the attacker is going to open it. Now all the hours and efforts that he has put into breaking in and staying hidden has essentially gone to waste since the security team knows with certainity that that user is never going to access that file.

That is a very simplistic example but provides that gist of why deception technology is so effective. The file with the canary token can be easily swapped out with a dedicated misconfigured network or an user account with a easy to crack password. It can also be servers that spam misinformation in the network which makes it hard for the attacker to distinguish between what is real and what's not. Any method that makes it harder for the attacker is a win for us.


