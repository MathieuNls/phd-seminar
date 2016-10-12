---
title: "BIANCA: Preventing Bug Insertion at Commit-Time Using Dependency Analysis and Clone Detection"
bibliography: config/library.bib
abstract:  Preventing the introduction of software defects at commit-time is a growing line of research in the software maintenance community. Existing approaches leverage code and process metrics to build statistical models that can effectively prevent defect insertion and propose fixes in a software project. Metrics, however, may vary from one project to another, hindering the reuse of these models. Moreover, these techniques operate within single projects only despite the fact that many projects share dependencies and are, therefore, vulnerable to similar faults. In this paper, we propose a novel approach, called BIANCA, that relies on clone detection and dependency analysis to detect _risky_ commits within and across related projects. When applied to 42 projects, BIANCA achieves an average precision, recall and F-measure of 90.75%, 37.15% and 52.72%, respectively. We also found that only 8.6% of the risky commits detected by BIANCA match other commits from the same project, suggesting that relationships across projects need to be considered for effective prevention of risky commits. In addition, BIANCA is able to propose qualitative fixes to transform _risky_ commits into _non-risky_ ones in 78.67% of the cases. 

author: 
- name: Mathieu Nayrolles,  Abdelwahab Hamou-Lhadj
  affiliation: SBA Lab, ECE Dept, Concordia University
  location: Montréal, QC, Canada
  email: \{mathieu.nayrolles, wahab.hamou-lhadj\}\@concordia.ca
- name: Emad Shihab
  affiliation: DAS Lab, CSE Dept, Concordia University
  location: Montréal, QC, Canada
  email: eshihab@cse.concordia.ca
csl: config/ieee.csl
classoption: conference
keyword: 
- Bug Prediction
- Risky Software Commits
- Clone Detection
- Software Maintenance

---

Introduction
============

Software maintenance activities such as debugging and feature
enhancement are known to be challenging and costly [@Pressman2005].
Studies have shown that the cost of software maintenance can reach up to
70% of the overall cost of the software development life cycle
[@HealthSocial2002]. Much of this is attributable to several factors
including the increase in software complexity, the lack of traceability
between the various artefacts of the software development process, the
lack of proper documentation, and the unavailability of the original
developers of the systems.

Research in software maintenance has evolved over the years to include
areas like mining bug repositories, bug analysis, prevention and
reproduction. The ultimate goal is to develop techniques and tools to
help software developers detect, correct, and prevent bugs in an
effective and efficient manner. Despite the recent advances in the
field, the literature shows that many existing software maintenance
tools have yet to be adopted by industry
[@Lewis2013; @Foss2015; @Layman2007; @Ayewah2007; @Ayewah2008; @Johnson2013; @Norman2013; @Hovemeyer2004; @Lopez2011].
We believe that this is caused by the following factors:

-   Integration with the developer’s workflow: Most existing maintenance
    tools
    ([@Kim2006a; @Ayewah2008b; @Findbugs2015; @Moha2010; @Palma; @Nayrolles2013d; @Nayrolles; @Nayrolles2013a; @Nayrolles2015a]
    are some noticeable examples) are not integrated well with the work
    flow of software developers (i.e., coding, testing,
    debugging, committing). Using these tools, developers have to
    download, install and understand them to achieve a given task. They
    would constantly need to switch from one workspace to another for
    different tasks (i.e., feature location with a command line tool,
    development and testing code with an IDE, development and testing
    front end code with another IDE and a browser,
    etc.)[@Robertson2004; @Robertson2006; @Beckwith2006].

-   Corrective actions: The outcome of these tools does not always lead
    to corrective actions that the developers can implement. Most of
    these tools return several results that are often difficult to
    interpret by developers. Take for example, FindBugs
    [@Hovemeyer2004], a popular bug detection tool. This tool detects
    hundreds of bug signatures and reports them using an abbreviated
    code such as <span>CO\_COMPARETO\_INCORRECT\_FLOATING</span>. Using
    this code, developers can browse the FindBug’s dictionary and find
    the corresponding definition <span>*“This method compares double or
    float values using pattern like this:
    $val1 > val2~?~1 : val1 < val2~?~-1 : 0$”*</span>. While the
    detection of this bug pattern is accurate, the tool does not propose
    any corrective actions to the developers that can help them fix
    the problem. Moreover, it has been reported in the literature that
    the output of existing maintenance tools tends to be verbose at the
    point where developers decide to simply ignore them
    [@Arai2014; @Kim2007b; @Kim2007c; @Ayewah2010; @Shen2011].

