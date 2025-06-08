## What is a rate limiter?

A rate limiter is a system that blocks a user request and only allows a certain number of requests to go through in a specified period. 

⛔

**Why do we need rate limiters?** We need rate limiters to prevent DDOS attacks and malicious users from overloading our site's traffic.

Preventing these attacks helps to reduce costs for the company. They need fewer servers to manage their traffic load. 

These attacks prevent users from accessing a service when it looks like they might overload a system.

![](https://exponent-blog.ghost.io/content/images/2022/11/rate-limiter-general-1-1.png)

Rate limiters can limit traffic and block user requests to protect a system.

The idea is that any one particular user or set of users doesn't exponentially take up most of your computing. You use rate limiters to **better manage computing power.**

## Prerequisites

How do you identify which users to rate limit?

You could segment users based on a unique identifier like their `userID` or `IP address`.

In this example, I'll use `IP address`. Why?

An IP address is **always unique to a user.** You can narrow down your traffic based on a set of IP addresses. 

It's **easy to spoof** user IDs or unique internal identifiers. New user accounts can get made, etc.  It's hard to identify which user IDs are valid and which ones might be spam. 

Using an IP address instead, it's easier to narrow down the source of the problem. 

Once I block a user, the system should notify them. 

![](https://exponent-blog.ghost.io/content/images/2022/11/rl-section-2.png)

Rate limiters identify potentially hazardous users, block their request, and return an error code to let them know they've been blocked. 

Luckily, the HTTP layer has a built-in protocol for that. We will send users a 429 response code from the server**,** letting them know they've been blocked. 

⛔

**HTTP 429** "Too Many Requests" — The user has sent too many requests in a certain amount of time. This response status code tells the user their request has been rate limited.

I'll also consider some kind of logging mechanism on the server side in order do analysis on the traffic patterns in the coming weeks.


### [[Rate Limiting Algorithms]]

