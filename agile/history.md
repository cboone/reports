---
created: 2026-01-23
updated: 2026-01-23
---

# The complete history of agile: from Toyota to transformation

Agile methodology represents one of software development’s most significant paradigm shifts, yet its origins extend far beyond the famous 2001 Snowbird meeting. The movement synthesized decades of iterative development practices, Japanese manufacturing philosophy, and hard-won lessons from waterfall’s failures into a coherent alternative that has since reshaped how software gets built worldwide. **Over 95% of organizations now use agile practices**,  though debates persist about whether mainstream adoption has preserved or diluted its original values.

The intellectual DNA of agile traces through three distinct tributaries: the Toyota Production System’s emphasis on continuous improvement and respect for workers, early iterative development experiments at NASA and IBM in the 1960s-70s, and the mounting evidence that sequential “waterfall” development consistently produced failed projects. When 17 practitioners gathered in Utah’s Wasatch Mountains in February 2001, they crystallized these converging insights into four values and twelve principles that launched a revolution. 

-----

## Manufacturing philosophy crossed into software through lean thinking

The Toyota Production System (TPS), developed by **Taiichi Ohno** and **Shigeo Shingo** from the late 1940s through the 1960s, established foundational concepts that would later permeate agile thinking. TPS introduced just-in-time delivery, jidoka (stop-the-line quality control), kaizen (continuous improvement), and systematic waste elimination. Ohno’s insight that “all we are doing is looking at the time line, from the moment the customer gives us an order to the point when we collect the cash” anticipated agile’s focus on flow and customer value. 

The Toyota-GM NUMMI joint venture in 1983 exposed American manufacturers to TPS firsthand, but the concepts didn’t fully enter software vocabulary until James Womack and Daniel Jones coined “lean” in their landmark 1990 book *The Machine That Changed the World*.  **Mary and Tom Poppendieck** became the primary translators of lean principles to software with their 2003 book *Lean Software Development: An Agile Toolkit*,  which won the Software Development Productivity Award  and identified seven fundamental principles adapted for software teams:   eliminate waste, amplify learning, decide as late as possible, deliver as fast as possible, empower the team, build integrity in, and see the whole. 

Mary Poppendieck discovered the disconnect between lean thinking and typical software development after decades at 3M and Bell Labs, where she had built one of 3M’s first just-in-time production systems.   Ken Schwaber noted in the foreword that the Poppendiecks gave agile “something truly solid upon which to hang their collective hats.”

-----

## Iterative development predated agile by decades

Long before the Agile Manifesto, practitioners had discovered that building software incrementally worked better than sequential phases. NASA’s Project Mercury (1958-early 1960s) used **half-day time-boxed iterations with test-first development**—planning and writing tests before each micro-increment.  Gerald Weinberg, who worked on Mercury, recalled: “We were doing incremental development as early as 1957… All of us, as far as I can remember, thought waterfalling of a huge project was rather stupid.”

IBM’s Federal Systems Division, seeded by Mercury personnel,  became a major proponent of iterative development. The USS Trident submarine command-and-control system (1972) used **four six-month iterations** for over one million lines of life-critical code—driven partly by $100,000/day late penalties. The Light Airborne Multipurpose System project in the mid-1970s employed **45 one-month iterations**, each delivered on time and under budget according to Harlan Mills.

**Tom Gilb** deserves recognition as one of the earliest persistent advocates for iterative approaches. His 1976 book *Software Metrics* first introduced “evolution” and “evolutionary” terminology to software processes, and his 1988 *Principles of Software Engineering Management* became the first book with substantial chapters dedicated to iterative and incremental development. Kent Beck and Jim Highsmith both acknowledged Gilb as an inspiration for their later work.

**Barry Boehm’s spiral model** (1986) introduced risk-driven iteration, combining prototyping with structured development phases.  Each spiral cycle determined objectives, identified and resolved risks, developed and tested solutions, then planned the next iteration. This approach influenced numerous large-scale projects including the Space Shuttle and various Department of Defense systems.

