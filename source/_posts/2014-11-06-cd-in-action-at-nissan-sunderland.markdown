---
layout: post
title: "CD in action at Nissan Sunderland"
date: 2014-11-06 13:54:14 +0100
comments: true
categories: agile lean automation nissan CD
author: Alex Butcher
---

The Nissan UK car plant at Sunderland produces almost as many cars every year as Italy’s entire automative industry. It’s a mammoth site.  Each of the four production lines in this factory is capable of producing one car every minute, continuously. As in 24x7x365. True continuous delivery into production. I was lucky enough to be invited on a factory tour by the good people at the [entrepreneurs forum](entrepreneursforum.net) recently. It was fascinating to see how Nissan have engineered every aspect of production to minimise waste and keep the line flowing. At Bede I've spent a lot of my time trying to enable agile teams to deliver high quality software into production as fast as possible, so after the factory tour my mind was left reeling with comparisons between Nissan and Bede's approaches to Continuous Delivery. 

{% img center /images/nissan-plant.jpg 'Aerial view of Nissan UK plant at Sunderland' 'Aerial view of Nissan UK plant at Sunderland' %}
*Source: [http://themanufacturer.com](http://www.themanufacturer.com/articles/nissan-smashes-uk-output-record-at-sunderland-plant/)*

<!-- more -->

I've visited a few factories in a previous life, but I've never visited a car production line: it's quite something to behold. Chassis are on a continuously moving conveyer, separated by a few feet, and move along the factory floor passing a succession of workstations.  The production conveyer moves at a constant speed - every work station has 60 seconds to perform its assigned tasks.  Some workstations have a single task to perform, others fit multiple parts to the chassis. Larger jobs are split up between multiple work stations.  Everything possible is done to ease the job of the fitter.  This is done by minimising the physical movements required, minimising risk of a line stoppage, reducing waste and maximising the amount of useful work that can be done in 60 seconds.

{% img center /images/nissan-line.jpg 'Nissan UK Qashqai production line' 'Nissan UK Qashqai production line' %}
*Source: [http://petrolblog.com/](http://petrolblog.com/2012/05/rising-sunderland-a-british-success-story/)*

Some rambling thoughts from the visit:

### Shared Vision

Nissan are big on shared vision. Our tour guides, both ex Nissan employees, bleed Nissan. It was clear that they understood Nissan's vision and were proud to be associated with them. Nissan work hard to install a sense of shared values in their staff. Information radiators are everyone throughput the plant - both static and display screens.  The primary metric on the shop floor is efficiency - which seems to be the number of cars rolling off the line compared to target.  On the day I visited the plant was running at 108% efficiency. A similar metric for Bede might be the number of successful production deployments achieved in a given time period.  Right now I'd estimate we push out 5-10 builds into production each week.

Despite employees showing a strong sense of Nissan pride, they know that efficiency relative to other plants will determine whether Sunderland receives future car models for manufacture, implying expansion and job security. This presents an interesting dichotomy - staff are passionate about Nissan as a brand, but have a local basis - they are in direct competition with other global Nissan car plants to ensure long term security.  Interestingly, Bede has 3 software development offices - Newcastle, London and Sofia.  We've worked hard to break down inter office rivalries, and have many product teams that span at least one office boundary.  In this sense, Bede's world of bits frees it from Nissan's physical limitations in terms of delivery across a distributed estate.

### Continuous, relentless delivery to production

Production workers understand that nothing can be allowed to stop the line. It's the shared responsibility of every employee to do whatever it takes to minimise downtime.  I heard that stoppages cost something like £10,000 per minute. This has two separate parallels in the software world.  Firstly, the production effort - keeping the build green, ensuring that only quality builds are deployed into environments. Through careful application of continuous integration, testing and sandbox deployments, especially with Bede's SOA, we strive to minimise the amount of time that the line is down (e.g. environments are broken). We've recently added an isolated automation testing environment at the head of our CI pipeline, to perform smoke tests on builds before automatically deploying into our first visible development environment. This increases the overall stability of the development environment at the cost of adding some time between commits being available for developer testing. We opted to make this trade off to focus on stability, since multiple product teams shares a common development environment.

{% img center /images/octopus-dashboard.png 'Continuous Integration Dashboard' 'Continuous Integration Dashboard' %}

Secondly, uptime in production - Bede manages the production environments for it's clients, so relentless monitoring of servers and applications is required to keep the status green and exceed stated SLA uptimes. Of course Nissan's relationship with their product continues long after delivery too, though a most shared approach for managing that client relationship exists in the form of dealerships.

{% img center /images/ops-nr-dashboard.jpg 'Environment Monitoring' 'Environment Monitoring' %}

### QA

Quality Assurance at the Nissan plant boils down to accountability.  As each chassis passes a workstation, the fitter stamps a card (yes, a physical piece of paper) with their own personalised stamp. In this way they're taking direct responsibility for the quality of the work they are performing. The card forms part of the permanent history of the car, it is never destroyed. As a result, fitters must take responsibility for their work.  Dedicated Quality Assurance staff do exist in the plant, but their job exists to catch unforeseen errors. I like the idea of producers being so closely linked to the produced artifacts. We already have the commit data for every release, but we don't publicise it internally at the moment, this is something I'm going to mull over...  I do think that as our usage of automation testing improves, so our testers are freed from having to spend the majority of their time writing automation suites, and can focus more on exploratory testing, like the Nissan QAs.

{% img center /images/bt-commiters.png 'Commit statistics for a Bede component' 'Commit statistics for a Bede component' %}

### Kaizen

Kaizen is a fancy word for continuous improvement. Key to this is accepting that process nirvana is an unachievable state, but that there is always something that can be done to move you closer to perfection. Suggestions for process improvement are grass roots as well as management driven. I think we can say the same for Bede - after working hard on Agile process, we are starting to find that team members initiate as much process change as managers. We've worked hard to create a [smart-fail](https://hbr.org/2013/01/to-increase-innovation-take-th/) culture, where employees feel confident to try things out.  Nissan don't offer incentives for improvements. An employee could suggest an improvement that saves the plant 1% of its running costs, but the staff member won't see a penny of that. At first I thought this was backward - why not incentivise staff to suggest improvements; but perhaps Nissan have nailed the culture so well that staff see it as their duty to continuously improve process.

### Leverage your bottlenecks

At one particular work centre, the chassis lifts up and passes overhead so that fitters can tighten up some bolts on the underside of the vehicle. The air tool used for this job is obviously very heavy, it used to be a difficult job and production managers complained of fatigue and minor injury rates at the workstation. Staff suggested building a semi-robotic arm, essentially taking the weight of the tool, but allowing the operator to position it as necessary. The plant's internal engineering team obliged, and the problem is now solved. This is a great example of leveraging the bottleneck at this workstation. For more on bottlenecks and constraints theory, see the reading recommendations at the end of this post. The obvious comparison in the world of bits is automation. Reducing repetition, with its accompanying opportunities for mistakes and time wastage.

### Staff Liquidity

Nissan has a dedicated training academy on site, in partnership with a local college.  The main goal of training is to get new employees up to the point that they can effectively perform at least 3 different workstation tasks to an acceptable level - staff are rotated frequently, even within shifts.  This kind of staff liquidity ensures that when there's a bout of flu going around, the production line doesn't grind to a halt - there are others trained to pick up the slack in any workstation. If things get desperate, line supervisors and managers are expected to get down to the shop floor and keep things running.

Liquidity is something that Bede software teams are still working to improve.  We've removed as many dependencies on specific individuals as possible, but there isn't yet perfect liquidity. Paul has blogged previously on this subject, in [Who is your brent](http://engineering.bedegaming.com/blog/2014/10/16/who-is-your-brent/). I think the KPI for staff liquidity is when we proactively move 'key' staff between teams. This has always been a long term aim - keeping engineers fresh by moving them around and exposing them to new problems, helping good ideas and practices to pollinate between teams, and further increasing liquidity.

Enjoyed this article? You should probably read [The Phoenix Project](http://www.amazon.co.uk/The-Phoenix-Project-Helping-Business-ebook/dp/B00AZRBLHO), or even better, go back to the source with [The Goal](http://www.amazon.co.uk/Goal-Process-Ongoing-Improvement-ebook/dp/B002LHRM2O/ref=sr_1_1?s=digital-text&ie=UTF8&qid=1415278766&sr=1-1&keywords=the+goal)






