
# Conclusion {#chap:conclusion}

The maintenance and evolution of complex software systems account for
more than 70% software’s life cycle. Hundreds of papers have been
published with the aim to improve our knowledge of these processes in
terms of issue triaging, issue prediction, duplicate issue detection,
issue reproduction and co-changes prediction. All these publications
gave meaning to the millions of issues that can be found in open source
issue & project and revision management systems. Context-aware IDE and
think tank in open source architecture ([@chansler2011architecture])
open the path to approaches that support developers during their
programming sessions by leveraging past indexed knowledge and past
architectures.

In this research proposal, we first presented the most influential
papers in the different fields our work lies on in Chapter
\[chap:relwork\]. Chapter \[chap:methodology\] presented our proposal in
details while chapter \[chap:plan\] detailed our attempt planning.

More specifically, in Chapter \[chap:methodology\], we presented four
approaches: <span>BUMPER</span>, <span>JCHARMING</span>,
<span>RESEMBLE</span>, <span>BIANCA</span>. Also, we proposed a taxonomy
of bugs. When combined into <span>pErICOPE</span> (Ecosystem Improve
source COde during Programming session with real-time mining of common
knowlEdge), these tools (i) provide the possibility to search related
software artifacts using natural language, (ii) accurately reproduce
field-crash in lab environment, (iii) recommend improvement or
completion of current block of code and (iv) prevent the introduction of
clones / issues at commit time.

<span>BUMPER</span> has been designed to handle heavy traffic while
<span>JCHARMING</span> can reproduce 85% of real-world issues we
submitted to it. On its side <span>BIANCA</span> is able to flag 41.5%
and 48.89% of commit introducing a bug as dangerous with 13.4% and 21%
of false positive using two code different code normalization,
respectively.

Our future works, according to our publication plan described in section
\[sec:publication-plan\], are as follows. First, we want to improve the
performances of <span>BIANCA</span> in terms of false positives. Then,
create the IDE plugin that will support <span>RESEMBLE</span>. Finally,
we want to refine our taxonomy by including as many as datasets as
possible.
