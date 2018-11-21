# An Analysis and Case Study of the Prettier Open Source Project.

The following is a case study I prepared for my open source development class.
It includes notes, links, and an interview I conducted with [Christopher Chedeau (vjeux)](https://github.com/vjeux) in November 2018.

## What is Prettier?

*"[Prettier](https://prettier.io/) is an opinionated code formatter"*

* Web site: [https://prettier.io/](https://prettier.io/)
* Twitter: [https://twitter.com/PrettierCode](https://twitter.com/PrettierCode)
* GitHub: [https://github.com/prettier/prettier](https://github.com/prettier/prettier)

Prettier takes source code input (e.g., JavaScript), parses it into an
[abstract syntax tree (AST)](https://en.wikipedia.org/wiki/Abstract_syntax_tree) and
uses that tree to output a pretty-printed version of the same code.

You can [try it in the browser](https://prettier.io/playground/), entering JavaScript code on the left, and having
it formatted on the right using the [Prettier Playground](https://prettier.io/playground/)

A good introduction to Prettier, and the problems it tries to solve, can be
seen in [this short talk]((https://www.youtube.com/watch?v=hkfBvpEfWdA)) by [James Long](https://twitter.com/jlongste) introducing Prettier at React Conf in 2017.

[![James Long presenting Prettier at React Conf 2017](https://img.youtube.com/vi/hkfBvpEfWdA/0.jpg)](https://www.youtube.com/watch?v=hkfBvpEfWdA)

James said he hit a "breaking point" when he tried to get his editor to properly indent
[JSX code](https://reactjs.org/docs/introducing-jsx.html) and it couldn't do it correctly:

> "I'm going to fix this," and prettier was born.

James says that he wanted to fix this problem because:

1. "Different styles make it hard to work on a team.”
1. Tools like [gofmt](https://blog.golang.org/go-fmt-your-code) were an inspiration
1. Prettier tries to end the *"death by a thousand cuts with nits in PRs"*

He goes on to focus on three areas where Prettier can help developers:

1. **Consistency** across a codebase and across a team
1. **Teachability**, where Prettier can tell you things about your code based on AST (insight into complex code)
1. **Freedom** to not be constrained or bother thinking about the structure of your code.  Just write code.

Today Prettier supports many languages, including:

JavaScript, JSX, Flow, TypeScript, JSON, Markdown, YAML, HTML, Vue, Angular, CSS, Less, SCSS, with support coming for Elm, Java, PHP, PostgresSQL, Python, Ruby, Swift, GraphQL

It's also widely supported in many development tools and editors:

Atom, Emacs, Sublime, Vim, Visual Studio, VSCode, WebStorm

## Project Timeline

*2 Dec 2016* - James tweets about his intial work. [“I have other things I should be working on”](https://twitter.com/jlongster/status/804888882408026113) with a short [animation of what his code could already do](https://video.twimg.com/tweet_video/CyuKKO3XgAAgSlm.mp4)

*Feb 20, 2017* - James [announces](https://twitter.com/jlongster/status/833794481438720000) he's moving Prettier to its [own GitHub Organization](https://github.com/prettier). This is a significant step, in that it sets the stage for Prettier to become a community project vs. one person's.  Doing this always requires someone (the original developer) to be willing to part with some aspect of what they’ve built, namely, absolute control, 1:1 name recognition, etc.  There is a cost to this investment. 

*13 April, 2017* - [Prettier reaches 1.0](https://jlongster.com/prettier-1.0)

*23 May, 2017* - Prettier reaches [10K starts on GitHub](https://twitter.com/jlongster/status/867065314097463302)

*16 Jan, 2018* - Prettier was the [7th most popular JS project on GitHub in 2017](https://twitter.com/PrettierCode/status/953231391323557890)

*26 Feb, 2018* - [Facebook is now using Prettier for 100% of its JavaScript codebase](https://twitter.com/Vjeux/status/968310207305564161)

*20 March, 2018* - [Prettier wins an open source award](https://www.infoworld.com/article/3263736/open-source-tools/the-best-open-source-rookies-of-2018.html)

## Project Development and Structure

Prettier is a massive test suite, which also happens to pretty-print code as a side effect:

|Directory | Lines Of Code (sloc) | % of Project |
|----------|----------------------|--------------|
| `src/`   | 25,122               | 25%          |
| `tests/` | 63,614               | 65%          |

As of November 2018, there are **5,711** tests in in **911** test suites.

> "Christopher Chedeau pushed this massive test suite and said, get this working!" -- James Long

![vjeux lands tests](images/vjeux-brings-tests.png)

A complete visualization of the repo's development from creation to November 2018
[is available](https://www.youtube.com/watch?v=3p6XR2VeHRw). The structure of the project,
as visualized via gource, really helps to tell a story about how the code was built:
`tests/` on the left dramatically outweigh the `src/` in upper right, and the community's time
is largely focused on building and improving test coverage.

[![Gource Visualization of Prettier Development to Nov 2018](https://img.youtube.com/vi/3p6XR2VeHRw/0.jpg)](https://www.youtube.com/watch?v=3p6XR2VeHRw)

If you look at the [statistics for the Prettier repository](https://www.openhub.net/p/prettier_io),
we see that it starts out as being just one person (James) for the first month, doubles to two in the second month
(James and Christopher Chedeau), then explodes to 28 people in the third. Since then it has averaged 16 contributors
per month consistently.  As of the time of writing, there have been [379 contributors](https://github.com/prettier/prettier/graphs/contributors) in total.

## Interview with Prettier's Christopher Chedeau

I reached out to the early developers of Prettier, and asked about how the project started, and how it
became so successful.  What follows is an email interview I had with Facebook's [Christopher Chedeau (vjeux)](https://github.com/vjeux), who has played a significant role in building, sustaining, and growing prettier and its community.
At the request of Christopher, I've made this public, as I think it's an interesting behind-the-scenes look at
how prettier got made.

### Project History

*[@humphd](https://github.com/humphd) - I think it’s amazing what you, James, and the community have managed to do with Prettier.  I’m interested in the ways in which you accomplished it, and what lessons there are for others to emulate. Prettier could have been quietly ignored like so many other attempts at doing this, but somehow you managed to make it stick. What was the process like?*

[@vjeux](https://github.com/vjeux) - I think that the story of the beginning is interesting. Here’s my side of it.

At my school [EPITA](http://www.epita.fr/), a 5 year computer science-focused engineering school in France, a lot of the curriculum is based on coding projects graded on a 20-point scale. They put in place what they call a *"moulinette"* a little program which automatically checks for code style and for every single mistake you make, you lose 2 points. This was a very harsh but effective way to teach you how to properly format your code. Of course they wouldn’t give you access to the moulinette, only a pdf of the style guide.
 
When [gofmt](https://golang.org/cmd/gofmt/) was announced, I got very intrigued. I knew that the status quo of having everyone to be military trained to format code was a bit silly but I didn’t know of a better way. I’ve been waiting ever since for an equivalent for JavaScript and saw many attempts but none succeeded.
 
[Jordan Walke](https://twitter.com/jordwalke) who I’ve been working with for a few years with [React](https://reactjs.org/) and [React Native](https://facebook.github.io/react-native/) left the React Native team which I was on to work on a new project which involved building a pretty printer for [OCaml](https://ocaml.org/) syntax where anyone can build their own syntax (which turned into [Reason](https://reason.town/jordan-interview). This will become relevant later...
 
At some point, I started becoming friends with [Julien Verlaguet](https://github.com/pikatchu) who built [Hack](https://hacklang.org/). A few months in, he told me that he spent a lot of time designing, building and shipping a Hack formatter and walked me through all the lessons he learned from that project, both in terms of technical design and how to go about shipping it.

I was on the [Nuclide team](https://nuclide.io/) at the time and was looking into making the tooltip that shows the [Flow](https://flow.org/) type when you mouse over a variable better when the type was big. I wrote (and shipped) a small printer based on the concepts that Julien taught me and wrote an internal Facebook note about it.

This note got a fair bit of attention internally and many folks chimed in saying that they’ve hacked on similar things in the past: Kyle Davis pointed to a previous attempt at doing a js formatter he did, Miller Medeiros shared that he built [esformatter](https://github.com/millermedeiros/esformatter) before joining Facebook and Jordan Walke explained some of refmt decisions.
 
[Pieter Vanderwerff](https://github.com/pieterv), who maintained [react-bootstrap](https://github.com/react-bootstrap/react-bootstrap) before joining Facebook, decided to hack on a pretty printer for JavaScript using reason and refmt infrastructure after reading this note.
 
I don’t remember why but I was chatting about this with [James Long](https://twitter.com/jlongster) and he mentioned to me that he started working on a pretty printer by forking [recast](https://github.com/benjamn/recast), a project built by Benjamin Newman, who used to work at Facebook.
 
I got so excited that two people were working on a js pretty printer. I didn’t want those two projects to be another failure, I felt like this time it could happen for real! I knew that at this point what was needed was to get both Pieter and James motivated to work on it during the winter break of 2016. I basically played the role of a cheerleader. I setup the same set of unit tests for the two projects, built small scripts to count down the number of syntax structures left to handle, ran their code through real world examples and summarized classes of issues that needed to be addressed.

Unfortunately, at the end of the winter break, both Pieter and James had to go back to their real work and weren’t willing to focus their time and attention to it. This was so sad because while both projects were early, they both were in a state where you could run it on your codebase and be convinced that with more work you could reasonably convert the entire codebase. I’ve been looking for this for so long.

I did a lot of soul searching and decided that I would focus all my energy and commit to work on it full time. It wasn’t an easy sell to my manager, but fortunately I had enough social capital at Facebook that I was able to do it.
 
I asked myself a lot, “All the previous projects failed, why would this one work?” Here are a few things that convinced me:

1. The output of both projects, even after only a few weeks worth of coding, was so much better than any automatic formatter I’d seen before.
1. I was able to have very long chats with a bunch of people that worked on the projects that didn’t succeed. I learned so much in the process and was able to design plans for all the things they considered reasons for failure and repeat their successes.
1. Both James and myself happened to be pretty well known in the JavaScript community at that time. I honestly never thought I’d be in that position, but here I was, so might as well make use of it.

### Prettier's Tests

*[@humphd](https://github.com/humphd) - I love the role that tests play in the project.  This is something we’ve discussed on Twitter in the past, and I know you are very fond of automated testing.  I think that tests for Prettier, in addition to playing their usual role in software development, also allowed you to gain people’s confidence and trust that their needs were being met in the direction of the project.  Being able to say “this is important to me” via a test helps give a sense of agency, and eventually buy-in from the wider community.  Can you talk about your approach to testing, and how Prettier uses them?*

[@vjeux](https://github.com/vjeux) - I saw that both projects (James’s and Pieter's pretty printers) were being developed by pretty printing a dummy JavaScript file that they expanded as they implemented more code. It was super painful because "coverage" was super small and they had to hunt down examples of real code that contain all the possible syntax in order to print it.
 
The Flow team had built their own JavaScript parser recently, so I figured that they would have all the possible variations in their tests. Bingo! I needed to find a way to actually run the pretter-printers against those files, but I wasn’t super excited about building my own. Around the same time [Christoph Nakazawa](https://twitter.com/cpojer) was working on [Jest snapshot tests](https://jestjs.io/docs/en/snapshot-testing). That feature wasn't designed for this use case but I was able to hack my way around it. I put all those tests in both projects and they were able to iterate much faster as a result.
 
The side effect I didn't really anticipate is that it made it extremely easy to review pull requests. You could just look at the before/after and without even reviewing the code, you could see if the output looked better. Not only that, but if someone wants to play with an idea, they can hack it up, run the tests and see all the things that it would change. Unlike many tests, they were not binary (right or wrong), they showed differences and then it was up to a human to say if the differences were acceptable or not.
 
The project itself also has an interesting characteristic which is that theres a __very__ long tail of features. In order for it to succeed, you need to go through hundreds of small combinations of syntax. I spent the first 6 months grinding through them and what was amazing is that I only committed half the number of commits. The open source community shipped as many of them as I could. The next 6 months I was only at a third. I stopped being actively involved nowadays and I just checked, I’m only at 20%.

### From One Person to a Community

*[@humphd](https://github.com/humphd) - Prettier is like a lot of open source projects, in that it starts out as one person's project, and then manages to draw others around it, before creating a sustaining community.  Let me read you a few tweets that speak to a bit of this, and then I'd love to hear your take on what it took to sustain the project, and get it to where it is today*

> I created Prettier solely for @actualbudget. I was so tired of being bogged down with formatting. @Vjeux pushed me to open-source it and helped make it a standard tool. Solving your own problems is the best!  I'm really amazed at how quickly it was adopted! It thrived as an OSS project - easy to work on w/snapshot tests, and made everyone feel involved in the style decisions. -- James Long, [30 Nov, 2017](https://twitter.com/jlongster/status/936302695652188160)

> I'm losing interest in prettier since it mostly "just works" but there are so many style opinions that issues keep coming. Who thinks it's fun to keep arguing about that? I haven't been contributing to prettier for a while now. @vjeux has been carrying the torch but will move on at some point. I'm sure we can figure out a community-centered way to manage prettier, but this brings up my main struggle with open source. When a project starts getting popular, it's hard to say no to small tweaks. But people request so many of them. It forces the project to become much more generalized, which makes software a lot more difficult to build. Coherence, performance, accessibility, maintainability, code size, testing, it all suffers when you have to generalize something, which isn't a bad thing, but it's *much* harder. Most projects do not need it. Most times a laser-focused project is a lot more powerful because you can focus on just a few edge cases, you can do things otherwise impossible for your specific project. Unfortunately I like building laser-focused projects that take my ideas as far as they can. This seems incompatible with open source. I've ended up writing my own libs and keeping them to myself. My own component interface, css-in-js, etc :/ It's working great though! It doesn't help that I'm contracting now, so literally whenever I choose to spend time on open source now I'm losing money. -- James Long [26 Jun 2017](https://twitter.com/jlongster/status/879331841769123841)

> Prettier was an interesting project where James Long open sourced it before it was used anywhere. Having a ton of people in open source using it early on side projects before it was solid enough to be widely deployed at Facebook really helped. -- Christopher Chedeau [9 June, 2018](https://twitter.com/Vjeux/status/1005602278034665472)

[@vjeux](https://github.com/vjeux) - While I believe that open source was a big part of prettier's success, I want to call out that Facebook, a company, was a critical reason why this project succeeded.

For 6 months my full time job was to work on prettier and Facebook paid James Long for a 2-week long contract (he wasn’t interested in working on it more), without this funding, the project wouldn’t have taken off.

The primary reason I worked on prettier was to get it to work on Facebook’s codebase. The fact that it worked outside of Facebook was a nice side effect but I would have done it without it. An historical artifact is that all the options early on were added so that it would adhere to Facebook style guide.
Facebook codebase was invaluable for this project as it helped find real-world examples of syntax in order to come up with a reasonable way to print it in various conditions, some rough numbers as to how people print various constructs in practice and a huge source of edge cases in order to harden the printer.

The fact that I worked at Facebook meant that I had direct access to people that built pretty printers for various languages including JavaScript in the past. Prettier itself was started as a fork of a project coming from Facebook. Without having all those people concentrated in one place, I would likely never have convinced myself to spend 6 months working on such a project.

*[@humphd](https://github.com/humphd) - So much is said these days about the sustainability of open source, and here you really see the need for a significant support system to be in place in order to sustain the work.  Facebook's role seems like a tremendous positive, and yet also points to some issues in reproducing this.  On the one hand it provides a framework for others to follow, namely, you need to be supported by a large ORG/COM to get beyond the initial “HackerNews” style release and into production.  But if you don't have that kind of support, you’re going to struggle.  A lot more people don’t work at Facebook than do, and can’t reproduce this result as you have.*

[@vjeux](https://github.com/vjeux) - I’ve mostly experienced open source within Facebook, so it has shaped the way I think about it. In our case, projects should first be useful for Facebook and then we spend 10-20% extra time to build a community around it to get all the benefits that people talk about.

I know that this setup has been proven to produce good results so I approached it the same way for prettier. It’s probably not the only way but it is certainly one way to go about it.

*[@humphd](https://github.com/humphd) - Thinking about the long tail of features is very useful.  People love to talk about the final 10% of the project taking 90% of the time, and that’s really what this is.  There are so many edge cases, I can’t imagine.  At this point, for a project like Prettier to work, it’s no longer enough for someone like Facebook to pay a single person: you need the wider community to get engaged, file bugs from their big code and real-world uses cases, do fixes, etc.  So to get this done, we can almost imagine a flow like this: 1 person starts something cool, add cheerleader(s) to prop that person up, build a project around them, at some point it needs to get support from an ORG/COM with $$$ to pay for dev time, and then you need support from outside that ORG, in order to solve the problems at a scale larger than one company can imagine.* 

[@vjeux](https://github.com/vjeux) - We [Facebook] have something we call the "prod infra playbook" that reads something like:

- First, we let engineers do whatever they want to fix their problems.
- Once we identify a common pattern, and only then, we try to find a solution for it in a bit of an R&D fashion.
- The goal of this original phase is to convince ourself that the idea has legs. Usually we try to identify all the things that could fail / hardest things and do small prototypes to see how this idea would work.
- Then we find a first customer. A single one. And work basically as a team with them to solve their problem along with building the tech.
- Then we add another customer, then another one.
- Usually when we have three successful ones, the tech is good enough to be used in more places so we let anyone start using it.
- We work at this point on trying to enboard as many customers as possible.
- Once it is good enough we bless it as the “right way” to do things and actively work on migrating existing code over to the new system.

It looks a bit similar to your list.

### Prettier: success bottom-up instead of top-down

*[@humphd](https://github.com/humphd) - There’s a part of your story that is implicit, and would be interesting to tease out: why does everyone suddenly care about pretty printers and standardized code formatting? I have some theories, but consider how long we’ve been working without such tooling (I’ve been programming JS since the late 1990s and never had it).  Then, all of a sudden, everyone is doing it, across all kinds of languages.  It would be interesting to consider what the tipping point was--gofmt?  Also, for something like prettier to work, you would imagine needing top-down approach.  Prettier seems to have accomplished what it did bottom-up, without a committee or standards process.  It seems to have happened via influence, the ability to project confidence, and network effects.  This tweet from James hints at it:*

> "Code formatting is largely a social problem and you aren't going to win unless it comes from somebody already deep inside the community who people trust will make good formatting decisions" --[James Long, 25 May, 2018](https://twitter.com/jlongster/status/1000206265124089857)

*you've also talked about this a bit on Twitter:*

> "I believe that prettier success is in part because its formatting is not tied to a specific IDE. If you are using Visual Studio but have contributors using Sublime and others using Emacs, they should all be able to have the same format." --[Christopher Chedeau, 24 May, 2018](https://twitter.com/Vjeux/status/999840949713879041)

[@vjeux](https://github.com/vjeux) - My biggest theory about this is the decentralization of language and tools. None of the pretty printers that came to market had any technological breakthrough. Prettier is based on [Wadler's paper from 1999](https://homepages.inf.ed.ac.uk/wadler/papers/prettier/prettier.pdf)!

But, until very recently, all the language tools were developed by the programming language creators or the language IDEs. And not the actual users of those languages. If you put yourself in their shoes, they want everyone to be able to use their product so it’s not in their best interest to make an opinionated printer that has the potential to alienate a lot of existing users.

Go is interesting in that it was designed to fix problems that Google engineers had, so they could push for more opinionated things.

In the JavaScript world, we’ve seen in the recent years a huge push towards tooling that manipulates the AST such as eslint, codemods, flow/typescript... so now if you want to enforce a code style, you could take the matter into your own hands and do it yourself.

If you look at the C++ world, you need to be inside of the compiler (clang-format) to be able to pull this off.

*[@humphd](https://github.com/humphd) - Thanks for doing this interview.  In my opinion it's no coincidence that you have been so successful with Prettier.  In a time where so much of the web is filled with hate, anger, ego, and tribalism, I’ve been impressed with your alternative approach to software, community, and open source.  I can’t think of too many more useful examples for my students to examine as they prepare to embark on their own careers, and for others looking at open source projects.*

[@vjeux](https://github.com/vjeux) - I’m super proud that we’ve been able to build a welcoming community with React. [I gave some context around how I tried to make it happen](https://youtu.be/cbNWm8gslOY?t=686) at the React Advanced meetup, you may be interested.
