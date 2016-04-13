---
title: The Modern Joel Test 
layout: post
---

[The Joel Test](http://www.joelonsoftware.com/articles/fog0000000043.html) is a classic article written [Joel Spolsky](http://www.joelonsoftware.com/) that describes 12 criteria often cited as useful to prospective employees when looking for a new software job.

> **The Joel Test**
> 
> 1. Do you use source control?
> 2. Can you make a build in one step?
> 3. Do you make daily builds?
> 4. Do you have a bug database?
> 5. Do you fix bugs before writing new code?
> 6. Do you have an up-to-date schedule?
> 7. Do you have a spec?
> 8. Do programmers have quiet working conditions?
> 9. Do you use the best tools money can buy?
> 10. Do you have testers?
> 11. Do new candidates write code during their interview?
> 12. Do you do hallway usability testing?

It was written *nearly 16 years ago* and is still applicable today. For instance, you'd be surprised how many software shops still don't use source control.

But for contemporary software development, I think the test could use a few updates.

**1. Do you use *modern* source control?**

These days it's not always enough to use source control. Developers often expect or demand to work with the best tools available. Such as Git or Mercurial (if anyone still uses that). You'll find a lot of shops still using SVN or TFSVC. And that's fine, but I think there are huge productivity gains to be had by adopting Git or Hg.

**2. Can you make a build in one step?**

Nothing new here. Except that it's common for builds to be scripted using something like make/rake/psake or some other task runner. Those scripts should be committed and versioned alongside the source code. XML build configurations are out, builds scripts are in.

**3. Do you use *Continuous Integration with automated builds*?**

Daily builds are a thing of the past. Automated CI builds are basically *want you need* to develop quality software.

And in the case of SaaS, I would take it a step further: **do you automate deployment into your qa/staging/prod environments?**

**4. Do you have a bug database?**

As most shops are practicing some form of "agile" development whereby requirements are represented as user stories or tasks in a backlog, bug tracking is usually integrated into that workflow. There are *countless* tools available: TFS, Jira, Pivotal Tracker, etc.

However I think it's important that the development teams control these tools - there's nothing worse than having a workflow dictated by management. Developers *hate* paperwork and documentation, and neither directly result in higher quality software.

**5. Do you fix bugs before writing new code?**

Nothing to do add here.

**6. Do you have an up-to-date schedule?**

What are you iterations like? Is it more SCRUM with strict time-boxed sprints, or is it a looser kanban approach? When and how are planning/retrospective meetings held? Standups?

How do you plan/schedule work on technical debt and other non-user-story tasks?

**7. Do you have a spec?**

How is the backlog managed? Who are the product owners and business analysts? Are user stories well written and include screenshots and acceptance criteria?

**8. Do programmers have quiet working conditions?**

Said differently, **do you have an open office plan?** This is probably quite subjective, but I personally hate open or shared offices.

How are meetings scheduled? As a developer, can I block off my morning and/or afternoon such that *no meetings are allowed*?

Can I work remotely? Are there communication and collaboration tools in place to facilitate remote workers?

**9. Do you use the best tools money can buy?**

Do I get to control what's installed on my machine? Do you use a proxy server or otherwise limit my Internet access? In the "enterprise" world it's very common to find restrictions which actively prevent a software developer from doing his or her job.

**10. Do you have testers?**

Do you do automated testing?

**11. Do new candidates write code during their interview?**

I'm not going to open this can of worms today...

**12. Do you do hallway usability testing?**

I think some take this rule too literally - especially regarding remote teams. Everybody can do screen sharing these days with ease.

It's definitely important, but there's a balance to find and it's hard. You don't want to just interrupt other people and drag them into your space to look at something, but you also don't want to burden folks with meetings and scheduling.