-----

## Waterfall became dominant despite its creator’s warnings

The irony of waterfall’s dominance lies in how thoroughly its original description was misread. **Winston Royce’s 1970 paper** “Managing the Development of Large Software Systems” presented the sequential model but explicitly warned: “I believe in this concept, but the implementation described above is risky and invites failure.” Royce actually recommended building the system twice, with a preliminary version preceding the delivered product, and incorporating feedback loops between phases.  His son Walker later confirmed: “He was always a proponent of iterative, incremental, evolutionary development.” 

Yet waterfall became entrenched because it offered apparent orderliness. Simple to explain and recall, it promised document-driven milestones that satisfied managers seeking accountability. Textbooks, courses, and consulting organizations promoted it, and the U.S. Department of Defense’s DoD-STD-2167 (1985) essentially mandated it for defense contracts.

The results proved disastrous. A 1999 review of DoD projects revealed **75% failed or were never used**, with only 2% deployed without extensive modification—representing approximately $37 billion in failed spending. The 1987 Defense Science Board Task Force, chaired by Fred Brooks, recommended radical overhaul: “DoD-Std-2167 continues to reinforce exactly the document-driven, specify-then-build approach that lies at the heart of so many DoD software problems.” Brooks declared at the 1995 International Conference on Software Engineering: “The waterfall model is wrong!” 

-----

## Seventeen practitioners gathered at Snowbird and changed everything

By early 2001, multiple lightweight methodologies had emerged independently—Extreme Programming, Scrum, Crystal, Adaptive Software Development, DSDM, Feature-Driven Development—each tackling waterfall’s failures differently.  **Robert C. Martin** (“Uncle Bob”) initiated a gathering with a September 2000 email seeking to convene “all the lightweight method leaders in one room.” 

The meeting occurred **February 11-13, 2001** at The Lodge at Snowbird ski resort in Utah’s Wasatch Mountains.  The seventeen signatories represented diverse approaches:

- **Extreme Programming**: Kent Beck, Ward Cunningham, Ron Jeffries
- **Scrum**: Ken Schwaber, Jeff Sutherland, Mike Beedle
- **Crystal**: Alistair Cockburn
- **Adaptive Software Development**: Jim Highsmith
- **DSDM**: Arie van Bennekum
- **Pragmatic Programming**: Dave Thomas, Andy Hunt
- **Object-Oriented/Patterns**: Robert C. Martin, Martin Fowler, Jon Kern, Steve Mellor, Brian Marick, James Grenning

The morning sessions featured presentations in the Aspen Room; afternoons allowed skiing; evenings continued discussions over dinner.  The fiercest debate concerned the name—“lightweight” was universally despised. Cockburn objected: “It somehow sounds like a bunch of skinny, feebleminded lightweight people trying to remember what day it is.”  **Mike Beedle proposed “agile,”** inspired by a book about lean manufacturing.  Martin Fowler’s only concern was that “most Americans didn’t know how to pronounce the word.” 

The four values were wordsmithed to **100% unanimity**:  individuals and interactions over processes and tools; working software over comprehensive documentation; customer collaboration over contract negotiation; responding to change over following a plan.  The twelve principles were refined via email and published in April 2001.  Ward Cunningham set up the manifesto website, and the **Agile Alliance** formed later that year as a nonprofit  with Martin as its first chairman.

-----

## The Chrysler C3 project gave birth to Extreme Programming

The Chrysler Comprehensive Compensation System (C3) became the crucible where Extreme Programming practices fused into a coherent methodology.  Initiated in 1993 to replace multiple legacy COBOL payroll systems with a single Smalltalk/GemStone application  for 87,000 employees, the project struggled for years before **Kent Beck** joined in March 1996—initially for performance tuning—and became project leader. 

