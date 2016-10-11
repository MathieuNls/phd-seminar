<span>BIANCA</span> (Bug Insertion ANticipation by Clone Analysis at
merge time) is an approach that we propose and which aims to prevent the
insertion of bugs at commit-time. Many tools exist to prevent a
developer to ship <span>*bad*</span> code
[@Dangel2000; @Hovemeyer2007; @Moha2010] or to identify
<span>*bad*</span> code after executions (e.g in test or production
environment) [@Nayrolles; @Nayrolles2013a]. However, these tools rely on
metrics and rules to statically and/or dynamically identify sub-optimum
code. <span>BIANCA</span> is different than the approaches presented in
the previous sections because it mines and analyses the change patterns
in commits and matches them against past commits known to have
introduced a defect in the code (or that have just been replaced by
better implementation). Also, <span>BIANCA</span> is an offline approach
that is triggered by a merge request. When maintainers see that their
work are ready to be integrated with the main branch, they open a merge
request[^1]. Merging a task branch is not an instantaneous process as
the code need to pass code review. <span>BIANCA</span> leverages this
<span>*down*</span> time to perform a complete history check on all
projects contained in <span>BUMPER</span>.

Figure \[fig:bianca-approach\] presents an overview of our approach.

![The BIANCA Approach
\[fig:bianca-approach\]](media/bianca-approach.png)

<span>BIANCA</span> builds a model where each issue is represented by
three versions of the same file. These three versions are stored in
<span>BUMPER</span>. The first version $n$ is called the <span>*stable
state*</span> because the code of this version was used to fix an issue.
The $n-1$ version, however, is called the <span>*unstable state*</span>
as it was marked as containing an issue. Finally, the third version is
called the <span>*before state*</span> and represents the file before
the introduction of the bug. Hereafter, we refer to the <span>*before
state*</span> as $n-2$. <span>BIANCA</span> extracts the change patterns
form $n-2$ to $n-1$ and from $n-1$ to $n$. It aslo generates the changes
to go from $n-2$ to $n$.

When a developer commits new modifications, <span>BIANCA</span> extracts
the change pattern from the version $n_{dev}$ (current version) and
$n-1_{dev}$ (version before modification) of the developer’s source code
and compares this change patterns to known $n-2$ to $n-1$ patterns. If
$n_{dev}$ to $n-1_{dev}$ matches a $n-2$ to $n-1$ then it means that the
developer is inserting a known defect in the source code. In such a
case, <span>BIANCA</span> will propose the related $n-1$ to $n$ patterns
to the developer, so s/he could improve the source code and will show
the related $n-2$ for the $n$ pattern so the developer will learn how to
s/he should have modified the code in the first place.

Moreover, if the issue was previously reproduced by
<span>JCHARMING</span>, then <span>BIANCA</span> will display the steps
to reproduce it.

To extract the change patterns and compare them, we use the same
technique as the one presented in section \[sub:Extract and Save
Blocks\]. The third and the fourth normalizations are removing all
<span>*less*</span> important calls in the normalization one and two of
<span>RESEMBLE</span>. We classify a call as less-important if, for
example, it only does display-related functionalities such as generating
HTML or printing something to the console. Finally, the fourth
normalization will transform the code to an intermediate language of our
own that will allow us to compare source code implemented in different
programming languages.

Then, as in <span>RESEMBLE</span>, if the LCS is above a user-defined
threshold, then a warning is raised by <span>BIANCA</span> alerting the
developer that the commit is suspected to insert a defect. The given
defect is shown to the developer and can either force the commit if s/he
don’t find the warning relevant or abort the commit.

We believe that the warning, alongside the previously mined change
patterns and steps to reproduce the suspected default — provided by
<span>JCHARMING</span>, if available — will statisfy developers in terms
of actionable inteligence. Thus, <span>BIANCA</span> could succeed,
where other tools failed, at being used in industrial environment
[@Lewis2013].

Early experiments
=================

We have assessed the efficiency of <span>BIANCA</span> with the same
datasets we used to build our bug taxonomy proposed in Section
\[sec:data-collection\].

<span>@c|c|c|c|c@</span> **Dataset** & **Fixed Issues** & **Commit** &
**Files** & **Projects**\
Netbeans & 53,258 & 122,632 & 30,595 & 39\
Apache & 49,449 & 106,366 & 38,111 & 349\
Total & 102,707 & 229,153 & 68,809 & 388\

We ran two different experiments using the two first normalizations we
described in Section \[sub:Compare Extracted Blocks\]. Both experiments
consider only a few months of history, from April to August 2008. While
this could hinder the accuracy of our results, these five-month history
data contain 167,597 commits related to bug fixes. We believe that the
results to be representative.

The first experiment yields the result presented in Figure
\[fig:bianca-exp-1\].

![<span>BIANCA</span> warnings from April to August 2008 using the first
normalization. \[fig:bianca-exp-1\]](media/bianca-13.png)

With the first normalization, <span>BIANCA</span> raised 69,519 warnings
out of 167,597 (41.5%) analysed commits. Out of these 69,519, 13.4%
turned out to be false positives. A false positive is a healthy commit
that has been tagged as introducing a bug by <span>BIANCA</span>.
However, false positives have to be dealt with carefully in this study
as the commit might have introduced a bug but the bug has not been
reported yet.

In our second experiment, we used the second normalization and
<span>BIANCA</span> raised 83,627 warnings out of 167,597 (48.89%)
commit we analyze. However, the false positive rate increased to 21%.
Figure \[fig:bianca-exp-2\] shows the results.

![<span>BIANCA</span> warnings from April to August 2008 using the
second normalization. \[fig:bianca-exp-2\]](media/bianca-20.png)

<span>BIANCA</span> experiments are still in their early stages and we
are still trying to improve our normalization in order to reduce the
false positive rate.

[^1]: Also known as pull request
