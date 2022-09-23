---
title: On software choice
author: Michael Flynn
date: '2021-04-22'
categories:
  - Blogging
  - stats
tags:
  - blogging
  - stats
summary: Thoughts on switching from Stata to R
lastmod: '2021-04-22T12:32:39-05:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
---


tldr; If Stata or SPSS works well for what you do, that's great—You should keep using the tool that makes you better at your job! However, as university faculty we're not just making choices for ourselves, but for the students we teach. And those choices have a lot of knock-on effects. In general I think we should tend to err on the side of lowering barriers to entry for students and providing them with the most flexible tools for the job they want to have, but also the jobs they *might* have. While R has a lot of problems and quirks, I think it's the best *relative* option considering these various factors.
 
This post is all about the glories of R. Not really. But maybe kind of? This recent tweet gained some traction and generated a bit of discussion on the subject of proprietary software vs open source alternatives like R:


<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Too many young students are having their time and money wasted by being forced to use out of date tools like SPSS because their professors are scared of the new stuff. There. I said it. <a href="https://twitter.com/hashtag/rstats?src=hash&amp;ref_src=twsrc%5Etfw">#rstats</a> <a href="https://twitter.com/hashtag/python?src=hash&amp;ref_src=twsrc%5Etfw">#python</a> <a href="https://twitter.com/hashtag/datascience?src=hash&amp;ref_src=twsrc%5Etfw">#datascience</a>.</p>&mdash; Keith McNulty (@dr_keithmcnulty) <a href="https://twitter.com/dr_keithmcnulty/status/1382870396408627202?ref_src=twsrc%5Etfw">April 16, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

I'm not going to delve much into the motivations for various faculty avoiding R (or even other alternatives like Python). I don't necessarily think it's fear driving people, but more likely they don't see the utility or think the return on investment is low. But there are two bits that caught my attention that speak to these issues. First, the idea of wasting money. Second, the idea that tools are out of date. 