Beck brought in **Ron Jeffries** as the first XP coach, with **Ward Cunningham** providing major intellectual influence.  The team formally adopted practices that would define XP: **pair programming, test-driven development, continuous integration, refactoring, simple design, collective code ownership, sustainable pace (40-hour weeks), on-site customer, the planning game, and small releases**. Reportedly, 75% of the codebase was thrown out during refactoring sessions.

C3 went live in 1997, paying approximately 10,000 salaried employees.  However, after Daimler-Benz acquired Chrysler in 1998, the on-site customer—a role critical to XP—resigned from burnout and couldn’t be replaced.  Development stopped in 1999,  and DaimlerChrysler officially cancelled C3 in February 2000  before it expanded beyond the initial population.

Martin Fowler noted: “C3’s cancellation also proves that XP is no guarantee of success.”  Yet by that point, enough other projects had replicated C3’s early successes to establish XP’s viability. Beck published *Extreme Programming Explained: Embrace Change* in October 1999,  and the methodology spread independently of its birthplace.

-----

## Scrum emerged from rugby metaphors and empirical process control

**Jeff Sutherland** created Scrum in 1993   at Easel Corporation,  inspired by Takeuchi and Nonaka’s 1986 Harvard Business Review paper “The New New Product Development Game,”  which used the rugby “scrum” metaphor for cross-functional teams moving together. Working with John Scumniotales and Jeff McKenna, Sutherland built the first Scrum implementation— later rated one of only two teams to score “10” on his hyperproductivity scale.

**Ken Schwaber** independently developed similar ideas, learning from a chemical process control expert at DuPont about distinguishing deterministic from experimental processes.  Schwaber and Sutherland unified their frameworks  and presented Scrum publicly at **OOPSLA 1995** in Austin, Texas—the first formal introduction of the methodology to the worldwide software community.  

Scrum defined three roles (Product Owner, Scrum Master, Development Team),  three artifacts (Product Backlog, Sprint Backlog, Increment), and ceremonies including Sprint Planning, Daily Scrum, Sprint Review, and Sprint Retrospective. Time-boxed sprints of one to four weeks created predictable delivery rhythm.

The organizational history grew complex. Schwaber co-founded the **Scrum Alliance** in 2002 with Mike Cohn and Esther Derby, creating the Certified ScrumMaster program. After philosophical disagreements, Schwaber left in 2009 to found **Scrum.org**, offering competency-based assessments rather than training-based certifications.  Sutherland runs **Scrum Inc.** The first official **Scrum Guide** appeared in 2010,  with major updates in 2016 (introducing Scrum Values)  and 2020 (simplifying language, adding Product Goal).  

-----

## Kanban adapted Toyota’s visual signals for knowledge work

**David Anderson** pioneered Kanban for software development during his time at Microsoft and later Corbis. At Microsoft’s XIT Maintenance Engineering team  in 2004-2005, he experimented with visual workflow management and work-in-progress limits. The full Kanban Method emerged at Corbis in 2006-2007, yielding **240% improvement in delivery rates and 90% reduction in delivery times**.

Anderson’s approach drew from Toyota’s production Kanban (Japanese for “signboard”), Eli Goldratt’s Theory of Constraints, and Don Reinertsen’s product development flow principles. Rather than imposing wholesale change, Kanban offered evolutionary improvement: start with what you do now, visualize the work, limit work in progress, manage flow, make policies explicit, and improve collaboratively.

His 2010 book *Kanban: Successful Evolutionary Change for Your Technology Business*  became one of the top five agile books ever published. Unlike Scrum’s prescribed roles and fixed sprints, Kanban has no mandatory roles and enables continuous flow—making it particularly suited for maintenance, operations, and teams handling unpredictable work.

-----

## The foundational books established agile’s intellectual canon

The year 1999 proved pivotal for agile literature. **Kent Beck’s** *Extreme Programming Explained: Embrace Change* defined XP’s practices and philosophy,   later revised in 2004 to add “respect” as a fifth value and reorganize practices based on accumulated experience.  Reviewers called it “dynamite” for changing industry thinking.

