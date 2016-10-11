In this section, we present PRECINCT (PREventing Clones INsertion at
Commit Time) that focuses on preventing the insertion of clones at
commit time, i.e., before they reach the central code repository.
PRECINCT is an online clone detection technique that relies on the use
of pre-commit hooks capabilities of modern source code version control
systems. PRECINCT intercepts this modification and analyses its content
to see whether a suspicious clone has been introduced or not. A flag is
raised if a code fragment is suspected to be a clone of an existing code
segment. In fact, PRECINCT, itself, can be seen as a pre-commit hook
that detects clones that might have been inserted in the latest changes
with regard to the rest of the source code. This said, only a fraction
of the code is analysed, making PRECINCT efficient compared to leading
clone detection techniques such as NICAD (Accurate Detection of
Near-miss Intentional Clones) [@Cordy2011]. Moreover, the detected
clones are presented using a classical ‘diff’ output that developers are
familiar with. PRECINCT is also well integrated with the workflow of the
developers since it is used in conjunction with a source code version
control systems such as Git[^1].

In this study, we focus on Type 3 clones as they are more challenging to
detect. Since Type 3 clones include Type 1 and 2 clones, then these
types could be detected separately by PRECINCT as well.

PRECINCT aims to prevent clone insertion while integrating the clone
detection process in a transparent manner in the day-to-day maintenance
process. This way, software developers do not have to resort to external
tools to remove clones after they are inserted such as the one presented
in Section \[sec:rel-clones\]. Our approach operates at commit time,
notifying software developers of possible clones as they commit their
code.

We evaluated the effectiveness of PRECINCT using precision and recall on
three systems, developed independently and written in both C and Java.
The results show that PRECINCT prevents Type 3 clones to reach the final
source code repository with an average accuracy of 97.7%.

