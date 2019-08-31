---
title: A capstone course in information systems
---

## An education for problem solvers

In an undergraduate information systems degree program, we strive to prepare students to be valuable employees, effective managers, and capable entrepreneurs using information technology in business.  This is not to say that they should be exclusively focused on commerce.  Everyone who is trying to accomplish anything in the world---entrepreneurs, government, nonprofits, artists---has "business".  Being a part of the business school distinguishes information systems (MIS, CIS, whatever) from other programs like computer science, not because we have a different domain of applications, but because business students are trained to pay attention to problems that need solving, rather than "solutions looking for problems".  

We ought to teach students *how* to approach problem solving with IT, not *which* problems are worth solving.  Jeff Hammerbacher, one of the first data scientists at Facebook, provides us with an observation about wasted talent that I find sobering:

> "The best minds of my generation are thinking about how to make people click ads.  That sucks." ---[Jeff Hammerbacher](http://www.fastcompany.com/3008436/takeaway/why-data-god-jeffrey-hammerbacher-left-facebook-found-cloudera)

[![people-coffee-notes-tea.jpg](https://svbtleusercontent.com/ru62dqanpm8kfa_small.jpg)](https://svbtleusercontent.com/ru62dqanpm8kfa.jpg)

## Designing the capstone class

My responsibility is the capstone class, the final course that information systems majors take before they graduate.  By the time they reach me, these students have had a courses in programming and databases, networking and web development, and---guided by the above---my objective is to teach them how to put all of this knowledge together to develop real world solutions enabled by IT.

So how do I do this?  Well, when I began teaching the course, I was assigned a project management textbook, and the focus of the course were the capstone projects.  Each student team was developing a real IT system for a local business or organization.  The learning experience was primarily about the clash between the predictability of textbook assignments and the unpredictability of real-world projects, or as my predecessor Tim Olsen described it, ["the messiness of execution"](http://blogs.wpcarey.asu.edu/knowit/capstone-project-the-messiness-of-execution/):

>“We want students to experience the messiness of execution,” said clinical assistant professor Timothy Olsen, who teaches the capstone class. “When we teach students concepts in classes, most of the homework and test assignments are pretty clear-cut and there are fairly good directions. But in the real world, there are political problems and integration problems and learning curves, and lots of reasons why execution is difficult.”
> 
> The capstone project gives the students experience in dealing with the messiness of execution, when working with a client who might change his mind often, or dealing with incomplete requirements. “Having that sort of experience is one of the more valuable lessons that comes out of this project,” Olsen said.

Delivering this learning experience, however, challenged me as a professor in a number of ways.  First, there was the question of what I could teach in the classroom that would be relevant to what the students needed, when most of them were fully focused on their projects.  One of the perennial complaints students put on their course evaluations was "we don't need the lecture or readings; just finish up and let us have our team meeting".  Second was the issue of visibility; I couldn't be an active participant in twenty teams at the same time, so I typically didn't know whether projects would succeed or fail until the "big release" at the end of the semester.

To solve the second problem, I began to change the project management approach of the course from a traditional [SDLC](https://en.wikipedia.org/wiki/Systems_development_life_cycle) to an Agile method based on [Scrum](http://amzn.to/1LwrLcg).  Instead of a big release at the end of the semester, we would have frequent "demo days" of work-in-progress throughout the semester.  This has been incredibly successful; not only is Agile an important trend that our students need to know for the job market, it's also fantastic for pedagogy.  Teams learn not only from feedback on their own work, but from seeing the evolution and the trial-and-error that other teams are going through.

The other problem, though---that teams don't quite know what to do with my readings or lectures---seems to be perennial.  After 2 1/2 years teaching the course, I'm contemplating a major redesign.

![library.jpg](/assets/images/library.jpg)

## Changing context, changing course

There are two drivers for changing the course design, one imposed on me from outside, and one that's more of an evolutionary development from what I've learned through the past several iterations.

### An outside force

In 2014, the capstone course's designation as an "L" class (Literacy and Critical Inquiry) for general studies credit was due to expire, and if it was not renewed, all students in the major would have to take an extra class for this credit.  College credits aren't cheap, and no senior wants to be told he needs to stick around for one extra semester before his degree is final, so it was a no-brainer that we'd want to keep the designation.  However, doing so required that 50% of class credit be based on writing or an individual research project, and would take the focus of the course away from the group capstone projects.

Adding an individual "thesis" to the capstone course isn't a bad idea from a teaching standpoint, either.  We use alumni surveys to find out how well we've done at preparing them in three areas: critical thinking, communication skills, and domain-specific knowledge; for our department, the one we always get bad marks on is "communication skills".

### A paradigm shift

Outside of the college, also, information systems development is undergoing a major conceptual change.  In the **Agile** paradigm, developers learn to work with the business client (represented by a "product owner") to elicit requirements and feedback.  It was a great feature of the capstone course that students could get this real-world experience and learn the struggles of communication and coordination that it entailed.

What is changing in the workplace today, though, is that developers can no longer ignore what happens before they start developing---how does the product owner come up with those requirements?---and what happens after they finish---i.e., how does the company know the project was a success?  In the **Lean** paradigm, we can no longer find out what the business needs just by asking it.  As [Steve Blank says](http://steveblank.com/2009/10/08/get-out-of-my-building/):

> "There are no facts inside the building so get the heck outside."

Instead of asking client organizations what they want, I believe that students need to be talking to potential customers or users.  They need to learn how to validate prototypes, not by whether a product owner signs off on them at the end of a sprint, but by the use of real analytics: did users like the features, and use them in the way developers expected or hoped?  This is an entrepreneurial paradigm, but entrepreneurship is running through everything these days, and is a [core design principle of our university](https://live-newamericanuniversity.ws.asu.edu/about/design-aspirations).

![blankpage.jpg](/assets/images/blankpage.jpg)

## A new design

Over the past two semesters, I've increasingly added "validation" activity to the course requirements; for example, students are required to use rapid paper prototyping with their clients in the first milestone, and must conduct usability studies of their software projects with real users for a later milestone.  Additionally, this fall I added an individual term paper that requires students to conduct empirical research on their own.  

All this has been incremental and feels a little bit disconnected, so I've been doing a lot of blank-slate thinking lately.  Here are my thoughts.

- Two semester-long projects at the same time (a group project and a term paper) is a heavy cognitive burden on students.  Maybe the projects should be done in sequence, with intense schedules for half a semester each.

  - The next question is: which one first?

- Design is a science, for sure.  Can we also say that science is a design activity?  I wonder if the [design thinking](https://www.ideo.com/by-ideo/design-thinking-in-harvard-business-review) or [design science](https://en.wikipedia.org/wiki/Design_science_research) paradigm might be a good theme to unify both projects.  Thus, the group project would involve explicit identification of hypotheses, and the term papers would require students to use design science techniques like prototyping.

  - [Nigel Cross (1982)](http://www.sciencedirect.com/science/article/pii/0142694X82900400) has argued that design in education provides some of the educational value of "critical thinking", developing innate abilities "to understand the nature of ill-defined problems, how to tackle them, and how they differ from other kinds of problems".  If he's right, then design fits into my mandate to help students learn how to learn (as clich&eacute; as that sounds).
  
- A "flipped classroom" approach could be useful.  I could grade students pass/fail on whether they had watched the required videos or read the readings, and then facilitate meaningful project work (through a variety of  workshops) during class time.  That might be a way to ensure they learn the principles of design thinking, Agile, DevOps, while using face-to-face time in a way that aligns with their most pressing concerns.

- It might not be heretical to drop "client interaction" from the class, if I replace it with "customer interaction".  In other words, students would still grapple with the "messiness of execution" but the challenge wouldn't be pleasing one customer but instead pleasing many, and using analytics to prove it.  Students could identify their own projects, in this case.

- Many outside organizations, particularly companies who hire our graduates, still want to be involved.  I haven't quite figured out how they'd fit in if I designed "client work" out of the course.  Some kind of "mentorship" role would appeal to many of them.

The upcoming spring semester is my next good opportunity to experiment with the design of the course, so I'll be developing and validating some ideas over the next couple of months.  I welcome feedback and ideas!