**Dave Thomas and Andy Hunt’s** *The Pragmatic Programmer: From Journeyman to Master* ran parallel to XP, introducing memorable concepts like “tracer bullets” and “broken windows.” **Martin Fowler’s** *Refactoring: Improving the Design of Existing Code* made code improvement a mainstream practice, providing a catalog of 70+ refactorings with step-by-step instructions.

The post-Manifesto period consolidated knowledge. **Ken Schwaber and Mike Beedle’s** *Agile Software Development with Scrum* (2001) introduced Scrum comprehensively. **Mike Cohn’s** *User Stories Applied* (2004) standardized the user story format,   while his *Agile Estimating and Planning* (2005) became the definitive guide to story points and velocity.

**Robert C. Martin’s** *Clean Code* (2008) defined code quality standards for agile teams,  followed by *The Clean Coder* (2011) addressing professional ethics. **Jez Humble and David Farley’s** *Continuous Delivery* (2010) won the Jolt Excellence Award and introduced deployment pipelines.  **Gene Kim’s** *The Phoenix Project* (2013) brought DevOps principles to wider audiences through narrative fiction. **Jeff Sutherland’s** *Scrum: The Art of Doing Twice the Work in Half the Time* (2014) provided a first-hand account aimed at general business readers.  

-----

## Conferences created community and spread practices globally

**XP2000** in Cagliari, Sardinia marked the first major agile conference—expected to draw 60 attendees, it attracted over 160. Kent Beck, Ron Jeffries, Robert Martin, and Ward Cunningham spoke to an audience “thick with current and ex Smalltalkers, now mostly in Java.” Ron Jeffries presented a complete C3 retrospective covering both successes and factors leading to cancellation.

The XP conference series continued annually, growing to become the largest agile conference outside North America. XP/Agile Universe 2002 in Chicago combined the second XP Universe with the first Agile Universe, featuring dynamic discussions and 41 peer-reviewed papers. **XP2024** celebrated the 25th anniversary in Bolzano, Italy, with XP2025 planned for Switzerland.

The **Agile Alliance** has organized annual conferences since its founding, bringing together practitioners and researchers for papers, experience reports, and workshops.  **OOPSLA** (Object-Oriented Programming, Systems, Languages, and Applications) served as an important early venue—it hosted Schwaber and Sutherland’s 1995 Scrum presentation and countless subsequent agile discussions.

-----

## Tools enabled and emerged from agile practices

**Ward Cunningham’s Wiki** (WikiWikiWeb), installed on c2.com on March 25, 1995,  enabled the organic knowledge sharing that spread XP concepts. Cunningham described wiki and agile as “two sides of the same coin”—both emphasizing collaboration and organic growth.  He later created Framework for Integrated Tests (FIT) in 2002 for executable acceptance testing. 

**JUnit** was created by Kent Beck and Erich Gamma on a flight to OOPSLA 1997.   Beck saw it as “a political statement—as a programmer I accept responsibility for the quality of my work.”  JUnit spawned the xUnit family across languages  and made test-driven development practical.

Version control evolved from CVS (1986) through Subversion to **Git** (2005), created by Linus Torvalds in response to a BitKeeper licensing dispute.  Torvalds began coding on April 3, 2005,  and released Git 1.0 on December 21, 2005. **GitHub** (2008) transformed Git from command-line tool to global collaboration platform;  **95% of developers** now use Git. 

Continuous integration tools progressed from CruiseControl through Hudson (2005) to **Jenkins** (forked in 2011 after an Oracle trademark dispute),  which now has 1,500+ plugins and dominates the CI/CD landscape. **Travis CI** and **CircleCI** (both 2011) added cloud-based options.

**Jira** launched in 2002 from Atlassian, originally as bug-tracking software.  After acquiring GreenHopper in 2009, Jira gained agile boards, sprints, and burndown charts,  becoming the de facto standard for agile project management—**58-66% of companies** now use it. Pivotal Tracker (2008) served XP-focused teams until Broadcom shut it down in April 2025. 

