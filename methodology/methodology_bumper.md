With the goal to support the research towards analyzing relationships
between bugs and their fixes we constructed a dataset of 380 projects,
more than 100,000 resolved/fixed and with 60,000 changesets that were
involved in fixing them from Netbeans and The Apache Software
foundation’s software that is (1) searchable in natural language at
https://bumper-app.com, (2) contains clear relationships between the bug
report and the code involved to fix it, (3) supports complex queries
such as parent-child relationships, unions or disjunctions and (4)
provide easy exports in json, csv and xml format.

In what follows, we will present the projects we selected. Then, we
present the features related to the bugs and their fixes we integrate in
BUMPER (BUg Metarepository for dEvelopers and Researchers) and how we
construct our dataset. Then, we present the API, based on Apache Solr
[@Nayrolles2014b], which allows the NLP search with practical examples
before providing research opportunities based on our dataset.

However, to the best of our knowledge, no attempt has been made towards
building a unified and online dataset where all the information related
to a bug, or a fix can be easily accessed by researchers and engineers.

Data collection\[sec:data-collection\]
======================================

Figure \[fig:bumper-approach\] illustrates our data collection and
analysis process that we present here and discuss in more detail in the
following subsections. First, we extract the raw data from the two bug
report management systems used in this study (Bugzilla[^1] and
Jira[^2]). The extracted data is consolidated in one database called
BUMPER where we associate each bug report with its fix. The fixes are
mined from different type of source versioning system. Indeed, Netbeans
is based on mercurial[^3] while we used the git[^4] mirrors[^5] for the
Apache Foundation software.

![Overview of the bumper database construction.
\[fig:bumper-approach\]](media/bumper-approach.png)

In this study, we used two distinct datasets: Netbeans and the Apache
Software Foundation projects. Netbeans is an integrated development
environment (IDE) for developing with many languages including Java,
PHP, and C/C++. The very first version of Netbeans, then known as Xelfi,
appeared in 1996. The Apache Software Foundation is a U.S non-profit
organization supporting Apache software projects such as the popular
Apache web server since 1999. The characteristics of the Netbeans and
Apache Software Foundation are presented in Table \[table:datasets\].

<span>@c|c|c|c|c@</span> **Dataset** & **R/F BR** & **CS** & **Files** &
**Projects**\
Netbeans & 53,258 & 122,632 & 30,595 & 39\
Apache & 49,449 & 106,366 & 38,111 & 349\
Total & 102,707 & 229,153 & 68,809 & 388\

Cumulatively, these datasets span from 2001 to 2014. In summary, our
consolidated dataset contains 102,707 bugs, 229,153 changesets, 68,809
files that have been modified to fix the bugs, 462,848 comments, and 388
distinct systems. We also collected 221 million lines of code modified
to fix the bugs, identified 3,284 sub-projects, and 17,984 unique
contributors to these bug report and source code version management
systems. Finally, the cumulated opening time for all the bugs reaches
10,661 working years (3,891,618 working days).

We choose to use these two datasets because they exposed a great
diversity in programming languages, teams, localization, utility and
maturity. Moreover, the used different tools, i.e. Bugzilla, JIRA, Git
and Mercurial, and therefore, BUMPER is ready to host any other datasets
that used any composition of these tools.

Architecture
============

<span>BUMPER</span> rely on a highly scalable architecture composed of
two distinct servers as depicted in Figure \[fig:bumper-arch\]. The
first server, on the left, handles the web requests and runs three
distinct components:

-   Pound is a lightweight open source reverse proxy program and
    application firewall. It is also served us to decode to request
    to http. Translating an request to http and then, use this HTTP
    request instead of the one allow us to save the http’s decryption
    time required at each step. Pound also acts as a load-balancing
    service for the lower levels.

-   Translated requests are then handled to Varnish. Varnish is an HTTP
    accelerator designed for content-heavy and dynamic websites. What it
    does is caching request that come in and serve the answer from the
    cache is the cache is still valid.

-   NginX (pronounced engine-x) is a web-server that has been developed
    with a particular focus on high concurrency, high performances and
    low memory usage.

On the second server, that concretely handles our data, we have the
following items:

-   Pound. Once again, we use pound here, for the exact same reasons.