The PRECINCT Approach {#sec:The PRECINCT Approach}
=====================

![image](media/approach.png){width="\textwidth"}

The PRECINCT approach is composed of six steps. The first and last steps
are typical steps that a developer would do when committing code.
Indeed, the first step is the commit step where developers send their
latest changes to the central repository and the last step is the
reception of the commit by the central repository. The second step is
the pre-commit hook, which kicks in as the first operation when one
wants to commit. The pre-commit hook has access to the changes in terms
of files that have been modified, more specifically, the lines that have
been modified. The modified lines of the files are sent to
TXL[@Cordy2006a] for block extraction. Then, the blocks are compared to
previously extracted blocks in order to identify candidate clones using
the comparison engine of NICAD[@Cordy2011]. We chose NICAD engine
because it has been shown to provide high accuracy [@Cordy2011]. The
tool is also readily available, easy to use, customizable, and works
with TXL. Note, however, that PRECINCT can also work with other engines
for comparing code fragments. Finally, the output of NICAD is further
refined and presented to the user for decision. These steps are
discussed in more detail in the following subsections.

PRECINCT Pre-Commit Hook {#sub:Pre-Commit Hook}
========================

Depending on the exit status of the hook, the commit will be aborted and
not pushed to the central repository. Also, developers can choose to
ignore the pre-hook. In Git, for example, they will need to use the
command `git commit –no-verify` instead of `git commit`. This can be
useful in case of an urgent need for fixing a bug where the code has to
reach the central repository as quickly as possible. Developers can do
things like check for code style, check for trailing white spaces (the
default hook does exactly this), or check for appropriate documentation
on new methods.

PRECINCT is a set of bash scripts where the entry point of these scripts
lies in the pre-commit hooks. Pre-commit hooks are easy to create and
implement as depicted in Listing \[gitprehook\]. This pre-hook is
shipped with Git, a popular version control system. Note that even
though we use Git as the main version control to present PRECINCT, we
believe that the techniques presented in this section are readily
applicable to other version control systems. In Listing \[gitprehook\],
from lines 3 to 11, the script identifies if the commit is the first one
in order to select the revision to work against. Then, in Lines 18 and
19, the script checks for trailing whitespace and fails if any are
found.

For PRECINCT to work, we just have to add the call to our script suite
instead or in addition of the whitespace check.

<span>0.90</span>

Extract and Save Blocks {#sub:Extract and Save Blocks}
=======================

A block is a set of consecutive lines of code that will be compared to
all other blocks in order to identify clones. To achieve this critical
part of PRECINCT, we rely on TXL[@Cordy2006a], which is a first-order
functional programming over linear term rewriting, developed by Cordy et
al.[@Cordy2006a]. For TXL to work, one has to write a grammar describing
the syntax of the source language and the transformations needed. TXL
has three main phases: *parse, transform*, *unparse*. In the parse
phase, the grammar controls not only the input but also the output form.
Listing \[txlsample\] — extracted from the official documentation[^2] —
shows a grammar matching an *if-then-else* statement in C with some
special keywords: \[IN\] (indent), \[EX\] (exdent) and \[NL\] (newline)
that will be used for the output form.

<span>0.90</span>

Then, the *transform* phase will, as the name suggests, apply
transformation rules that can, for example, normalize or abstract the
source code. Finally, the third phase of TXL, called *unparse*, unparses
the transformed parsed input in order to output it. Also, TXL supports
what the creators call Agile Parsing[@Dean], which allow developers to
redefine the rules of the grammar and, therefore, apply different rules
than the original ones.

PRECINCT takes advantage of that by redefining the blocks that should be
extracted for the purpose of clone comparison, leaving out the blocks
that are out of scope. More precisely, before each commit, we only
extract the blocks belonging to the modified parts of the source code.
Hence, we only process, in an incremental manner, the latest
modification of the source code instead of the source code as a whole.

We have selected TXL for several reasons. First, TXL is easy to install
and to integrate with the normal workflow of a developer. Second, it was
relatively easy to create a grammar that accepts commits as input. This
is because TXL is shipped with C, Java, Csharp, Python and WSDL grammars
that define all the particularities of these languages, with the ability
to customize these grammars to accept changesets (chunks of the modified
source code that include the added, modified, and deleted lines) instead
of the whole code.

Algorithm \[alg:extract\] presents an overview of the “extract" and
“save" blocks operations of PRECINCT. This algorithm receives as
arguments, the changesets, the blocks that have been previously
extracted and a boolean named compare\_history. Then, from Lines 1 to 9
lie the $for$ loop that iterates over the changesets. For each changeset
(Line 2), we extract the blocks by calling the
$~extract\_blocks(Changeset~cs)$ function. In this function, we expand
our changeset to the left and to the right in order to have a complete
block.

As depicted by Listing \[commitsample\], changesets contain only the
modified chunk of code and not necessarily complete blocks. Indeed, we
have a block from Line 3 to Line 6 and deleted lines from Line 8 to 14.
However, in Line 7 we can see the end of a block, but we do not have its
beginning. Therefore, we need to expand the changeset to the left in
order to have syntactically correct blocks. We do so by checking the
block’s beginning and ending (using { and }) in C for example. Then, we
send these expanded changesets to TXL for block extraction and
formalization.

For each extracted block, we check if the current block overrides
(replaces) a previous block (Line 4). In such a case, we delete the
previous block as it does not represent the current version of the
program anymore (Line 5). Also, we have an optional step in PRECINCT
defined in Line 4. The compare\_history is a condition to delete
overridden blocks.

We believe that deleted blocks have been deleted for a good reason (bug,
default, removed features, …) and if a newly inserted block matches an
old one, it could be worth knowing in order to improve the quality of
the system at hand. This feature is deactivated by default.

In summary, this step receives the files and lines, modified by the
latest changes made by the developer and produces an up to date block
representation of the system at hand in an incremental way. The blocks
are analysed in the next step to discover potential clones.

Compare Extracted Blocks {#sub:Compare Extracted Blocks}
========================

In order to compare the extracted blocks and detect potential clones, we
can only resort to text-based techniques. This is because lexical and
syntactic analysis approaches (alternatives to text-based comparisons)
would require a complete program to work, a program that compiles. In
the relatively wide-range of tools and techniques that exist to detect
clones by considering code as
text[@Johnson1993; @Johnson1994; @Marcus; @Manber1994; @StephaneDucasse; @Wettel2005],
we selected NICAD as the main text-based method for comparing clones
[@Cordy2011] for several reasons. First, NICAD is built on top of TXL,
which we also used in the previous step. Second, NICAD is able to detect
all Types 1, 2 and 3 software clones.

NICAD works in three phases: *Extraction*, *Comparison* and *Reporting*.
During the *Extraction* phase all potential clones are identified,
pretty-printed, and extracted. We do not use the *Extraction* phase of
NICAD as it has been built to work on programs that are syntactically
correct, which is not the case for changesets. We replaced NICAD’s
*Extraction* phase with our own scripts, described in the previous
section.

In the *Comparison* phase, extracted blocks are transformed, clustered
and compared in order to find potential clones. Using TXL sub-programs,
blocks go through a process called pretty-printing where they are
stripped of formatting and comments. When code fragments are cloned,
some comments, indentation or spacing are changed according to the new
context where the new code is used. This pretty-printing process ensures
that all code will have the same spacing and formatting, which renders
the comparison of code fragments easier. Furthermore, in the
pretty-printing process, statements can be broken down into several
lines. Table \[tab:pretty-printing\] shows how this can improve the
accuracy of clone detection with three `for` statements,
` for (i=0; i<10; i++)`, `for (i=1; i<10; i++)` and
` for (j=2; j<100; j++)`. The pretty-printing allows NICAD to detect
Segments 1 and 2 as a clone pair because only the initialization of $i$
changed. This specific example would not have been marked as a clone by
other tools we tested such as Duploc[@Ducasse1999]. In addition to the
pretty-printing, code can be normalized and filtered to detect different
classes of clones and match user preferences.

Finally, the extracted, pretty-printed, normalized and filtered blocks
are marked as potential clones using a Longest Common Subsequence (LCS)
algorithm[@Hunt1977]. Then, a percentage of unique statements can be
computed and, depending on a given threshold (see
Section \[sec:Experimentations\]), the blocks are marked as clones.

The last step of NICAD, which acts as our clone comparison engine, is
the *reporting*. However, to prevent PRECINCT from outputting a large
amount of data (an issue that many clone detection techniques face), we
implemented our own reporting system, which is also well embedded with
the workflow of developers. This reporting system is the subject of the
next section.

As a summary, this step receives potentially expanded and balanced
blocks from the extraction step. Then, the blocks are pretty-printed,
normalized, filtered and fed to an LCS algorithm in order to detect
potential clones. Moreover, the clone detection in PRECINCT is less
intensive than NICAD because we only compare the latest changes with the
rest of the program instead of comparing all the blocks with each other.

Output and Decision {#sub:Output and Decision}
===================

In this final step, we report the result of the clone detection at
commit time with respect to the latest changes made by the developer.
The process is straightforward. Every change made by the developer goes
through the previous steps and is checked for the introduction of
potential clones. For each file that is suspected to contain a clone,
one line is printed to the command line with the following options: (I)
Inspect, (D) Disregard, (R) Remove from the commit as shown by Figure
\[fig:hook\]. In comparison to this simple and interactive output, NICAD
outputs each and every detail of the detection result such as the total
number of potential clones, the total number of lines, the total number
of unique line text chars, the total number of unique lines, and so on.
We think that so many details might make it hard for developers to react
to these results. A problem that was also raised by Johnson et al.
[@Johnson2013] when examining bug detection tools. Then the potential
clones are stored in XML files that can be viewed using an Internet
browser or a text editor.

![PRECINCT output when replaying commit `710b6b4` of the Monit system
used in the case
study.\[fig:hook\]](media/commit.png){width="48.00000%"}

\(I) Inspect will cause a diff-like visualization of the suspected clones
while (D) disregard will simply ignore the finding. To integrate
PRECINCT in the workflow of the developer we also propose the remove
option (R). This option will simply remove the suspected file from the
commit that is about to be sent to the central repository. Also, if the
user types an option key twice, e.g., II, DD or RR, then the option will
be applied to all files. For instance, if the developer types DD at any
point, the PRECINCT’s results will be disregarded and the commit will be
allowed to go through. We believe that this simple mechanism will
encourage developers to use PRECINCT like they would use any other
feature of Git (or any other version control system).

Case Study {#sec:Experimentations}
==========

In this section, we show the effectiveness of PRECINCT for detecting
clones at commit time in three open source systems[^3].

The aim of the case study is to answer the following question: *Can we
detect clones at commit time, i.e., before they are inserted in the
final code, if so, what would be the accuracy compared to a traditional
clone detection tool such as NICAD?*

### Target Systems {#sub:Target Systems}

Table \[tab:sut\] shows the systems used in this study and their
characteristics in terms of the number files they contain and the size
in KLoC (Kilo Lines of Code). We also include the number of revisions
used for each system and the programming language in which the system is
written.

Monit[^4] is a small open source utility for managing and monitoring
Unix systems. Monit is used to conduct automatic maintenance and repair
and supports the ability to identify causal actions to detect errors.
This system is written in C and composed of 826 revisions, 264 files,
and the latest version has 107 KLoC. We have chosen Monit as a target
system because it was one of the systems NICAD was tested on.

JHotDraw[^5] is a Java GUI framework for technical and structured
graphics. It has been developed as a “design exercise”. Its design
relies heavily on the use of design patterns. JHotDraw is composed of
735 revisions, 1984 files, and the latest revision has 44 KLoC. It is
written in Java and it is often used by researchers as a test bench.
JHotDraw was also used by NICAD’s developers to evaluate their approach.

Dnsjava[^6] is a tool for implementing the DNS (Domain Name Service)
mechanisms in Java. This tool can be used for queries, zone transfers,
and dynamic updates. It is not as large as the other two, but it still
makes an interesting case subject because it has been well maintained
for the past decade. Also, this tool is used in many other popular tools
such as Aspirin, Muffin and Scarab. Dnsjava is composed of 1637
revisions, 233 files, the latest revision contains 47 KLoC.

Process {#sub:Process}
=======

Figure \[fig:precinct-branching\] shows the process we followed to
validate the effectiveness of PRECINCT.

![PRECINCT
Branching.\[fig:precinct-branching\]](media/branch.png){width="30.00000%"}

As our approach relies on commit pre-hooks to detect possible clones
during the development process (more particularly at commit time), we
had to find a way to *replay* past commits. To do so, we *cloned* our
test subjects, and then created a new branch called *PRECINCT\_EXT*.
When created, this branch is reinitialized at the initial state of the
project (the first commit) and each commit can be replayed as they have
originally been. At each commit, we store the time taken for PRECINCT to
run as well as the number of detected clone pairs. We also compute the
size of the output in terms of the number of lines of text output by our
method. The aim is to reduce the output size to help software developers
interpret the results.

To validate the results obtained by PRECINCT, we needed to use a
reliable clone detection approach to extract clones from the target
systems and use these clones as a baseline for comparison. For this, we
turned to NICAD because of its popularity, high accuracy, and
availability [@Cordy2011], as discussed before. This means, we run NICAD
on the revisions of the system to obtain the clones then we used NICAD
clones as a baseline for comparing the results obtained by PRECINCT.

It may appear strange that we are using NICAD to validate our approach,
knowing that our approach uses NICAD’s code comparison engine. In fact,
what we are assessing here is the ability for PRECINCT to detect clones
at commit time using changsets. The major part of PRECINCT is the
ability to intercept code changes and build working code blocks that are
fed to a code fragment engine (in our case NICAD’s engine). PRECINCT can
be built on the top of any other code comparison engine.

We show the result of detecting Type 3 clones with a maximum line
difference of 30% as discussed in Table \[tab:result\]. As discussed in
the introductory section, we chose to report on Type 3 clones because
they are more challenging to detect than Type 1 and 2. PRECINCT detects
Type 1 and 2 too so does NICAD. For the time being, PRECINCT is not
designed to detect Type 4 clones. These clones use different
implementations. Detecting Type 4 clones is part of future work.

We assess the performance of PRECINCT in terms of precision (Equation 1)
and recall (Equation 2). Both precision and recall are computed by
considering NICAD’s results as a baseline. We also compute
F$_{1}$-measure (Equation 3), i.e., the weighted average of precision
and recall, to measure the accuracy of PRECINCT.

$$precision = \frac{|\{ NICAD_{detection} \} \cap \{ PRECINCT_{detection} \} |}{| \{ PRECINCT_{detection} \}|}$$

$$recall = \frac{|\{ NICAD_{detection} \} \cap \{ PRECINCT_{detection} \} |}{| \{ NICAD_{detection} \}|}$$

$$F_1-measure = 2 * \frac{precision * recall}{precision + recall}$$

Results {#sub:Results}
=======

Figures \[fig:r1\], \[fig:r2\], \[fig:r3\] show the results of our study
in terms of clone pairs that are detected per revision for our three
subject systems: Monit, JHotDraw and Dnsjava. We used as baseline for
comparison the clone pairs detected by NICAD. The blue line shows the
clone detection performed by NICAD. The red line shows the clone pairs
detected by PRECINCT. The brown line shows the clone pairs that have
been missed by PRECINCT. As we can quickly see, the blue and red lines
almost overlap, which indicates a good accuracy of the PRECINCT
approach.

Table \[tab:result\] summarizes PRECINCT’s results in terms of
precision, recall, F$_{1}$-measure, execution time and output reduction.
The first version of Monit contains 85 clone pairs and this number stays
stable until Revision 100. From Revision 100 to 472 the detected clone
pairs vary between 68 and 88 before reaching 219 at Revision 473. The
number of clone pairs goes down to 122 at Revision 491 and decreases to
128 in the last revision. PRECINCT was able to detect 96.1% (123/128) of
the clone pairs that are detected by NICAD with a 100% recall. It took
in average around 1 second for PRECINCT to execute on a Debian 8 system
with Intel(R) Core(TM) i5-2400 CPU @ 3.10GHz, 8Gb of DDR3 memory. It is
also worth mentioning that the computer we used is equipped with SSD
(Solid State Drive). This impacts the running time as clone detection is
a file intensive operation. Finally, the PRECINCT was able to output
88.3% less lines than NICAD.

JHotDraw starts with 196 clone pairs at Revision 1 and reaches a pick of
2048 at Revision 180. The number of clones continues to go up until
Revisions 685 and 686 where the number of pairs is 1229 before picking
at 6538 and more from Revisions 687 to 721. PRECINCT was able to detect
98.3% of the clone pairs detected by NICAD (6490/6599) with 100% recall
while executing on average in 1.7 second (compared to 5.1 seconds for
NICAD). With JHotDraw, we can clearly see the advantages of incremental
approaches. Indeed, the execution time of PRECINCT is loosely impacted
by the number of files inside the system as the blocks are constructed
incrementally. Also, we only compare the latest change to the remaining
of the program and not all the blocks to each other as NICAD. We also
were able to reduce by 70.1% the number of lines output by NICAD.

Finally, for Dnsjava, the number of clone pairs starts high with 258
clones and goes up until Revision 70 where it reaches 165. Another quick
drop is observed at Revision 239 where we found only 25 clone pairs. The
number of clone pairs stays stable until Revision 1030 where it reaches
273. PRECINCT was able to detect 82.8% of the clone pairs detected by
NICAD (226/273) with 100% recall, while executing on average in 1.1
second while NICAD took 3 seconds in average. PRECINCT outputs 83.4%
less lines of code than NICAD.

Overall, PRECINCT prevented 97.7% of the 7000 clones (in all systems) to
reach the central source code repository while executing more than twice
as fast as NICAD (1.2 seconds compared to 3 seconds in average) while
reducing the output in terms of lines of text output the developers by
83.4% in average. Note here that we have not evaluated the quality of
the output of PRECINCT compared to NICAD’s output. We need to conduct
user studies for this. We are, however, confident, based on our own
experience trying many clone detection tools, that a simpler and more
interactive way to present the results of a clone detection tool is
warranted. PRECINCT aims to do just that.

The difference in execution time between NICAD and PRECINCT stems from
the fact that, unlike PRICINCT, NICAD is not an incremental approach.
For each revision, NICAD has to extract all the code blocks and then
compares all the pairs with each other. On the other hand, PRECINCT only
extracts blocks when they are modified and only compares what has been
modified with the rest of the program.

The difference in precision between NICAD and PRECINCT (2.3%) can be
explained by the fact that sometimes developers commit code that does
not compile. Such commits will still count as a revision, but TXL fails
to extract blocks that do not comply with the target language syntax.
While NICAD also fails in such a case, the disadvantage of PRECINCT
comes from the fact that the failed block is saved and used as reference
until it is changed by a correct one in another commit.

[^1]: https://git-scm.com/

[^2]: http://txl.ca

[^3]: The programs used and instructions to reproduce the experiments
    are made available for download from
    https://research.mathieu-nayrolles.com/precinct/

[^4]: https://mmonit.com/monit/

[^5]: http://www.jhotdraw.org/

[^6]: http://www.dnsjava.org/