-----

## Companies pioneered and championed agile adoption

**ThoughtWorks**, founded in 1993, embedded agile DNA deeply into its corporate culture  after Martin Fowler joined as Chief Scientist in 2000. Fowler co-authored the Agile Manifesto  and hosted martinfowler.com, which became one of the most influential resources on software development practices. ThoughtWorks distinguished itself by tackling problems other firms considered impossible— particularly distributed agile development. 

**Pivotal Labs** (founded 1989 by Rob Mee)  became synonymous with disciplined XP adoption: eight hours of daily pair programming,  strict test-driven development, and collaborative product design. After acquisitions by EMC (2009), VMware (2019), and Broadcom (2023), Tanzu Labs was shut down in January 2025— a significant loss for the XP community. Rob Mee founded Mechanical Orchard in 2022 to continue similar practices. 

**Atlassian** (founded 2002 in Sydney) grew from Jira into a comprehensive tooling ecosystem including Confluence and Trello, recognized in 2025 as a Leader in Gartner’s Magic Quadrant for Collaborative Work Management. 

The **Spotify Model**, documented in a 2012 whitepaper by Henrik Kniberg and Anders Ivarsson, organized work around autonomous squads (6-12 people), tribes (collections of squads), chapters (same-skill people across squads), and guilds (voluntary communities of interest). Critically, Kniberg stated it was “just an example”—never intended as a framework to copy.  Spotify itself has largely moved away from the model, yet it became widely imitated with mixed results.

-----

## DevOps extended agile principles to operations

**Patrick Debois**, frustrated by development/operations disconnect during a 2007 Belgian data center migration, became the catalyst for the DevOps movement.  In August 2008, he attended an “Agile Infrastructure” session at the Toronto Agile Conference—as the only attendee—and met Andrew Shafer, forming the Agile System Administration Group. 

The pivotal moment came in June 2009 when John Allspaw and Paul Hammond presented “10+ Deploys a Day: Dev and Ops Cooperation at Flickr”  at O’Reilly’s Velocity Conference. Watching remotely, Debois organized the first **DevOpsDays** in Ghent, Belgium in October 2009.  Needing a Twitter hashtag, he combined “Dev” and “Ops” and “Days”—the #DevOps hashtag named the movement. Debois later admitted: “There never was a grand plan for DevOps as a word.” 

DevOps evolved naturally from agile, sharing values of collaboration, flexibility, and continuous improvement while extending them to operations teams.  Key concepts include CI/CD pipelines, infrastructure as code (Puppet, Chef, Ansible), and Site Reliability Engineering (Google hired Ben Treynor in 2003 to lead this practice). **Jez Humble’s** work on continuous delivery bridged agile development with operational deployment,  and his later research at DORA (acquired by Google in 2018) established empirical links between technical practices and organizational performance. 

-----

## Scaled frameworks addressed enterprise complexity

As organizations sought to apply agile at scale, multiple frameworks emerged:

**SAFe (Scaled Agile Framework)**, created by Dean Leffingwell and first released in 2011,   became the most commercially successful scaling approach. Built on foundations from Leffingwell’s books *Scaling Software Agility* (2007) and *Agile Software Requirements* (2010),  SAFe organized work into Agile Release Trains at the program level, with team, solution, and portfolio layers above. Version 6.0 (2023) emphasizes flow-based principles and business agility. 

**LeSS (Large-Scale Scrum)**, created by Craig Larman and Bas Vodde  at Nokia Siemens Networks in 2005,  took the opposite approach—maintaining Scrum’s simplicity at scale rather than adding layers. Their philosophy of “descaling” emphasizes that large-scale Scrum IS Scrum, with basic LeSS supporting 2-8 teams and LeSS Huge organizing 8+ teams into requirement areas.

