RESEMBLE: REcommendation System based on cochangE Mining at Block LEvel\[sec:RESEMBLE\]
=======================================================================================

<span>RESEMBLE</span> (REcommendation System based on cochangE Mining at
Block LEvel) is a contextual recommendation system that will take the
form of another pre-commit hook. <span>RESEMBLE</span> will leverage the
branches’ history and the blocks extracted by <span>PRECINCT</span> to
recommend blocks that should be modified. More specifically,
<span>RESEMBLE</span> will mine the blocks extracted by
<span>PRECINCT</span> in order to establish sequences of changes. On new
changes, the previously established sequence can be compared to the new
ones and recommendation made about which blocks you have been modified.

In the data mining field, association rule mining (ARM) is a
well-established method for discovering co-occurrences between
attributes in the objects of a large data set
[@Gregory1991; @HEIKKI1997]. Plain associations have the form
$X \rightarrow Y$, where $X$ and $Y$, called the *antecedent* and the
*consequent*, respectively, are sets of descriptors (purchases by a
customer, network alarms, or any other general kind of events). Even
though plain association rules could serve some relevant information, we
are interested here in the sequences of changes that we believe will
yield more precise result. Indeed, we think that similar modifications
are often done in the same order by the same developer (e.g top to
bottom or bottom to top). We, therefore, adopt a variant called
sequential association rules in which both $X$ and $Y$ become sequences
of descriptors. Moreover, our sequences follow a temporal order with the
antecedent preceding the consequent. Rules of this type mined from
changes reveal crucial information about the likelihood of blocks of
code to be modified together in a programming session and, more
importantly, in a specific order. For instance, a strong rule *$Block_A$
$,$ $Block_B$* implies *$Block_C$* would mean that after modifying
$Block_A$ and then $Block_B$, there are good chances that the developer
needs to modify $Block_C$. The conciseness of this example should not
confuse the reader as in practical cases the sequences appearing in a
rule can be of an arbitrary length. Furthermore, the strength of the
rule is measured by the *confidence* metric. In probabilistic terms, it
measures the conditional probability of C appearing down the line.
Beside that, the significance of a rule, i.e. how many times it appears
in the data, is provided by its *support* measure. To ensure only rules
of potentially high interestingness are mined, the mining task is tuned
by minimal thresholds to output only the sufficiently high scores for
both metrics.

To extract the association rules from changes, two choices are possible.
On one hand, sequential pattern mining and rule mining algorithms have
been designed for structures that are slightly more general than the
ones used here. In fact, sequential patterns are defined on transactions
that represent sequences of sets. Efficient sequential pattern miners
have been published, e.g. the PrefixSpan method [@Pei2004]. On the other
hand, sequence of changes do not compile to fully-blown sequential
transactions as the underlying structures are mere sequences of
individual elements. Such data has been known since at least the mid-90s
but received less attention by the data mining community, arguably
because it is less challenging to mine. In the general data mining
literature, mining from pure sequences, as opposed to sequences made of
sets, has been addressed under the name of episode mining [@HEIKKI1997].
Episodes are made of *events* and in a sense, code changes are events.
Arguably the largest body of knowledge on the subject belongs to the web
usage mining field: The input data is again a system log, yet this time
the log of requests sent to a web server [@Pei2000]. It is noteworthy
that sequential patterns are more general than the pure sequence ones,
hence mining algorithms designed for the former might prove to be less
efficient when applied to the latter (as additional steps might be
required for listing all significant set). Nevertheless, to jump-start
our experimental study, we will use a sequential pattern/rule miner that
has the advantage to be freely available on the web[^1]. Although it has
not been optimized for pure sequences its performances are more than
satisfactory.

We did not started the experiments for <span>RESEMBLE</span> yet.

We believe that <span>RESEMBLE</span> will be a real asset in a
developer tool belt in order to ship better code in terms of quality,
performances and security. However, as <span>RESEMBLE</span> aims to
provide recommendations in at commit-time, using the local history and
ressources, it will not be able to be as exhaustive as an offline
process. To fill this gap, we built <span>BIANCA</span> that we present
in the next section.

[^1]: http://www.philippe-fournier-viger.com/spmf/
