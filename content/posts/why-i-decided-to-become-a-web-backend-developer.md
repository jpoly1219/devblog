---
title: "Why I Decided to Become a Web Backend Developer"
date: 2022-02-08T11:24:04+09:00
draft: true
---

### Intro

I'm sure you all have experienced this. You just learned your first language, and your full of hopes and dreams, even if the only thing you know at the moment is how to write a simple command-line program that takes a user input and does something with it. Awesome! The question that you now have is, "What now?" This was me after learning Python in codecademy.com. I didn't know what I wanted to do with my newfound powers. 

After digging into different fields, I found a role that fits me quite well. Backend development. Many tutorials out there told me to start off with frontend development. I tried both frontend and backend development, and it was clear as a day that I would become a backend developer. Here are the reasons why I decided to become one.

### Working on frontend is a PITA.

FYI, PITA means "pain in the ass."

Working with UI is tedious. You know the meme where developers are struggling to center a div? I thought that was a joke too. Until it was my turn to actually do it.

Scaffolding a UI that looks like a website from the 90s is doable. However as soon as you try to introduce some styles and Javascript to it, it becomes an arduous task.

Don't get me wrong, I am NOT trying to say that bad design is good. Bad design deverses to be ignored. I also think I have a pretty good taste when it comes to aesthetics. It's just that... translating what's in my mind to reality is tough. Drawing out the UI on a piece of paper isn't too bad, but translating that into code is really annoying. Even if you split the frontend into styling and logic, now you have twice the problem with four times the pain.

If I get a job as a developer and have to see millions of components with fifteen nested divs, each with different styling, I will pull the plug and quit my job. If anyone can help me find resources to make frontend less tedious, please leave a comment down below and save this poor soul.

Also, one issue that I have with frontend development is that the only viable language is Javascript. Javascript just doesn't click with me. I could use Javascript, only because it is the Lingua Franca of the web, but I find using it very stressful. It's like trying to learn English to get a job. Necessary, but unexciting and annoying. I have a favorite quote from an amazing game *Borderlands 2* that perfectly describes how it feels like using Javascript:

> Apologies, but when Claptrap (annoying character) speaks, I feel my brain cells committing suicide, one by one.

### There's more problem solving involved in backend.

I love solving problems. It's the reason why I dived into STEM. Even the games I play are strategy games such as *Civilization 6*, *XCOM 2*, or puzzle games such as *Portal* or *Professor Layton Series*. The reason why I love programming is that there is a lot of problem solving in the process of making something. There's just something cathartic about overcoming huge challenges and thinking logically.

In the world of web development, I think that backend developers are faced with more challenges to tackle than frontend developers.

Frontend developers do some problems as well. I also develop the frontend for my personal projects and have encountered many issues that I had to solve. However, these problems aren't fun to solve. Most issues that I've ran across are:

- Why is the styling not working as intended?

- How do I create X feature to improve user experience?

- How do I render this piece of data into a user-friendly UI?

- Can I just work on something else please?

Except for the last one, the rest don't spark a sense of joy in me. Now let's look at what I can think about instead when working on my backend:

- How do I scale my service to handle millions of users?

- What is the most efficient way to implement a recommendation engine?

- How do I build an API that no one, including me, hates to use?

- What is the best system design for my use case?

I think that backend development provides a plethora of "nerdy" problems that is actually fun to solve. 

Let me share with you some of my experiences working on my projects. In university, I was a member of a project team MRover, where we built mars rovers and competed  against other universities in the University Rover Challenge. I, along with a junior friend, was responsible for creating a web-based simulation for the rover's autonomous navigation code. After brainstorming for some time, we came up with the idea to split the functionality into a frontend Vue app and a backend Python app. I had the opportunity to choose whichever side I wanted, so being an indecisive person I decided to try both. Here's a quick summary of the problems I needed to tackle:

| Frontend                                                                                            | Backend                                                                                                |
|:--------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| How do I render the rover, the field and the obstacles?                                             | How do I handle the connection between the frontend, backend, and the autonomous navigation code?      |
| What is the best way to display the data that the API is sending over?                              | How do I emulate the terrain data so that the autonav code can process it?                             |
| Where do I place these buttons? How do I implement the CSS to make the UI more pleasing to the eye? | How do I implement the CV portion of the code? Trigonometry?                                           |
| Why is this prop not being sent over? Why is this event dispatch not being picked up?               | How do I design a schema for the database table to store the rover's performance data each trial runs? |

I don't know about you, but to me, the backend portion was a lot more fun and enjoyable to work on.

### Cool things are mostly implemented in the backend.

Think about it for a bit. When you think of technology and amazing services, what comes to mind first? For me, Google's search engine, Netflix's recommendation engine, Uber's ETA calculation and driver-passenger matching system, and similar services come first. Most services that drive the modern world is processed in the backend. If any of these can be implemented on the frontend, I'd be surprised.

Most modern web applications require a backend to run properly. That shows how important the backend is in providing reliable service to the user. Intuitive UI is what pulls the users in, but a quality service is what keeps the user. Imagine an app with a beautiful UI that can only print "Hello World" over and over. Imagine having to log in every single time you refresh the page. Imagine not being able to upload videos to TikTok, and having to download the full 10min videos from YouTube to just watch them locally on your phone. Imagine Uber's ETA calculation is in fact a random number generator. Apps have to pull data from somewhere, and even if the client device is powerful enough, it is a waste of resources and bad UX to run complciated calculations within the client.

Along with services, system design is a big part of the backend experience. I think the whole monolith vs. microservice debate, contanerization, caching, streaming, and servers are very interesting. Have you seen the Netflix microservice example? If you haven't, check it out here: [Mastering Chaos - A Netflix Guide to Microservices - YouTube.](https://www.youtube.com/watch?v=CZ3wIuvmHeM&t=1745s)When I first watched that keynote and the speaker showed me the gif of Netflix's internal architecture, I thought to myself, *"That is freaking amazing."*, followed by, *"I want to make something like this for a living."* Relating back to the second point, if you are going to solve problems and build things for a living, you might as well solve fun problems and build awesome things.

### Conclusion

I guess everyone's mileage may vary. For me, backend development is a lot more interesting than frontend development. Judging by how important interest is in terms of long-term learning, I think I made the right choice to walk the path of the backend developer.

Thanks for reading! Share your opinions down below. You can also view this post on [medium.com]() and []my personal site]().



*Have fun,*

Jacob Kim