**Nexus**, released by Ken Schwaber in 2015 through Scrum.org, provides a lightweight framework for 3-9 Scrum teams working on a single product backlog. The Nexus Integration Team coordinates across teams with formalized cross-team refinement and retrospectives.

-----

## Crystal, FDD, DSDM, and other methodologies contributed distinct perspectives

**Crystal** (Alistair Cockburn, 1991-1994)  rejected one-size-fits-all thinking, offering methodology variants scaled by team size: Crystal Clear for up to 8 people, Yellow for 10-20, Orange for 20-50,  and heavier weights for larger or safety-critical systems. Cockburn’s core properties—frequent delivery, reflective improvement, osmotic communication, personal safety, focus, expert user access, and proper technical environment—informed the Manifesto discussions.

**Feature-Driven Development** (Jeff De Luca and Peter Coad, 1997) emerged from a 50-person Singapore bank project, organizing work around features with individual class ownership (controversial versus XP’s collective ownership) and color-coded modeling. David Anderson, later the Kanban pioneer, participated in the original FDD team.

**Dynamic Systems Development Method** (1994, UK) grew from RAD practitioners including representatives from British Airways, American Express, and Oracle. Its eight principles and techniques including MoSCoW prioritization influenced many subsequent approaches. Arie van Bennekum represented DSDM at Snowbird.

**Adaptive Software Development** (Jim Highsmith, early 1990s) emphasized speculate-collaborate-learn cycles for complex adaptive systems, winning the 2000 Jolt Award. Highsmith became the “official historian” of the Agile Manifesto and later co-led the PM Declaration of Interdependence (2005) extending agile thinking to project management. 

-----

## Modern practice confronts criticism and evolution

By the 2020s, agile had achieved mainstream dominance—**71% of organizations** use agile in software development,   with 97% reporting some agile adoption. Yet success brought concerns about dilution and commercialization.

**Martin Fowler** identified the “Agile Industrial Complex” as a key problem  in his 2018 keynote, criticizing the imposition of processes on teams rather than letting teams choose their own approaches.  **Ron Jeffries** coined “Dark Scrum”  and “Flaccid Scrum” to describe implementations that use agile’s name while abandoning its principles— particularly technical practices like TDD, refactoring, and pair programming that many teams skip.

Jeffries’ provocative 2018 article “Developers Should Abandon Agile” argued that programmers should focus on technical excellence regardless of organizational methodology:  “Dark Scrum is actively using the name ‘agile’ against the basic principles.”   The certification industry—what some call “Certified Vanity Planes”—drew particular criticism for prioritizing revenue over genuine capability building.

Post-agile thinking emphasizes outcomes over process compliance, moving from “doing agile” to “being agile.” The Modern Agile movement advocates “ultra-light” approaches opposite to mainstream agile bureaucracy. Remote work since 2020 has both challenged (losing co-located collaboration) and enabled (video conferencing, digital boards) agile practices.

Current trends include AI-powered transformation tools, expansion beyond IT (35% of non-IT departments have adopted agile),  product-oriented team organization, and value stream focus. The enterprise agile transformation market reached **$48.75 billion in 2025**, projected to grow to $96.28 billion by 2029. 

-----

## Comprehensive timeline from origins to present