-   Leverage of historical data: These tools do not leverage a large
    body of knowledge that already exists in open source systems. For
    defect prevention, foe example, the state of the art approaches
    consists of adapting statistical models built for one project to
    another project [@Lo2013; @Nam2013]. As argued by Lewis
    <span>*et al.*</span> [@Lewis2013] and Johnson <span>*et al.*</span>
    [@Johnson2013], approaches based solely on statistical models are
    perceived by developers as black box solutions. Developers are less
    likely to trust the output of these tools.

In this thesis, we propose to address some of the above-mentioned issues
by focusing on developing techniques and tools that support software
maintainers at commit-time. As part of the developer’s work flow, a
commit marks the end of a given task or subtask as the developer is
ready to version the source code. We propose a set of approaches in
which we intercept the commits and analyse them with the objective of
preventing unwanted modifications to the system. By doing so, we do not
only propose solutions that integrate well with the developer’s work
flow, but also there is no need for software developers to use any other
external tools. As we will show in the rest of this proposal, some of
the techniques we propose rely on best practices found in a a large
repository of open source systems. In other words, we aim to leverage
historical data to guide new development efforts. We refer to the field
of study that encompasses software analysis techniques that operate on
code commits as software maintenance at commit-time.

More precisely, we propose the following contributions that we present
here and discuss in more detail in the next section:

-   An aggregated bug repository system.

-   A clone prevention technique at commit-time.

-   A bug prevention technique at commit-time

-   A bug reproduction technique based on directed model checking and
    crash traces

-   A new classification of bugs based on the locations of
    the corrections.

## Research Contributions {#sec:objective-thesis}

### An aggregate bug repository for developers and researchers

When facing a new bug, one might want to leverage decades of open source
software history to find a suitable solution. The chances are that a
similar bug has already been fixed somewhere in another open source
project. The problem is that each open source project hosts its data in
a different data repository, using different bug tracking and version
control systems. Moreover, these systems have different interfaces to
access data. The data is not represented in a uniform way either. This
is further complicated by the fact that bug tracking tools and version
control systems are not necessarily connected. The former follows the
life of the bug, while the latter manages the fixes. As a result, one
would have to search the version control system repository to find
candidate solutions. Moreover, developers mainly use classical search
engines that index specialized sites such as StackOverflow. These sites
are organized in the form of question-response where a developer submits
a problem and receives answers from the community. While the answers are
often accurate and precise, they do not leverage the history of open
source software that has been shown to provide useful insights to help
with many maintenance activities such as bug fixing [@Saha2014], bug
reproduction [@Nayrolles2015], fault analysis [@Nessa2008], etc.

In this work, we introduce BUMPER (BUg Metarepository for dEvelopers and
Researchers), a web-based infrastructure that can be used by software
developers and researchers to access data from diverse repositories
using natural language queries in a transparent manner, regardless of
where the data was originally created and hosted. The idea behind BUMPER
is that it can connect to any bug tracking and version control systems
and download the data into a single database. We created a common schema
that represents data, stored in various bug tracking and version control
systems. BUMPER uses a web-based interface to allow users to search the
aggregated database by expressing queries through a single point of
access. This way, users can focus on the analysis itself and not on the
way the data is represented or located. BUMPER supports many features
including: (1) the ability to use multiple bug tracking and control
version systems, (2) the ability to search very efficiently large data
repositories using both natural language and a specialized query
language, (3) the mapping between the bug reports and the fixes, and (4)
the ability to export the search results in Json, CSV and XML formats.

### An incremental approach for preventing bug and clone insertion at commit time

