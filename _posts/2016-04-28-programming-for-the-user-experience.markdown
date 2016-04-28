---
layout: main
title: Programming for the user experience
---

# Programming for the user experience


It seems to me that there's a huge temptation to code for code's sake.

We get caught up in coding style, structural correctness, and writing "beautiful" or "idiomatic" Python. A lot of discussions online will centre around at least one of these ideas, and they are often assumed to be obviously valuable. Many people seem to have their own idea of the "correct" solution or the most readable structure, and tend to talk about it as though everyone else feels the same way. You even see people quoting the Zen of Python, which originated as a joke, as an actual argument to support their point.

I feel as though this mindset seeps into our working processes, especially when it comes to code review. I have spent many hours (way too many hours) mired in minor code style discussions on pull requests, and we've made a lot of decisions based on what would be the Right Thing To Do in terms of code structure rather than working from what we're trying to accomplish. I think our mindset and process is too far away from the specifications and the needs of the client and user. Even our discussions of code style have largely neglected why we have code conventions in the first place, and so we end up chasing someone's idea of the "correct" way to write individual Python statements rather than ensuring the code has a decent level of clarity and efficiency. Code review very rarely catches bugs or bad high-level design decisions, since the reviewers don't have anywhere near enough context to understand the program/task at large, and instead ends up being largely focused on minor nonfunctional changes.


### What I would prefer

Lately, I've started thinking about my code as a key part of the user experience, rather than as some machine that I'm building because it must be built. As a backender, I don't always write code that the user will directly see, although even that goes away if you think of the Javascript and iOS/Android teams working with our APIs as users. But the end user will be affected if we return bad error messages, perform slowly, don't display enough data, crash, and so on. The company, the client, and the end user will all be affected if we deliver late (as will we - stress is not nothing) or fail to include key features.

The end user doesn't care if our code is pretty or if it conforms to style guidelines. Chances are that a typical user has no interest in programming, and even if they did, they don't get to see what we've written. They don't care if the solution is easily extensible, idiomatic, or flexible; they care that it's delivered on time, works well, serves their needs, and feels right to use.

This giant disparity between what I've been doing mechanistically and what we as a team or company are trying to achieve has come to bother me. Time spent on tweaking code style has always been frustrating, but I've also started to see a larger-scale time sink - we often start programming something without considering the end goal for the user, which can lead to some very inefficient solutions.


### What is code style for?

As I alluded to above, we often forget to ask 'why?' on code style itself.

Although not many programmers would consider themselves writers, creating code is an act of communication - both to the computer itself and to the next person who reads this section of the project. Comments, variable names, layout, class structure and so on can all work to improve that next person's understanding and reduce the amount of effort they have to go through to update what you've written.

But writing is hard. Code convention provides a set of rules that purport to make your communication clear. If we stick to these, our code will be consistent (which helps understand new code once you're used to the convention) and well-formatted in some way.

Unfortunately, readability and the aesthetics of formatting are very subjective. Different programmers will prefer different ways of writing code, and often end up debating over insignificant aspects of formatting and layout simply because each considers a different approach to be obviously more readable. Furthermore, any given rule will work better for some programming situations than others. For example, there's a convention of splitting up unit test methods with a newline between the data setup, the call to the method being tested, and the assertions; this doesn't work quite so well when you have a large test class with several three-line tests, so absolutely everything has a newline between it and it's harder to tell where the individual methods are at a glance.

It's hard for us to stomach the idea that there isn't a perfect system sometimes, I think. I get the impression that a lot of programmers wish everything worked in a strictly systematic structured way, like computers do, and so there should be rules and correct solutions that you can apply to any situation. I've felt this way myself at times - certainly it would be nice if we could just do everything the same way and have it be easy - although at that point, you do lose what makes programming interesting. In any case, we're humans writing programs that will be read by humans, tested by humans, specced out by humans and used by humans, and that introduces a bunch of squishy subjectiveness that we can't and (I think) shouldn't move away from.

When I code nowadays, I try to keep in mind that someone is going to read it, and structure it to make the flow and the reasoning behind my decisions as obvious as possible. I try to minimise the amount of time they have to spend hopping between files or looking things up, because I hate doing that. I'm happy to explain anything complex in comments, because in my experience, the idea of code being self-documenting or clear is a bitter, filthy lie. Code is hard.


### Priorities

I've basically said this already, but I think it's more important to get the work done and performing solidly than it is to ensure the code is perfect. We work to close deadlines and often-shifting requirements, and efficiency is important.

I don't have much to recommend on this one besides the obvious. I would like to see a broader shift in our thinking from getting the code written in the right way to getting the project built to spec and on time. We're going to have to do that one way or another, and the more cleanly and efficiently we can do it, the better.

This doesn't necessarily mean writing slapdash code. It means setting aside the last few points of correctness or subjective nonfunctional changes in order to get a pull request in. It means drawing up an implementation based on finding a clean and efficient way to address the requirements. It means not coding for coding's sake. It means caring about the burden we place on ourselves by writing too many tests or too many classes.


### Thinking from the requirements up

I've found it useful to start any difficult task by considering what we're trying to provide to the user.

Implementing a given task based only on what's in the issue and our typical approach to solving problems isn't optimal. It's very easy to end up doing everything with an increasing number of database models, for instance. We might end up with overly large amounts of code being written to do something small, or doing unnecessary work to satisfy a requirement that's changed shape as it's been passed from user to client to account executive to PM to developer.

Every task, requirement and feature has a reason the client or user needs it, and it's always worth finding out what that is before embarking on something complicated. User requirements hold a lot of this information, although that breaks down later in a project as change requests come in. Understanding the user's needs has several benefits. There is more than one way to provide for any given requirement, and we can adjust or push back on issues that are going to be difficult to implement when a simpler solution exists.

Often, as devs, we see only the very end of the specification process, divorced from the motivation behind any of the tasks. For example, we might be given a feature request of "allow the HCP to archive old patient data", when it turns out they just want to be able to see only recent entries in a summary graph. In that case, a date filter is a much better approach to the user's needs. In this particular example, the client originally asked for a date filter, and we gave a high estimate for how long it would take owing to the project being difficult to work on. They then gave us the second feature request a few days later, with no obvious connection between the two (for a little while, I thought the client wanted both, but I think they meant the second as an alternative implementation to the first).

There's a narrative that we devs don't usually see, which is more important to our work than maybe we realise. It's worth asking questions and understanding as much of that story as we can. And for any PMs or accounts folk reading this, it's worth giving us whatever background you know - it helps a lot. Clients are mysterious creatures, but at least we can puzzle out their manifold whims together.


### Code review

One point I haven't made yet is that when a comment is made on a pull request, there's a real cost involved.

It takes time to notice a pull request comment. It takes time to read it, make the change, run tests, and commit, even for a simple syntax tweak. It takes time for the person who made the comment to notice the update, check the PR again, and merge or make further comments. This process often takes an hour, and can block work for a long period of time. Waiting for Travis adds additional heft to the loop. This not only uses up valuable time, but is frustrating and stressful.

Code review is not pointless, and nor is commenting and suggesting changes, but there is definitely a downside to it. Personally, I would prefer it if our process took that into account a little more, especially when it comes to the differences of opinion and minor syntactic tweaks I've discussed here.

In short:

![https://giant.gfycat.com/LastBrokenHorsefly.gif](https://giant.gfycat.com/LastBrokenHorsefly.gif)