-   SolrCloud is the scalable version of Apache Solr where the data can
    be separated into shards (e.g chunk of manageable size). Each shard
    can be hosted on a different server, but it’s still indexed in a
    central repository. Hence, we can guarantee a low query time while
    exponentially increasing the data.

-   Lucene is the full text search engine powering Solr. Each Solr
    server has its own embedded engine.

![Overview of the bumper architecture.
\[fig:bumper-arch\]](media/bumper-arch.png)

Request from users to the servers and the communication between our
servers are going through the CloudFlare network. CloudFlare acts as a
content delivery network sitting between the users and the webserver.
They also provide an extra level of caching and security.

To give the reader a glimpse about the performances that this unusual
architecture can yield; we are able to request and display the result of
a specific request in less than 100 ms while our two servers are, in
fact, two virtual machines sharing an AMD Opteron (tm) Processor 6386 SE
(1 core @ 2,000 MHz) and 1 GB of RAM.

UML Metamodel
=============

Figure \[fig:bumper-approach\] presents the simplified
<span>BUMPER</span> metamodel that we designed according to our bug
taxonomy presented in section \[fig:bug-taxo\] and according to our
future needs for <span>JCHARMING</span>, <span>RESSEMBLE</span> and
<span>BIANCA</span>.

![Overview of the bumper meta-model. \[fig:bumper-approach\]
](media/bumper-model.png)

An <span>*issue*</span> (<span>*task*</span>) is characterized by a
<span>*date*</span>, <span>*title*</span>, <span>*description*</span>,
and a <span>*fixing time*</span>. They are reported (created) by and
assigned to <span>*users*</span>. Also, <span>*issues*</span>
(<span>*tasks*</span>) belong to <span>*project*</span> that are in
<span>*repository*</span> and might be composed of
<span>*sub-projects*</span>. <span>*Users*</span> can modify an
<span>*issue*</span> (<span>*task*</span>) during <span>*life cycle
events*</span> which impact the <span>*type*</span>, the
<span>*resolution*</span>, the <span>*platform*</span>, the
<span>*OS*</span> and the <span>*status*</span>. <span>*Issues*</span>
(<span>*tasks*</span>) are resolved (implemented) by
<span>*changeset*</span> that are composed of <span>*hunks*</span>.
<span>*Hunks*</span> contain the actual changes to a <span>*file*</span>
at a given revision, which are versions of the <span>*file*</span>
entity that belongs to a <span>*Project*</span>.

Features
========

In this section, we present the features of bug report and their fixes
in details.

Bug Report
----------

A bug report is characterized by the following features:

-   ID: unique string id of the form bug\_dataset\_project\_bug\_id

-   Dataset: the dataset of which the bug is extracted from.

-   Type: The type help us to distinguish different type of entities in
    BUMPER, i.e the bugs, changesets and hunks. For bug report, the type
    is always set to BUG

-   Date: The date at which the bug report has been submitted.

-   Title: The title of the bug report.

-   Project: The project that this bug affects.

-   Sub\_project: The sub-project that this bug affects.

-   Full\_name\_project: The combination of the project and
    the sub-project.

-   Version: the version of the project that this bug affects

-   Impacted\_platform: the platform that this bug affects

-   Impacted\_os: the operating system that this bug affects

-   Bug\_status: The status of the bug. As in bumper, our main concern
    is on the relationship between of fix and a bug, we only have
    RESOLVED bugs

-   Resolution: How the bug was resolved. Once again, as we are
    interested in investigating the fixes and the bugs, we only have
    FIXED bugs.

-   Reporter\_pseudo: the pseudonym of the person who report the bug.

-   Reporter\_name: the name of the person who reported the bug

-   Assigned\_to\_pseudo: the pseudonym of the person who have

-   been assigned to fix this bug

-   Assigned\_to\_name: the name of the person who have been assigned to
    fix this bug

-   Bug\_severity: the severity of a bug

-   Description: the description of the bug the reporter gave

-   Fixing\_time: The time it took to fix the bug, i.e the elapsed time
    between the creation of the BR and its modification to
    resolve/fixed, in minutes

-   Comment\_nb: How many comments have been posted on the bug report
    system for that bug

-   Comment: Contains one comment. A bug can have 0 or many comments