Code clones appear when developers reuse code with little to no
modification to the original code. Studies have shown that clones can
account for about 7% to 50% of code in a given software system
[@Baker; @StephaneDucasse]. Developers often reuse code (and create
clones) in their software on purpose [@Kim2005]. Nevertheless, clones
are considered a bad practice in software development since they can
introduce new bugs in the code [@Kapser2006; @Juergens2009; @Li2006]. If
a bug is discovered in one segment of the code that has been copied and
pasted several times, then the developers will have to remember the
places where this segment has been reused in order to fix the bug in
each place. In the last two decades, there have been many studies and
tools that aim at detecting clones. They can be grouped into three
categories. Although these techniques and tools have been shown to be
useful in detecting clones, they operate in an off-line fashion (i.e.,
after the clones have been inserted). Software developers might be
reluctant to use these tools on a day-today basis (i.e., as part of the
continuous development process), unless they are involved in a major
refactoring effort. This problem is somehow similar to the problem of
adopting bug identification tools. Johnson et al. [@Johnson2013] showed
that these tools are challenging to use because they do not integrate
well with the day-to-day work flow of a developer. Also they output a
large amount of data when applied to the entire system, making it hard
to understand and analyse their results.

In this research, we present PRECINCT (PREventing Clones INsertion at
Commit Time) that focuses on preventing the insertion of clones at
commit time, i.e., before they reach the central code repository.
PRECINCT is an online clone detection technique that relies on the use
of pre-commit hooks capabilities of modern source code version control
systems. A pre-commit hook is a process that one can implement to
receive the latest modification to the source code done by a given
developer just before the code reaches the central repository. PRECINCT
intercepts this modification and analyses its content to see whether a
suspicious clone has been introduced or not. A flag is raised if a code
fragment is suspected to be a clone of an existing code segment. In
fact, PRECINCT, itself, can be seen as a pre-commit hook that detects
clones that might have been inserted in the latest changes with regard
to the rest of the source code.

Similar to clone detection, we propose an approach for preventing the
introduction of bugs at commit-time. Many tools exist to prevent a
developer to ship <span>*bad*</span> code
[@Dangel2000; @Hovemeyer2007; @Moha2010] or to identify
<span>*bad*</span> code after executions (e.g in test or production
environment) [@Nayrolles; @Nayrolles2013a]. However, these tools rely on
metrics and rules to statically and/or dynamically identify sub-optimum
code. Our approach, called <span>BIANCA</span> (Bug Insertion
ANticipation by Clone Analysis at merge time), is different than the
approaches presented in the literature because it mines and analyses the
change patterns in commits and matches them against past commits known
to have introduced a defect in the code (or that have just been replaced
by better implementation).

### A bug reproduction technique based on a combination of crash traces and model checking

When a system crashes, software developers need to reproduce the crash
(usually in a lab environment) so as to provide corrective measures. A
survey conducted with the developers of major open source software
systems such as Apache, Mozilla and Eclipse revealed that one of the
most valuable piece of information that can help locate and fix the
cause of a crash is the one that can help reproduce it
[@Bettenburg2008]. Crash reproduction is an expensive task because the
data provided by end users is often scarce
[@Artzi2008; @Jin2012; @Chen2013]. It is therefore important to invest
in techniques and tools for automatic bug reproduction to ease the
maintenance process and accelerate the rate of bug fixes and patches.
Existing techniques can be divided into two categories: (a) On-field
record and in-house replay
[@Steven2000; @Narayanasamy2005; @Artzi2008; @Roehm2015], and (b)
In-house crash explanation
[@Jin2012; @Jin2013; @Zuddas2014; @Chen2013a; @Nayrolles2015].

In this work, we propose an approach, called JCHARMING (Java CrasH
Automatic Reproduction by directed Model checkING) that uses a
combination of crash traces and model checking to automatically
reproduce bugs that caused field failures. Unlike existing techniques,
JCHARMING does not require instrumentation of the code. It does not need
access to the content of the heap either. Instead, JCHARMING uses a list
of functions output when an uncaught exception in Java occurs (i.e., the
crash trace) to guide a model checking engine to uncover the statements
that caused the crash.

