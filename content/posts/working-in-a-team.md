---
title: "Working in a Team"
date: 2022-08-01T01:26:47+09:00
draft: false
---

I spend a lot of time developing solo, working on my personal projects. However, I have worked in big project teams in the past, and have had some experience working with other developers and non-developers. Here are some tips that I would like to share with you. Please understand that this is a beginner’s viewpoint and can be subjective. If there are more tips that you would like to share, please comment below!

During my first year at university, I joined a Mars rover project team. Our team was tasked with creating a web-based simulator to test the rover’s autonomous navigation capabilities. I learned a lot of things and gained a lot from the experience. There were a couple of roadblocks that I have faced, and I think many new developers will face these issues as well.

### Reading other people’s code

Reading other people’s code can be really confusing, especially in a large project. This was one of the areas that I struggled with the most. Heck, I can’t even read my own code the next day, how am I supposed to read others’ code?

#### First, get a good grasp of the tools used in the project.

Figure out what stack your team is using. The language, framework, tooling, anything. If you don’t understand the tools used, you will have a rough time figuring out what a piece of code is doing. If your team uses different languages for different parts of the code, you can usually pick one and choose to work on that part. For me, we were using Vue for the frontend and Python for the backend. I wasn’t very keen on learning frontend tools, so I decided to stick to the Python backend.

#### Second, make full use of your editor/IDE.

I use Visual Studio Code, and it comes with a plethora of tools to make my life easier. The peek feature was a lifesaver in many occasions, because it helped me figure out what a function does. This feature is also useful when you are working with a complicated struct or classes. You can also use a split view, where you can open two tabs side by side. This is useful when debugging a function that refers to another one.

Fireship.io has a [very good](https://www.youtube.com/watch?v=ifTF3ags0XI&t=4s) video on VSCode features, and I highly recommend you watch it.

#### Third, reading code from top to bottom might not be the best way.

One of the biggest mistakes I made was to clone the repository, open any file, and start reading from top to bottom. As you may have expected, this left me even more confused. I wasted too much time reading unimportant code, and couldn’t get to the important ones in time.

The method that I found useful was reading the code alongside the overall system design. Find someone who understands how the system works and ask a lot of questions. Figure out how data is inputted. Understand how that input is processed, and then outputted. For each step, find the code responsible for it and read it.

A function usually calls other functions and creates instances of data structures defined in another file. These create rabbit holes that you need to follow and can get confusing quickly. I started to pin these on the Paint app. If function funcA() called funcB(), then I would screenshot funcA() and funcB(), paste those inside the Paint app, and draw an arrow pointing from funcA() to funcB(). It looks very similar to a UML diagram that describes how classes are laid out.

#### Fourth, you don’t need to understand everything from the get-go.

As an analytical thinker, I find it very uncomfortable to skip over information that I do not understand. I tend to try and understand everything before moving on. This process of fitting knowledge into my logical realm works when I can understand the topic, but it holds me back when topics get more complicated. Sometimes, moving on is the smarter option.

Instead, use the black box method. This term is derived from *black box testing*, a type of software testing that tests the code without having to look at its inner workings. The key point here is that the code itself is abstracted. We can take a page from this book. If you can only understand one part of the code, then consider the rest of the code as a black box for now. You can go back to this later when the time calls. Just keep a note to yourself and don't forget to ask about these other team members.

### You don't always have to chase the fancy roles

A project consists of many different parts, some of them being cooler than the other ones. Some people in our team had a chance to work on the autonomous navigation code. Many people wanted to work on the algorithm because it was the meat of the project. However, I decided to stick with the simulator, writing backend code and libraries. I sometimes thought about chasing the clout. I thought that too many people were riding on the hype train.

It turned out that my work was crucial because it made everyone's life easier. Having a robust backend that communicated with the autonomous navigation code and the frontend was key to testing new features. Writing libraries allowed everyone in the team to quickly scaffold a prototype without having to re-implement every feature on their own.

Your team may be focused on delivering a novel, fancy feature, but you can still be a huge asset to the team even if you don't chase the hype. Find yourself a blue ocean and become a master of that sea.

### Communication

You don’t need to worry a lot about communication when working solo, but it is crucial when working with a team. I was never really an eloquent salesman, but I knew how to form coherent thoughts. If you aren't good at speaking fast like me, you should take your time to collect your thoughts first. Let others talk and listen to what they have to say. Then you can say what you want, and people will tend to listen to you more because A: you have a very well-thought-out idea, and B: it's impolite to cut a shy speaker in the middle.

When there is a bug, not only should you point out the issue but also think of a fix. Anyone can say that module A is not working as expected, but not everyone understands why the bug happens and how one could fix it. Also, pointing out issues makes you sound rather pessimistic. No one wants to work with a pessimistic, cynical person, even if you are right.

Finally, always write documentation as you go. When I first joined the team, I didn’t have any documentation to refer to, so I had a rather slow start. Documenting code is tedious, but it is very important when it comes to kickstarting newbies like me. A well-documented code should highlight what the snippet of code does.

### Conclusion

Thank you for reading the post. I hope that new developers can use this post to kickstart their team development journey. For seasoned veterans, I hope this post was a quick and fun read about a junior developer's experiences.

This post is also available on [Medium](https://medium.com/@jpoly1219/working-in-a-team-e026972fcc4c) and [Dev.to](https://dev.to/jpoly1219/working-in-a-team-2269).