|Period         |Key Events                                                                               |
|---------------|-----------------------------------------------------------------------------------------|
|**1940s-1960s**|Taiichi Ohno develops Toyota Production System; Shewhart/Deming PDSA cycles              |
|**1958-1963**  |NASA Project Mercury uses half-day iterations with test-first development                |
|**1968**       |NATO conference coins “software crisis”; Randell/Zurcher IID paper                       |
|**1970**       |Winston Royce’s paper (misinterpreted as waterfall advocacy)                             |
|**1972-1977**  |Major IID projects: Trident submarine, LAMPS (45 one-month iterations), Space Shuttle    |
|**1975-1976**  |Basili/Turner “Iterative Enhancement” paper; Gilb’s *Software Metrics*                   |
|**1986**       |Takeuchi/Nonaka “rugby” paper; Boehm’s spiral model; CVS created                         |
|**1988**       |Gilb’s *Principles of Software Engineering Management*; Ohno’s *TPS* in English          |
|**1989**       |Rob Mee founds Pivotal Labs                                                              |
|**1990**       |*The Machine That Changed the World* coins “lean”                                        |
|**1991**       |James Martin publishes *Rapid Application Development*; Crystal begins                   |
|**1993**       |First Scrum at Easel Corporation; ThoughtWorks founded; C3 initiated                     |
|**1994**       |DSDM Consortium founded in UK                                                            |
|**1995**       |WikiWikiWeb launched (March 25); Scrum presented at OOPSLA                               |
|**1996**       |Kent Beck becomes C3 project leader                                                      |
|**1997**       |FDD created; C3 goes live; JUnit created on flight to OOPSLA                             |
|**1999**       |*XP Explained*, *Pragmatic Programmer*, *Refactoring* published; C3 development stops    |
|**2000**       |XP2000 conference (first major agile conference); C3 cancelled; Fowler joins ThoughtWorks|
|**2001**       |**Agile Manifesto signed (February 11-13, Snowbird)**; Agile Alliance formed             |
|**2002**       |Atlassian founded; Jira launched; Scrum Alliance created                                 |
|**2003**       |Poppendiecks publish *Lean Software Development*; first Agile Alliance conference        |
|**2004**       |Kanban experiments at Microsoft; *User Stories Applied* published                        |
|**2005**       |LeSS created; Git created (April); Hudson development begins                             |
|**2006-2007**  |Kanban Method emerges at Corbis                                                          |
|**2008**       |GitHub launched; *Clean Code* published; Pivotal Tracker released                        |
|**2009**       |First DevOpsDays, “DevOps” coined; Schwaber founds Scrum.org                             |
|**2010**       |*Continuous Delivery* published; first Scrum Guide; Kanban book published                |
|**2011**       |SAFe 1.0; Jenkins forked; Travis CI and CircleCI founded                                 |
|**2012**       |Spotify Model whitepaper published                                                       |
|**2013**       |*The Phoenix Project* published                                                          |
|**2014**       |LeSS Company founded; *Scrum: The Art of Doing Twice the Work* published                 |
|**2015**       |Nexus framework released                                                                 |
|**2016**       |Scrum Guide adds Scrum Values                                                            |
|**2018**       |Microsoft acquires GitHub; Fowler keynote on “Agile Industrial Complex”                  |
|**2019**       |*Clean Agile* published; VMware acquires Pivotal                                         |
|**2020**       |Major Scrum Guide update; COVID accelerates remote agile adoption                        |
|**2023**       |SAFe 6.0; Broadcom acquires VMware                                                       |
|**2025**       |Tanzu Labs shut down; Pivotal Tracker retired; market reaches $48.75B                    |

-----

## The arc of agile continues to bend

The history of agile methodology illuminates a recurring pattern: practitioners discovering through painful experience that software development resists industrial-era management approaches. From NASA engineers using half-day iterations in 1958 to today’s debates about Dark Scrum and post-agile thinking, the core tension persists between treating software development as a manufacturing process versus recognizing it as a learning and communication challenge.

The Agile Manifesto’s enduring power lies not in prescribing specific practices but in establishing values that guide adaptation. Kent Beck, Ward Cunningham, and their co-signatories didn’t invent iteration or customer collaboration—they synthesized decades of accumulated wisdom into principles that spread virally because they named what experienced practitioners already knew worked.

What began as a ski lodge conversation among 17 people has transformed into a global industry with its own conferences, certifications, and (inevitably) pathologies. Whether mainstream agile preserves or betrays its founders’ intentions remains contested. But the underlying insight—that complex, creative work requires different approaches than repetitive production—continues shaping how software gets built, and increasingly, how organizations of all kinds manage uncertainty and change.