### A new taxonomy of bugs based on the locations of the corrections — an empirical Study

There have been several studies (e.g., [@Weiß2007; @Zhang2013]) that
study of the factors that influence the bug fixing time. These studies
empirically investigate the relationship between bug report attributes
(description, severity, etc.) and the fixing time. Other studies take
bug analysis to another level by investigating techniques and tools for
bug prediction and reproduction (e.g.,
[@Chen2013; @Kim2007a; @Nayrolles2015]). These studies, however, treat
all bugs as the same. For example, a bug that requires only one fix is
analysed the same way as a bug that necessitates multiple fixes.
Similarly, if multiple bugs are fixed by modifying the exact same
locations in the code, then we should investigate how these bugs are
related in order to predict them in the future. Note here that we do not
refer to duplicate bugs. Duplicate bugs are marked as duplicate (and not
fixed) and only the master bug is fixed. From the bug handling
perspective, if we can develop a way to detect related bug reports
during triaging then we can achieve considerable time saving in the way
bug reports are processed, for example, by assigning them to the same
developers.We also conjecture that detecting related bugs can help with
other tasks such as bug reproduction. We can reuse the reproduction of
an already fixed bug to reproduce an incoming and related bug.

We investigate the relationship between bugs by examining their
locations of the fixes. By a fix, we mean a modification (adding or
deleting lines of code) to an exiting file that is used to solve the
bug.We argue that bugs can be classified into four types: A bug of Type
1 refers to a bug being fixed in one single location (i.e., one file),
while Type 2 refers to bugs being fixed in more than one location. Type
3 refers to multiple bugs that are fixed in the exact same location.
Type 4 is an extension of Type 3, where multiple bugs are resolved by
modifying the same set of locations. Note that Type 3 and Type 4 bugs
are not duplicates, they may occur when different features of the system
fail due to the same root causes (faults). We conjecture that knowing
the proportions of each type of bugs in a system may provide insights
into the quality of the system. Knowing, for example, that in a given
system the proportion of Type 2 and 4 bugs is high may be an indication
of poor system quality since many fixes are needed to address these
bugs. In addition, the existence of a high number of Types 3 and 4 bugs
calls for techniques that can effectively find bug reports related to an
incoming bug during triaging. This is similar to the many studies that
exist on detection of duplicates (e.g.,
[@Runeson2007; @Sun2010; @Nguyen2012]), except that we are not looking
for duplicates but for related bugs (bugs that are due to failures of
different features of the system, caused by the same faults).

## Outline {#sec:outline}

The remaining chapters of this proposal are:

-   Chapter \ref{chap:relwork} - <span>*Background & Related work*</span>.
    In this chapter, we present the major studies related to our
    research field, namely, crash reproduction, aggregating bug
    repositories for mining purposes, and clone detection.

-   Chapter \ref{chap:bumper} - <span>*An Aggregate Bug Repository for
    Developers and Researchers*</span>. In this chapter, we present
    <span>BUMPER</span> (BUg Metarepository for dEvelopers and
    Researchers), our bug meta-repository. <span>BUMPER</span> acts as
    our data source for the different contributions.

-   Chapter \ref{chap:jcharming} - <span>*JCHARMING: Java CrasH Automatic
    Reproduction by directed Model checkING*</span>. In this chapter we
    discuss the components of JCHARMING, the bug reproduction approach
    we propose.

-   Chapter \ref{chap:clone-detection-pragmatic} - <span>*Preventing Clone
    Insertion*</span>. This chapter describes one approach to prevent
    the insertion of clones at commit time.

-   Chapter \ref{chap:bianca} - <span>*Preventing Bug Insertion Using
    Clone Detection*</span>. In this chapter, we present an approach
    named <span>BIANCA</span> (Bug Insertion ANticipation by Clone
    Analysis at merge time) which uses clone detection to prevent
    bug insertion.

-   Chapter \ref{chap:plan} - <span>*Remaining Work*</span> presents the
    remaining work and a publication plan.