This post is really primarily about my thoughts on switching from someone who uses R to an R user. I think this is a distinction with a meaningful difference, and it definitely pertains to some of these issues. I went from someone who was trained on and primarily used Stata for all of my work-related modeling (as well as other stuff, like tracking my cat's weight when he was sick) to someone who uses R for just about everything, from modeling, to writing letters, to building my website. Suffice it to say, I'm currently a big fan.

What this post is not intended to be is a screed about why R is *objectively* the best programming language/platform for social scientists. That said, I do think it's *relatively* the best. I learned R about a decade ago while I was still using Stata for most of my work, but remained a very casual user and it took me another several years to make the switch. Part of this was because it felt like the learning curve was pretty long and arduous. Even now, there are a lot of things about R that are a massive pain in the ass. For example, the tedious process of updating a particular package when you have five projects up and running, or a package with a given function name masking the identically-named function you *really* want to call. I'm looking at you `{plyr}`. Even still, I lose a lot of time dealing with some of these issues, so I completely understand and even share a lot of the frustrations that people have with R. Hence the disclaimer about it not being *objectively* best. But, *relatively* I think it has a lot of qualities that are desirable, and even better than, many of the other major software packages used by social scientists. The rest of this post is intended to highlight some of these benefits. 
 

# Cost

First, the most basic issue. Stata was always fairly expensive, but in the last year or two they announced that they were switching to an annual subscription model as opposed to the one-time charge for a perpetual license for whatever version you happened to purchase. Even when I started buying Stata as a graduate student in 2007, it was pretty expensive considering that I was making all of 17,000 per year before taxes. For students who are currently looking to purchase Stata Stata SE is running 179 per year for *student* pricing. The multicore variant is upwards of 375 per year. That's a lot. 

The upside of R here is clear—it's free. There *can* be costs associated with using R and some of the packages you can run across. Querying Google Maps, for example, can incur costs if you run over a certain threshold. But if your goal is primarily to read in data, clean it, and run a linear regression you're on solid ground.

Resources for students in many PhD programs have been getting slim for a while now. Particularly at state schools. And while faculty often have limited control over the baseline rate of pay for graduate students at these institutions, one thing we can control is the cost associated with things like books, software, etc. Is \$180 going to break the bank for a graduate student? Maybe not, but it's also lumped in with a lot of other costs that they incurring while in graduate school. 

It's also not unreasonable to expect that graduate students will need more powerful tools to do the kind of work they're pursuing. Maybe work that they, or their advisers, are conducting requires more powerful software capabilities. With Stata MP running from about 300-400 per year, that's a pretty huge cost. While faculty may have funds to pay for this kind of thing, graduate students might not. This creates a kind of capabilities gap between graduate students and faculty in a way that's nowhere near as evident with alternatives like R. Maybe some departments have licenses to run better versions on multiple lab computers, but this is also a massive expense that many departments may not be in a position to keep up either. 


# Out of date software?

This one's a little trickier. I think it's *kind of* true. Running a linear regression is going to be the same, regardless of what you use. If that's all you're doing then it doesn't really matter, and it's hard to be out of date on this point. But I remember a major motivation for learning R was that Stata's graphical capabilities were extremely poor compared with the things I was seeing coming out of R. They've gotten better for sure. I remember being super excited in 2014 or 2015 when Stata announced that they were *finally* implementing transparency capabilities. I've always enjoyed the data visualization part of the job, so Stata's lag in implementing capabilities that were long-since standard in R was frustrating. But that's the thing, while running a regression is the same whether you do it in R, Stata, or SPSS, the ability to effectively communicate that information is not divorced from running the models themselves. The ability to effectively communicate information is a huge part of our jobs, so the inferior quality of data visualization capabilities in some software packages is a big problem. 

The problem with capability lags isn't limited to data visualization. We see it across the board in modeling capabilities as well. There's a pretty strong assumption on the part of faculty who get by just fine with a given proprietary software platform that their students will also get by just fine with those capabilities. But this just doesn't seem realistic. Stata finally implemented Bayesian modeling capabilities with version 15, but these capabilities had been present on other software platforms for a long time. And while Stata did finally enable users to implement Bayesian modeling, these capabilities were seriously limited compared to other packages available through R (e.g. not running multiple chains). 

The point here isn't that everyone needs to get into Bayesian modeling—that's just an example—but that if students want to learn about state of the art methods, proprietary software packages often aren't going to be the place for it. And state of the art here shouldn't be taken as synonymous with "highly advanced" or "futuristic". But if and when they're interests take them into methodological territory that doesn't overlap perfectly with yours as an adviser the standard of what's expected may diverge considerably from what's capable with the tools you assume are fine. When I learned social network analysis I was taught on Pajek. But looking around at what was being published in Political Science and Sociology, it seemed like everyone was using R, not Pajek. While I learned a lot of substantive knowledge, the implementational/computational skills I learned were useless and I had to go back and relearn a lot of material. Not just because of tastes, but because the software just wasn't capable of estimating the sorts of models that were relevant to my interests. More to the point, the field in practice seemed to have advanced beyond that that particular package was even capable of.

Having to learn a new software package to implement what I wanted wasn't the end of the world, but it did create a lot of costs in terms of time and effort to re-learn basic implementation issues that could have been avoided had the person who taught the course been more in line with the state of the art, so to speak. But I get it—Pajek probably worked fine for them for what they were doing. The problem is that wasn't effective or useful for what was expected of the students they were teaching. The advantage of R in this context is that if you only need to use linear regression for your work, great. But in teaching that material to your students you're also teaching them the basic language and ecosystem they'll need to simply install another package that can handle material that is more relevant to their interests. That is, when you're teaching someone in R they'll use that environment and language for their basic stats and regression skills, but if their interests take them in directions that diverge from their instructor's they don't have to reinvent the wheel by learning a new platform and language. Even if you only ever use frequentist models, it's a short jump for a student whose research interests take into into multilevel Bayesian modeling with packages like `{brms}`. I'd even venture to guess that it will be easier for an adviser to keep up with what the student is doing if they're both working from a shared language as the synatx is largely the same. 


# Extended benefits

Another advantage of R seems to be that learning it seems to have much greater utility for students who are seeking non-academic jobs. Don't get me wrong, there are some private firms that definitely use other proprietary software packages, like Stata or SPSS. In our work we've dealt with survey firms who send us our data in SPSS file formats. I think that's bad practice, but the point is the probability of being able to use some of these software packages outside of academia is not 0. That said, use of R and other languages like Python seem to be much more widespread in industry. Again, proceeding from the premise of "I can afford this software and it's fine for my work" seems faulty when, as instructors and advisers, our role extends beyond what works for us. Not all students are going to end up in academia. Even many of those on the PhD track who aspire to get tenure track jobs will be forced out of the academic job market and into private sector jobs. Not all of them will choose to go into analytics or data science jobs (substantive skills are enormously beneficial, too), but a lot of them will. Combining substantive expertise with methodological skills that are more readily transferable across the academic/industry divide seems like another way to better prepare students for whatever is to come. 

I'd also venture to guess that R makes interdisciplinary collaboration easier. Obviously this isn't ironclad, but my experience has generally been that different disciplines seem to cluster around particular software packages. While not insurmountable, this certainly raises the cost of conducting collaborative work with people outside of your discipline. Not just learning a new language, but figuring out how to unify workflows based around totally separate software languages and the processes those incentivize can be tricky.


# Culture

This last one is admittedly more idiosyncratic, but I just find the culture of the broader R community to be far more welcoming and helpful than Stata. To some extent this isn't surprising. The people contributing to open source software development genuinely like working on what they're doing. They have to, otherwise they wouldn't be doing it. It's not hard to find enthusiastic R users, but I'm not sure the same can be said for alternative platforms. This makes an enormous difference insofar as there is a massive reserve of people who are willing and happy to help when you run into problems. 

But you don't have to be a superfan of the software you're using—it's a tool, a means to an end. But there are other cultural bits that I do think make a difference. I guess I'd label this broadly as "workflow" culture. Again, this sort of thing is present with other software packages. Scott Long's *Workflow of Data Analysis Using Stata* was a great resource for managing your workflow and projects. That said, I think the broader culture that has grown up around and along with R is one that is much more cognizant of tying the initially disparate elements of your projects together into a single, unified, workflow. You can see this with the marriage of statistical modeling and writing in RMarkdown, or in the development of different packages used to communicate information through web-based platforms, like `{shiny}`, `{bookdown}`, or `{pkgdown}`, or interfacing easily with GitHub. 

Obviously a lot of this is still up to the individual user. Rather, using R by itself doesn't automatically make you a better social scientist. It doesn't automatically make your work more replicable. People can still be bad at their job regardless of what platform they use. What it does do is front-loads a lot of the capabilities and ancillary packages that make these things more visible and accessible. It's more likely to make you aware that these tools even exist, whereas they tend to be more obscured on platforms like Stata. 


# Takeaway?

To reiterate, if Stata or SPSS works well enough for what you do, that's great—You should keep using the tool that makes you better at your job! However, as university faculty we're not just making choices for ourselves, but for the students we teach. And those choices have a lot of knock-on effects. In general I think we should tend to err on the side of lowering barriers to entry for students and providing them with the most flexible tools for the job they want to have, but also the jobs they *might* have. While R has a lot of problems and quirks, I think it's the best *relative* option considering these various factors.


