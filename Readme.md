# RMarkdown with the IEEEtran style

This is an attempt at incorporating the IEEEtran.cls into RMarkdown.
I've used it to write a conference paper, though needed a bit of manual editing (see caveats).
I attempted to rip the skeleton (minus the paper) out of that setup, so it hasn't been extensively tested (and only the IEEEtran conference mode).

No guarantees made, and it's not necessarily going to be maintained. I'm a PhD student and hacking around with this sort of thing is a fun way to procrastinate :) [Let me know](mathematical.coffee@gmail.com) if you have any improvements/suggestions/revisions (e.g. make a github issue), or if you liked it :).

It is designed to compile to PDF only (though it is written in Markdown). You can probably try to compile to HTML but I make no guarantees (IEEEtran is a TeX style so it won't be styled similarly in HTML).
You can mix LaTeX into the markdown as with normal pandoc syntax (things inside the LaTeX will not be parsed by pandoc, however).


**Caveats**: if you want to give a label to an equation with pandoc-crossref it has to be its own paragraph, so you may have to edit the tex file and remove all the extra paragraph space if you don't like it.
ie to get an equation reference, you have to write:

```
Einstein's equation is

$$
e = m c^2,
$$ {#eq:einstein}

which is pretty amazing.
```

which ends up with the equation in its own paragraph in the TeX. While I think it looks fine and improves readability, much whitespace may be saved by going to the TeX and removing the blank lines which will set the equation as part of the paragraph rather than its own. (Only do this where it makes sense to, obviously).


# Configuration

You can just start modifying `example_conf.rmd` and everything will work out of the box.


## Document

By default we have a4 paper, and use the conference option to IEEEtran. If you want a draft, add `draftcls` too.

```
papersize: a4paper
classoption: conference # or eg [draftcls, conference]
```

Document comments with `%` will work for PDF output, but not for HTML, whereas HTML-style comments `<!-- -->` will work for them both.

## Authors

Set the author(s) in the YAML. The default format in `bare_conf.tex` is for author names and affiliations to be in multiple column layout for up to three different authors, where the affiliation is centred below each author. To do this, specify authors with `name` and `affiliation` in the YAML like below. If you need linebreaks in the name/author, put them in yourself or make one array element per line and we will put them in for you. The 'Email: ' line is surrounded in quotes because YAML does not like unprotected ':'.

```
author:
    - name: Michael Shell
      affiliation:
        - School of Electrical and
        - Computer Engineering
        - Georgia Institute of Technology
        - Atlanta, Georgia 30332--0250
    - name: Homer Simpson
      affiliation:
        - Twentieth Century Fox
        - Springfield, USA
    - name:
        - James Kirk
        - Montgomery Scott
      affiliation:
        - Starfleet Academy
        - San Francisco, California 96678--2391
        - 'Telephone: (800) 555--1212'
        - 'Fax: (888) 555--1212'
```

If you want the compact form (where each affiliation gets a symbol superscript on the author, good for more more than 3 authors or authors with multiple affiliations), you have to specify `affiliation` separately and manually match up the superscripts (it's not very elegant):

```
author:
    - name: Michael Shell
      affiliation: 1
    - name: ', Homer Simpson'
      affiliation: 2
    - name: ', James Kirk'
      affiliation: 3
    - name: '\ and John Doe'
      affiliation: [2,3]
affiliation:
    - key: 1
      name:
        - School of Electrical and Computer Engineering
        - Georgia Institute of Technology, Atlanta, Georgia 30332--0250
    - key: 2
      name:
        - Twentieth Century Fox, Springfield, USA
    - key: 3
      name:
        - Starfleet Academy, San Francisco, California 96678--2391
        - 'Telephone: (800) 555--1212, Fax: (888) 555--1212'
```

The author's name is listed, then the postmarks, then a space. If you want separators (commas, penultimate `\and`) you have to put them in yourself; this is because use of `\and` mucks up the alignment for the affiliation block. The commas etc have to go *preceding* the name, not after, because otherwise the postmark is placed directly after the `name` property. (I warned you it's not elegant).

## Abstract

Fill this out in the YAML header.

```
abstract: |
  Here is my abstract.
  It can go for lines and lines and lines.
```

## Bibliography and citations

Use `[@bibkey]` to put in a citation or `[@bibkey1, @bibkey2]` for multiple (note: though this will end up as `\citep{bibkey1, bibkey2}`, IEEE style renders this as `[bibkey1], [bibkey2]` not `[bibkey1, bibkey2]`).

I can't quite remember, but I think that `natbib:true` must be specified in the preamble or else the citations will be *manually* hard-coded in e.g. `{[}Author 1 (1993){]}` rather than using the `\cite{}` command. And now that we have `natbib:true`, pandoc will use `\citep{}`. However, the template doesn't *actually* load in natbib (couldn't find the IEEE style) so the template defines `\citep{}` to be `\cite{}`.

The title used for the bibliography section is "References" by default. You can configure it.

```
reference-section-title: References
link-citations: yes # citations have links to bibliography
```

As described
\IEEEtriggeratref{5}

## LaTeX preamble

In the rmarkdown config is a `includes: in_header` field where the path to the preamble may be specified. If you do not have one, then just delete it.

```
output:
  pdf_document:
    includes:
      in_header:
      - ./preamble.tex
```

## Pandoc-crossref

We use [pandoc-crossref](https://github.com/lierdakil/pandoc-crossref) for cross references.
We have suppressed the prefix for equation references (ie "(1)" instead of the default "eq. (1)") and also configured the figure, table, section prefixes (to full words instead of "fig.", "tbl.", "sec."). You can configure further in the YAML if you want.

```
# pandoc-crossref
eqnPrefix:
    - ''
    - ''
figPrefix:
  - "figure"
  - "figures"
tblPrefix:
  - "table"
  - "tables"
secPrefix:
  - "section"
  - "sections"
autoSectionLabels: true # prepends sec: to section titles
```

# Devnotes

## Files

* `lib/`: files not directly related to paper but needed to compile it
    -  `lib/IEEEtran.cls`: from [CTAN](https://www.ctan.org/tex-archive/macros/latex/contrib/IEEEtran/?lang=en). Version in this repo is 2015-08-26 v1.8b. Probably don't need this if you have IEEEtran installed through your LaTeX package manager.
    -  `lib/ieee-pandoc-template.tex`: pandoc template:
        * sets documentclass to `IEEEtran` by default (you can override in YAML metadata if you wish);
        * IEEE author block syntax for the authors and affiliations
    - `lib/bibtex`: bibtex files. I am only using IEEEabrv (I think)
* `bare_conf.rmd`: example of conference style.

## Check

* Use `\thebibliography` or similar, don't hard-code
* Citations use `\cite`, not hard-coded

## Todo

* Port `bare_jrnl`, `bare_conf_compsoc`, `bare_jrnl_compsoc`, `bare_jrnl_comsoc`, `bare_adv`, `bare_jrnl_transmag`
* A bunch of conditional packages - are they from pandoc template or from IEEEtran?
* Incorporate the rest of the IEEE bare_conf.tex header that is commented out, under YAML conditionals (IEEEpeerreviemaketitle, IEEEspecialpapernotice)
* Not sure if it's using my provided IEEEx or the ones installed on my system.
* At least allow HTML compiling (at the moment the preamble won't be loaded if you go to HTML)
