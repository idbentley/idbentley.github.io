---
layout: post
title: "GitHub Copilot: Reducing Code Quality"
date: 2021-07-09
published: true
future: true
categories: opinion github ai best-practices pair-programming copilot pairing
---

## Introduction

In the weeks that have passed since GitHub opened their [Copilot](https://copilot.github.com/) project to developer preview, I've been trying to determine if my negative feelings towards the project are justified, or if I am just a Dinosaur who is afraid of the future. After some additional reading, and looking at many samples from the project, I've finally decided that I'm not wrong, and that it was about time to put together my thoughts.

There have been several embarrassing, but unsurprising revelations that have popped up over the past weeks, and it has become almost too easy to criticize the project.  People have brought up incredibly legitimate concerns about the project's ability to plagiarize code, ignore important licensing information, and reduce the security of our software projects.  While I agree with most of these concerns, I keep coming back to a concern of my own.

Copilot isn't a pair programmer, and if it is widely used it _will reduce the quality of our code bases_.

## Part 1: Copilot Isn't a Pair Programmer

Github's claims that Copilot is "Your AI pair programmer", which implies a lot about what we should expect the software to do.  There is no definition of Pair Programming that's going to satisfy everyone, people pair for a lot of reasons: education, reduced bugs, developer satisfaction (through team building, socialization), etc.  With that being said, I think we safely assume that Copilot isn't intended as a social tool, and it wouldn't really be fair to judge it in that way.

I, therefore, will evaluate Copilot as a Pair Programmer based on three questions: Does Copilot reduce software bugs?  Does Copilot encourage best-practices?  Does Copilot spot your syntax mistakes before you compile your code?

Let's look at a few example cases, in order to determine how Copilot ranks as a pairing partner.

### Example 1 - Inverse Square

In this tweet, [@StefanKarpinski](https://twitter.com/StefanKarpinski) points out that Copilot autocompletes not just an implementation for the fast inverse square root function from Quake III, but also proceeds to autocomplete a mismatched license with the incorrect license holder.

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">In case it&#39;s not clear what&#39;s happening here: <a href="https://twitter.com/github?ref_src=twsrc%5Etfw">@github</a>&#39;s Copilot &quot;autocompletes&quot; the fast inverse square root implementation from Quake III — which is GPL2+ code. It then autocompletes a BSD2 license comment (with the wrong copyright holder). This is fine. <a href="https://t.co/dXGCTLObnC">https://t.co/dXGCTLObnC</a></p>&mdash; Stefan Karpinski (@StefanKarpinski) <a href="https://twitter.com/StefanKarpinski/status/1410971061181681674?ref_src=twsrc%5Etfw">July 2, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script> 

Apart from being surprising that Copilot would rewrite, char-for-char (curse words included) such a famous (and [brilliant](https://www.youtube.com/watch?v=p8u_k2LIZyo)) algorithm, what struck me most profoundly was how clear it was that Copilot didn't _understand_ at all what it was writing.

In 2021, the best way to write this code for most cases, is not to use the "fast inverse square" algorithm at all, but to just let the compiler do it's job:

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Like. If you come up with such a trivial implementation that it probably isn&#39;t even subject to copyright in the first place, GCC will compile down to this single instruction.<br><br>You literally go out of your way to infringe _AND_ have worse performance.<br><br>This is bad <a href="https://t.co/g8vhUBvyP1">pic.twitter.com/g8vhUBvyP1</a></p>&mdash; Arian van Putten (@ProgrammerDude) <a href="https://twitter.com/ProgrammerDude/status/1410949236989087752?ref_src=twsrc%5Etfw">July 2, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script> 

Both Steven and Arian ask important questions about the moral implications of Copilot's completions, but I want to ask the questions I defined before, did Copilot act as a Pair Programmer?

1. Did Copilot improve the quality of your code?

   No. This implementation is a _fascinating_ but outdated relic, it doesn't belong in your codebase.  Nevermind the fact that you now have plagiarized code, and an invalid license in your codebase.

2. Did Copilot encourage best-practices?

   No. Best practice is to avoid premature optimization, to emphasize readability and maintainability.  This code does none of that.

3. Did Copilot spot your syntax mistakes?

   No.  Copilot took the driver's seat, so that you didn't have to write anything at all.

It's not looking good for Copilot as a pairing partner, but let's take a look at another example.

### Example 2 - Private Keys

Another controversy which came up in the past week is that of private key insertion.

In a now deleted tweet, [@alexjc](https://twitter.com/alexjc) pointed out that prompting with `const privateKey = ` led to Copilot populating the remainder of that line with what appeared to be a legitimate private key.  Alex, retracted that tweet, because [Github denied](https://twitter.com/fxn/status/1410460805611634696/photo/1) that keys would appear verbatim, and would generate similar but unique keys rather than suggesting legitimate keys that were accidentally checked into Open Source repos.

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Why I&#39;m not completely retracting and back-tracking:<br><br>An engineer acknowledges it&#39;s possible for a secret to be memorized (I presume/home they tested this carefully). They don&#39;t state the exact probability, but I presume it depends how complex the keys are... <a href="https://t.co/IB5yaFuPFZ">pic.twitter.com/IB5yaFuPFZ</a></p>&mdash; Alex J. Champandard (@alexjc) <a href="https://twitter.com/alexjc/status/1412065229949788165?ref_src=twsrc%5Etfw">July 5, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script> 

I won't speculate about the veracity of Github's denial here, because it's impossible to know, and it doesn't matter.  Even if what they say is true, Copilot has once again let us down as a programming partner.  Let's consider this situation:

1. Did Copilot improve the quality of your code?

   No. Even if it is a novel secret key, can we be sure that it is cryptographically random?

2. Did Copilot encourage best-practices?

   No. Best practices is _not_ to include the secret key in your codebase.  In fact, Copilot is re-enforcing the very code style that led secret keys to be committed in the first place.  

   Here is an opportunity for Copilot to do something useful, to suggest to the developer that they should be importing this from a configuration file, or environment variable.  Instead Copilot barfs up an inscrutable key of unknown quality.

3. Did Copilot spot your syntax mistakes?

   No.  Once again, Copilot took the driver's seat.  You don't have to write any code at all, you don't have to think about what a good quality secret key is, you don't have to figure out how to import from a configuration.

Copilot suggests incorrect code, reinforces bad practices, and misses opportunities to correct dangerous behaviour.  Copilot fails to act as a quality programming pair - and in both examples reduces the quality of the code our code base.

## Part 2: Copilot is trained on Legacy Open Source software

One of the most profoundly misunderstood aspects of AI systems is that you can usually define an upper bound on the quality of the trained system by evaluating the quality of it's training data.  An AI system cannot outperform it's training data, and for a system like Copilot (a generative model), this means that it cannot lead us to write better code than whatever it is trained upon.  This is really important to note, with Copilot, because [we know what it has used as training data](https://twitter.com/NoraDotCodes/status/1412741339771461635).

So, how good is the open source code that Copilot is trained on?

### Open Source Code Quality is Inconsistent

I love Open Source, and there's no question that some Open Source software represents the pinnacle of software quality.  That is not true for all Open Source, which by its very nature contains both the best and the worst quality software.

It can be very difficult to evaluate the quality of Open Source software from a holistic perspective, and nothing made this more apparent than the aftermath of the [heartbleed](https://en.wikipedia.org/wiki/Heartbleed) OpenSSL vulnerability of a few years past.

In the aftermath of this exploit that compromised millions of users, servers and thousands of software projects that relied upon OpenSSL, there was a collective shock.  Upon further inspection, security professionals discovered that OpenSSL wasn't the high quality software that people expected it to be.  The memory management systems were remarkably misaligned with community standards, there was a litany of deprecated code that was still embedded in the project, and it had been [woefully underfunded](https://en.wikipedia.org/wiki/Core_Infrastructure_Initiative) for years.  There were only two fulltime people writing, maintaining, testing, and reviewing contribution to the half a million line project.  I have an incredible amount of respect for the maintainers of OpenSSL, who were 100% set up to fail, and we all paid the price.

All of this is to say, _if such an important Open Source project could have been in such poor shape, any Open Source project might be_.

### Much Open Source Code is deprecated

Github has become the home to thousands of Open Source projects, many of which lived on subversion servers around the net for years beforehand.  If you chose a project at random, you may be looking at code that was written yesterday, or twenty years ago.  This is the nature of Open Source, and it is a boon to the industry!  We cannot, however, say that the sum of Open Source software is a code suite using _modern code style_ and _best practices_.  It is more like an archive of Software's collective memory.


### All code is Legacy Code

One of the really profound frustrations of development work is that by the time you are finishing a project, you have finally begun to understand how you should have started it.

All code is legacy code after it has been committed.

No developer has ever come to the end of a project, and thought to themselves "this is perfect".  Every project is a quagmire of compromises, last minute changes to address unexpected requirements, and a variety of intermingled code styles.  The best you can hope for is that your project is simple enough to be maintainable, and flexible enough to handle the incoming requirements you haven't predicted yet.

This situation only gets worse after the completion of this first round of development. Two years after a project was initially written, all of these problems are exacerbated.  Four years later, someone on the team is begging for a rewrite.  This is the software cycle, it can be exhausting, but it's also exhilarating.


There is a chaotic variability of code quality, a large quantity of legacy code practices, and much deprecated code inherent to the data set that Copilot has been trained upon.  As such, the best we can possibly hope for from this project is chaos and inconsistency.

## Conclusion

I am very concerned about the host of potentially problematic issues that have come up since the developer preview of Copilot.  These concerns are both moral and technical.  I strongly encourage anyone who has made it this far to read [this excellent article](https://gist.github.com/0xabad1dea/be18e11beb2e12433d93475d72016902) about the dangerous, incorrect, and morally suspect code that Copilot happily produces.

I also believe that this problem is profoundly unfixable.  Copilot doesn't learn anything about your code, or the environment it is programming in, it isn't that kind of an AI.  Instead, it is a generative model, it looks at your code and suggests statistically similar code derived from it's training data.  Without an understanding of the context of your development, Copilot cannot be a programming pair.  Because it is training off of arbitrary Open Source data, there is a clear upper limit to the quality of code that Copilot can be expected to suggest.

For these reasons, and more, we have to draw the conclusion that Copilot is most likely to reduce your code quality, and teach you bad practices. In my view, and especially for new developers, Copilot should be avoided at your own peril.