-   File: A file qualified name that has been modified in order to fix
    a bug. A bug can have 0 (in case we did not find its related commit)
    or many files.

We selected this set of features for bug report as they are the ones
that are analyzed in many past and recent studies. In addition, bugs can
contain 0 or many .

Changesets
----------

In this section, we present the features that characterize changeset
entities in BUMPER.

-   ID: the SHA1 hash

-   User: the name and email of the person who submitted that commit

-   Date: the date at which this commit has been fixed

-   Summary: the commit message entered by the user

-   File: The fully qualified name of a file modified on that commits. A
    changeset can have 1 or many files.

-   Number\_files: How many files have been modified in that commit

-   Insertions: the number of inserted lines

-   Deletions: the number of deleted lines

-   Churns: the number of modified lines

-   Hunks: the number of sets of consecutive changed lines

-   Parent\_bug: the id of the bug this changeset belongs to.

In addition, changesets contain one or many hunks.

Hunks
-----

A hunks are a set of consecutive lines changed in a file in order. A set
of hunks form a fix that can be scattered across one or many files.
Knowing how many hunks a fixed required and what are the changes in each
of them is useful, as explained by \[2\] to understand how many places
developers have to go to fix a bug.

Hunks are composed of:

-   ID: unique id based on the files, the insertion and the SHA1 of the
    commits

-   Parent\_changeset: the SHA1 of the Changeset this hunk belongs to

-   Parent\_bug: the id of the bug this hunk belongs to.

-   Negative\_churns: how many lines have been removed in that hunk

-   Positive\_churns: how many lines have been added in that hunk

-   Insertion: the position in a file at which this hunk takes place.

-   Change: One line that have been added or removed. A Hunk can contain
    one or many changes.

Application Program Interface (API)\[sec:bumper-api\]
=====================================================

BUMPER is available for engineers and researchers at
<span>**https://bumper-app.com**</span> and take the form of a regular
search engine. Bumper supports (1) natural language query, (2)
parent-child relationships, query, (3) disjunctions and union between
complex queries and (4) a straight forward export of query results in
XML, CSV or JSON format.

Browsing BUMPER, the basic query mode, perform the following operation:

$$\begin{split}
(type:BUG~AND~report\_t:(``YOUR~TERMS''))~OR~(!parent~which=type``BUG'')~\\fix\_t:``YOUR~TERMS'')
\end{split}$$

The first part of the query component of the query retrieves all the
bugs that contains the $``YOUR~TERMS''$ query in at least one its
features by selecting type: BUG and report\_t, which is an index
composed of all the features of the bug, set to $``YOUR~TERMS''$. Then,
we merge this query with another one that reads\
$(!parent~which=type``BUG'')fix\_t:~``YOUR~TERMS'')$. In this one, we
retrieve the parent documents, i.e the bugs, of fixes that contains
$``YOUR~TERMS''$ in their $fix\_t$ index. The $fix\_t$ index is, as for
the BUG, an index based on all the fields of changeset and hunk both. As
a result, we search seamlessly in the bug report and their fixes in
natural language.

As a more practical example, Figure \[fig:bumper-live\] illustrate a
query on https://bumper-app.com. The search term is
“<span>*Exception*</span>” and we can see that 20,285 issues / tasks
have been found in 25 ms This particular set of issues, displayed on the
left side, match because they contain “<span>*Exception*</span>” in the
issue report or in the source code modified to fix this issue (implement
this task). Then on the right side of the screen, the selected issue
(task) is displayed. We can see the basic characteristic of the issue
(task) followed by comments and finally, the source code.

![Screenshot of https://bumper-app.com with “Exception” as research.
\[fig:bumper-live\]](media/bumper-live.png)

Moreover, BUMPER supports AND, OR, NOR operators and provide results in
order of seconds.

As we said before, BUMPER is based on Apache Solr which have an
incredibly rich API that is available online[^6].

<span>BUMPER</span> serves as data repositories for the upcoming
approaches presented in the chapters.

[^1]: https://netbeans.org/bugzilla/

[^2]: https://issues.apache.org/jira/issues/?jql=

[^3]: http://mercurial.selenic.com/

[^4]: http://git-scm.com/

[^5]: https://github.com/apache

[^6]: http://lucene.apache.org/solr/resources.html
