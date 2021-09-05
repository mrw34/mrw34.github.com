---
title: Jupyter Notebooks and reproducible data science
---
## Introduction

One of the ideas pitched by [Daniel Mietchen](https://twitter.com/EvoMRI) at the London [Open Research Data do-a-thon](https://github.com/sparcopen/Open-Research-doathon) for [Open Data Day 2017](http://opendataday.org) was to [analyse Jupyter Notebooks mentioned in PubMed Central](https://github.com/sparcopen/open-research-doathon/issues/25). This is potentially valuable exercise because these [notebooks](http://jupyter-notebook.readthedocs.io/en/latest/notebook.html) are an increasingly popular tool for documenting data science workflows used in research, and therefore play an important role in making the relevant analyses replicable. [Daniel Sanz](https://twitter.com/LifeDanielSanz) (DS) and I spent some time exploring this with help from Daniel (DM). We’re reasonably experienced Python developers, but definitely not Jupyter experts.

## Methodology

[Ross Mounce](https://twitter.com/rmounce) had kindly populated [a spreadsheet](https://docs.google.com/spreadsheets/d/1txg0u9zARHrLkY4MYuz5vmsVCZUOItgybEHqx13Bkbc/edit) with metadata for the [107 papers](http://europepmc.org/search?query=jupyter%20OR%20ipynb) returned by a query of Europe PMC for “jupyter OR ipynb”. My initial thought was that analysing the validity of the notebooks would simply involve searching the text of each article for a notebook reference, then downloading and executing it using [nbconvert](http://nbconvert.readthedocs.io/en/latest/install.html) in a headless sandbox. This could be turned into a service providing a badge for inclusion in the relevant repository’s README indicating whether the notebook was verified. It turned out that this was hopelessly naive, but we learnt a lot in discovering why.
Here’s what we discovered about how notebooks are being published:

- Our full-text search for “Jupyter” or “ipynb” returned several false positives, where the mention referred to external resources rather than notebooks directly used in preparation of the article. This reflects the fact that there’s no standardised way of directly attaching assets such as Jupyter notebooks. We ignored the relevant articles - they are annotated as “n/a” in the spreadsheet.
- This issue of attachment can also make it time-consuming to identify and retrieve notebooks, even if there is one of direct relevance. In some cases, they were referenced by URL in the abstract or the body of the text. In others, they were attached as supplementary data, albeit as text rather than ipynb files, presumably due to limitations in the filetypes that the publisher’s system allowed. In one case merely a snapshot of the notebook was included as an image. This issue further limits the potential for automation.
- Genuine references to notebooks were typically links to [nbviewer](http://nbviewer.jupyter.org). This is a service developed by the Jupyter project that provides a read-only view of any public notebook. It *renders* rather than *executes* notebooks i.e. it displays cell outputs cached within the notebook before its publication. This means that nbviewer does not verify whether the notebook is actually in an executable state. The most obvious way to perform this verification is to follow the nbviewer link, click “Download”, and then load it into a local Jupyter installation.

We both tried downloading and executing a small sample of notebooks from our list. We had very limited success, so DS continued to systematically try other notebooks on the list and document the range of errors encountered (see the [“Problem” column](https://docs.google.com/spreadsheets/d/1txg0u9zARHrLkY4MYuz5vmsVCZUOItgybEHqx13Bkbc/edit)), whilst I took a [particularly complex notebook](http://nbviewer.jupyter.org/github/maayanlab/Zika-RNAseq-Pipeline/blob/master/Zika.ipynb) (from [this article](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4972086/)) and attempted to troubleshoot it to completion. By doing this, we learnt the following about Jupyter:

- There’s no means for notebooks to directly express the dependencies they require. Most assume the availability of packages required by the [SciPy stack](https://www.scipy.org/stackspec.html), but if the end user has (for example) installed Jupyter using pip then these packages won’t necessarily be available. Some notebooks did embed a list of requirements in free text, but didn’t make it clear in terms of the exact package names and/or versions. Others provided a requirements.txt file in their source repository, but again didn’t make it clear that this file existed. This is a particular problem when the article links directly to nbviewer rather than to the notebook alongside related files in its repository. This approach to linking also causes import statements to fail when they reference local, unpublished packages, that appear alongside the notebook in the repository. In this case, the notebook must be run from a repository checkout.
- There’s nothing to stop notebooks [accessing the system shell](http://ipython.readthedocs.io/en/stable/interactive/reference.html#system-shell-access), and therefore making assumptions about the underlying platform. This obviously limits their portability, especially when authored on MacOS or Linux and executed on Windows.
- It seems that older versions of the notebook format didn’t require the runtime kernel to be specified, and that Jupyter’s auto-update process indiscriminately inserts a specifier for the current kernel. In one of our examples, this led to Python 2 code being run by a Python 3 interpreter, and immediately failing.
- Notebooks can assume the availability of non-Python software being available on the local system. Even if this is unambiguously specified up-front (typically in free text) the software may not be freely available, or available at all for the relevant runtime platform (or even the specific distribution). This may be unavoidable, but we mention it here as a general challenge to reproducibility - automated or otherwise.

Some authors have taken the laudable approach (and extra effort) of bundling their notebook(s) inside containers, typically using [Docker](https://www.docker.com). This is very effective in implicitly documenting, via a Dockerfile, the required runtime platform, system packages, Python environment and even third-party software. This is also the [approach taken](https://github.com/MaayanLab/Zika-RNAseq-Pipeline/blob/master/Dockerfile) by the complex notebook we analysed. We tried to build and run the relevant container from scratch, which demonstrated the following:

- The Dockerfile refers to and downloads a couple of tarballs from third-party sources. This is perfectly acceptable, especially as some dependencies may ultimately not be redistributable, and it reduces the size of the source repository, but does assume that these links remain valid in the long term.
- The container was not immediately rebuildable due to a missing file (```subread-1.4.6-p2-Linux-x86_64.tar.gz```). It was possible to search the web for this and retrieve it manually. The container could then be rebuilt and run, and the provided notebook executed within the provided Jupyter server (based on the SciPy stack, as required).
- The container dynamically downloaded many gigabytes of vial sequence. Again, this is not problematic in itself, but an automated notebook execution/verification service would have to consider caching this data for efficiency.
- The notebook did not run to completion as it assumed the presence of data files for a reference genome (```genomes/Homo_sapiens/UCSC/hg19```) that were missing. It wasn’t obvious from where this data could be retrieved.
- It wasn’t clear whether the reviewers of the article had encountered these issues, and if so how they resolved them.

This whole exercise was hugely valuable in improving our understanding of Jupyter and of the challenges facing notebook authors. Based on our experiences, and acknowledging that non-trivial analyses will depend on potentially large numbers of dependencies and amounts of source data, we recommend taking the following approach to publishing notebooks:

- If a notebook depends on external files then link to repositories rather than directly to rendered notebooks
- Obtain a DOI for your repository, and use this link consistently. Don’t variously link to the repository (e.g. on GitHub), the notebook (e.g. on nbviewer) and any published container (e.g. on Docker Hub).
- Attempt to limit the use of “shelling out” from notebooks: wherever practicable err on the side of not making assumptions about the runtime platform.
- Take a judicious approach to bundling vs retrieving dependencies on-demand
- Provide a container definition, and ideally publish the container itself, with clear instructions for its execution
- Ensure that containers can be rebuilt - this is key for end-to-end reproducibility
- If you require a large amount of data, then make it clear where this should be retrieved from, and provide clear instructions on how to mount this from the local drive to the container (so that it is effectively cached on the host system)
- Ensure that you double-check that your notebook/container can be run on a clean system.

## Next steps

This was a one-day project, which leaves lots of potential work to be done. Here are some ideas:

- Considering the time available we decided to focus on breadth rather than depth, i.e. to check as many notebooks as possible, with a very high bar for success: the notebook in question needed to run unmodified, without installing any packages that weren’t explicitly listed (with appropriate installation instructions) in the preamble. However, the required modules could sometimes be inferred by the (failing) import statements in the code. A second pass over the notebooks would be sensible, using pip/conda to install any obvious prerequisites. There is a good chance that this would lead to more successful tests. A third pass could focus on running notebooks from a git checkout, so that any other source files were available. However, these attempts would not excuse cases where this requirement was not made clear by the notebook itself.
- After performing these two iterations it could be valuable to analyse the remaining failures in order to discover the most frequent problems, and propose solutions.
- Propose “[10 Simple Rules](http://collections.plos.org/ten-simple-rules)” for publishing workflows as Jupyter notebooks.
- Our experience has only increased the justification for automatic verification of notebooks - ideally using a Jupyter-specific CI system.

## Conclusions

- Technologies such as Jupyter and Docker present great opportunities to make digital research more reproducible, and authors who adopt them should be applauded.
- These technologies do not, by themselves, ensure replicability. This was demonstrated by the fact that we were able to successfully execute only one of the ~25 notebooks that we downloaded.
- Care must be taken when creating and publishing the resultant assets (notebooks, containers etc). Publishers could assist with this, and with validation. Reviewers should also take verification into account.

## Acknowledgements

Many thanks to [SPARC](https://sparcopen.org) and the [NIH](https://www.nih.gov) for funding the event at which we performed this investigation, and Daniel Mietchen and Ross Mounce for their advice and guidance.

## Notes

This investigation was very much a collaborative effort. Any errors or omissions in this write-up are my own.

## Further reading
- [Reproducible Data Analysis in Jupyter](http://jakevdp.github.io/blog/2017/03/03/reproducible-data-analysis-in-jupyter/)
- [Reproducible Computational Workflows with Continuous Analysis](http://biorxiv.org/content/early/2016/08/11/056473)
- [Is software reproducibility possible and practical?](https://danielskatzblog.wordpress.com/2017/02/07/is-software-reproducibility-possible-and-practical/)

## Updates

07 March 2017: A [repository demonstrating automatic verification](https://gitlab.com/mwoodbri/jupyter-ci/tree/master) via CI is now available.
