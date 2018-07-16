---
layout: post
title:      "Trials and Tribulations Scraping a Dynamic Site"
date:       2018-07-16 18:51:33 +0000
permalink:  trials_and_tribulations_scraping_a_dynamic_site
---


In my journey through the learn curriculum, I am at the point of the CLI app final project, and am required to create a blog post about the project. I decided make a cli app that scrapes Fandango for movie times. My initial thought was that I wanted to make a simple app which could get movie data, allow a user to see details about a specific movie that they selected, and then also see local theaters and movie times by inputting their zip code. I thought that this would be a relatively simple task, as all the information I could possibly need was easily viewable on Fandango and therefore should be easy to scrape. I was wrong.

The first task I decided to complete once I got a basic skeleton made of the classes and UI I needed in the app was to get my scraper getting the data I needed. So I started off by going to Fandango.com, typing in my zip code into the search bar, and looking through the HTML of the webpage. The url included the parameters of the zip code and date, so I did not see any issues with scraping this data. I found the HTML elements which housed the data I needed, and plugged them into my Nokogiri parser. But I kept getting a nil return. Obviously, I thought I made a typo somewhere, so I went over my code character by character for about half an hour before I decided that wasn't the issue. I decided to grab a parent element of the data I needed to see if I could at least start outside and work my way in. When I finally got a return from my html parser, the only thing in it was a spinner. 

So, the site makes a call after the initial load. I started googling ways to get this data, but there was very little information on how to scrape dynamic sites. There was one gem called Watir, but that required downloading a Chrome plugin, and then altering my PATH variable ... not something I would want a user of a potential gem to have to deal with. After a few hours searching and trying different methods to no avail, I decided to accomplish something more managable and return to the problem later. 

I found a site in the Fandango domain that lists upcoming movies and movies in theaters now which appeared immediately upon the initial load, so that looked like a good starting point. I was able to implement some functionality with that data at least. After some time, I had an app that listed out movies coming out this week, movies in theaters, and the ability to look at the details of a user-selected movie. This was ok, but it wasn't the funcitonality I wanted. So, I decided to try to solve my dynamic website problem. I just didn't know how, and I couldn't find anything on google, stackoverflow, etc. that helped me.

So, I started looking at the network requests the site made after initial load. There were a lot of them. After some searching, I found one specifically whose URL looked promising, saying something about movie times in the url. I put the URL in my browser, and got a page of JSON back. Well, problem solved, I thought. Unfortunately, not so fast. 

I used HTTParty to make that same call, and got a single json element that said {message: Not Authorized}....I checked my spelling, and nothing was wrong. I refreshed my broswer page. All the json data was replaced with that one "not authorized" message. And I was back to square one. 

So I tried to cheat and sign up for Fandango API. I got my keys and tested them out. Nothing. A response saying my keys were inactive. I googled it, and it looks like Fandango no longer provides API to anyone but their partners. I couldn't even get cheating to work.

Back to the drawing board. Why was this happening? All the data I needed was on that json page, and there was SOMETHING my browser was doing to get it. I just had to find out what. Using postman, I was able to find out that I could GET that json site for about 60 seconds after loading the initial page in my browser. However, it was only the browser that was giving me that ability. If I made the GET request on Postman or in my code, I would not get that window of opportunity. So  what was my browser doing to make that informaiton available? I went through all the network calls the base site was making, and almost all of them were js files or images. Finally, I inspected the request headers. My browser was sending a cookie, data about it being a browser, and a referer referencing the original website with the call to the json data. That had to be the key. I harvested the cookie in my ruby app using HTTParty, and, and added headers to that json site telling it the referrer was the fandango url, and that the user agent was Google Chrome. And there it was! All the json I could ever want. 

Happy ending to the story - I got my CLI app all the functionality I wanted from the beginning.

I guess what I learned most from this experience is that there is a need to make more tools available for parsing dynamic websites. I don't know if Fandango just doesn't have the best security on it's API and that's why I was able to get it programmatically or what, but client side calls are becoming more and more prevalent, and for services like Fandango which do not provide public API, that could be enough to significantly slow or even stop us from being able to access that data. But, it's a hard process. My fix was very specific to the Fandango site. I hope to get enough experience and knowledge to someday help try to tackle this problem. 

Lastly, this also reinforced the lesson that there are multiple ways to solve a problem, and that persistance is key. Now, there are probably plenty of sites that I would not be able to get similar results from, so I'm not advocating banging your head against an unsolvable problem. But, taking a step back and patiently and thoroughly trying to solve a problem one small step at a time will either solve the problem or give you a clear idea of why the problem is unsolvable, and that another route should be taken. 

The harder the problem, the more satisfying the solution.
