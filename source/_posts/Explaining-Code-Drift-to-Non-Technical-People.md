---
title: Explaining Code Drift to Non-Technical People
subtitle: Using Documents!
date: 2017-11-18
---

Long-lived branches or forks of code take on a life of their own away from an active mainline such as master or trunk. Someone may commit code to the mainline that conflicts with changes made in that long-living branch or fork. It might be a simple one-line change that only causes a small merge conflict. It could also be a rewrite in the mainline that completely removes code the branch depended on. Mainline waits for no one!

A while back, I was working with a non-technical product owner discussing whether we should pull a half-implemented feature set from the mainline because the company wanted the team to pivot. Not ideal, I know. I explained the risks and time it would take to pull it out versus just finishing and releasing. In the end, the product owner decided to "shelf" the feature.

The concept of "shelving" a feature set is what spurred my explanation of code drift. In his mind, the code could sit in a branch and we would pick it up and continue working on it once the company was ready, without any issue. Here is how I explained what happens to code as it sits in long-lived branches as we continue work on the mainline.

---

Imagine there is a 100 page article. You spend hours pouring over the text. The article is very interesting and you have some ideas on how to improve it. You take notes on some spelling mistakes, write some extra sections with new references you found, and think up a little bit of re-organization that, you hope, will help future readers. You compile all of this feedback and send it off in a beautifully crafted email to the original author. Then you wait.

However, you are not the only person who has been enthusiastically reading this article and suggesting helpful changes to the original author. It turns out, there are hundreds of emails awaiting the author's feedback. She does not get to read your email until a year later. By then, there have been countless revisions to the original text. The article looks nothing like the one you originally read and commented on. That beautifully crafted email, with its specific page numbers and line numbers, no longer provides actionable feedback for the author. If you want your changes to make it in, you will have to re-read the article and re-submit those ideas, if they even make sense in the new context.

Lame.

---

A few months later, I had an opportunity I was not expecting to ever have. We were back together discussing another round of features that were going into the platform. One of the feature sets happen to touch some of the same sections of code as the infamous "shelved" code. Even luckier for me, it touched some UI code. I took the opportunity to remind my non-technical friend of our code drift discussion and of that code that still sits waiting. I used the UI changes, specifically, to highlight how the updates that were being proposed would conflict with the UI updates from the old feature that sits abandoned. I watched as a deeper understanding of code drift and its impact wash over him. I never mentioned it again.
