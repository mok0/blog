+++
title = "Tips for Hugo"
date = 2024-06-13T22:18:00+02:00
tags = ["emacs", "org-mode", "hugo"]
categories = ["Hugo"]
draft = false
author = "Morten Kjeldgaard"
description = "My Hugo tips"
keywords = "hugo ox-hugo emacs"
+++

Tips and tricks on Hugo, Emacs, Org-mode and `ox-hugo` I discovered while developing this site, in random order.

<!--more-->

<div class="ox-hugo-toc toc has-section-numbers">

<div class="heading">Table of Contents</div>

- <span class="section-num">1</span> [Emacs Org Mode support in Hugo](#emacs-org-mode-support-in-hugo)
    - [Hugo's translation of Org mode files not working](#hugo-s-translation-of-org-mode-files-not-working)
    - [Advantage of using `ox-hugo` over Hugo's Org translation](#advantage-of-using-ox-hugo-over-hugo-s-org-translation)
- <span class="section-num">2</span> [Highlight of code and other markup settings](#highlight-of-code-and-other-markup-settings)
- <span class="section-num">3</span> [Allow HTML in files](#allow-html-in-files)
    - [Literal export in Org mode files](#literal-export-in-org-mode-files)
- <span class="section-num">4</span> [Set Hugo code block theme](#set-hugo-code-block-theme)
- <span class="section-num">5</span> [Add metatags for SEO](#add-metatags-for-seo)
    - [Keywords](#keywords)
    - [Description](#description)
    - [How to insert description and keyword metatags with ox-hugo](#how-to-insert-description-and-keyword-metatags-with-ox-hugo)

</div>
<!--endtoc-->


## <span class="section-num">1</span> Emacs Org Mode support in Hugo {#emacs-org-mode-support-in-hugo}

If your content format is Emacs Org Mode, you may provide front matter using Org Mode keywords.
For example:

```org
#+TITLE: Example
#+DATE: 2024-02-02T04:14:54-08:00
#+DRAFT: false
#+AUTHOR: John Smith
#+GENRES: mystery
#+GENRES: romance
#+TAGS: red
#+TAGS: blue
#+WEIGHT: 10
```

Note that you can also specify array elements on a single line:

```org
 #+TAGS[]: red blue
```

which is now the preferred way, multiline tags are deprecated.

When using the Emacs Org Mode content format, use a `# more` divider to indicate the end of the content summary.

Check out the Hugo manual [here](https://gohugo.io/content-management/front-matter/#emacs-org-mode).


### Hugo's translation of Org mode files not working {#hugo-s-translation-of-org-mode-files-not-working}

Hugo can translate Org files but the dates are messed up, Hugo can't interpret the `#+DATE:` field as written by Emacs.  I have given up to figure out how it works and I am going back to `ox-hugo`.

These are the fields you need to tag for `ox-hugo`, they are different from the tags Hugo uses when translating Org files.

```org
#+HUGO_BASE_DIR: ../
#+HUGO_SECTION: ./posts
#+HUGO_WEIGHT: 2001
#+HUGO_AUTO_SET_LASTMOD: t
#+HUGO_DESCRIPTION: "A description of this page"
#+HUGO_TAGS: hugo org
#+HUGO_CATEGORIES: Emacs
#+HUGO_DRAFT: false
#+HUGO_MENU: :menu "main" :weight 20
#+HUGO_CUSTOM_FRONT_MATTER: :foo bar :baz zoo :alpha 1 :beta "two words"
```

There is a problem with `ox-hugo` when it exports author, in the toml frontmatter it outputs:

```toml
author = ["Morten Kjeldgaard"]
```

but it needs to be:

```toml
author = "Morten Kjeldgaard"
```

therefore I also  have to specify no export of author, AND export the author verbatim, like this:

```org
#+OPTIONS: author:nil
#+HUGO_CUSTOM_FRONT_MATTER: :author "Morten Kjeldgaard"
```


### Advantage of using `ox-hugo` over Hugo's Org translation {#advantage-of-using-ox-hugo-over-hugo-s-org-translation}

The <span class="underline">advantage</span> of using `ox-hugo` is that document remains a pristine Org-mode document, with proper Org-mode header lines, but most importantly, without Hugo patches to the format. The most notable is the end-of-summary marker, which in Hugo's org is `#more`, but that line will appear if you export your Org document to other formats. Instead, use this:

```org
#+html: <!--more-->
```

The <span class="underline">drawback</span> of using `ox-hugo` is that there's an extra step exporting the document to _markdown_, usually with a `C-c C-e H h` key combo. However, this can be skipped if the `org-hugo-auto-export-mode` is set, this can be done on a per file basis by entering as the last lines of the document:

```org
# Local Variables:
# eval: (org-hugo-auto-export-mode)
# End:
```

or it can be activated globally in `init.el`.


## <span class="section-num">2</span> Highlight of code and other markup settings {#highlight-of-code-and-other-markup-settings}

```toml
[markup]
  [markup.highlight]
    anchorLineNos = false
    codeFences = true
    guessSyntax = false
    hl_Lines = ''
    hl_inline = false
    lineAnchors = ''
    lineNoStart = 1
    lineNos = false
    lineNumbersInTable = true
    noClasses = true  # false turns off code highlighting
    noHl = false
    style = 'monokai'
    tabWidth = 4
```


## <span class="section-num">3</span> Allow HTML in files {#allow-html-in-files}

To allow the export of raw HTML in content files, the following has to be in the site's `config.toml` file:

```toml
 [markup.goldmark.renderer]
  unsafe = true # Allow HTML in md files

```


### Literal export in Org mode files {#literal-export-in-org-mode-files}

A single line of raw HTML can be exported like this:

```org
#+HTML: <p>Literal HTML code for export</p>
```

Larger blocks can be embedded in an HTML block:

```org
#+BEGIN_EXPORT html
  <p><i>All lines between these markers are exported literally</i></p>
#+END_EXPORT
```

Below is an example of this:

<hr />

<p>Literal HTML code for export</p>

<p><i>All lines between these markers are exported literally</i><p>

<hr />

Check out the Org manual [here](https://orgmode.org/manual/Quoting-HTML-tags.html).


## <span class="section-num">4</span> Set Hugo code block theme {#set-hugo-code-block-theme}

Hugo uses the Chroma highlighter engine[^fn:1]. The default theme is apparently monokai, but it is too much contrast for my site with super black code blocks. I found the list of themes [here](https://github.com/alecthomas/chroma/tree/master/styles) and chose `monokailight` which looks much nicer. The code theme is defined in `config.toml` thus:

```toml
 pygmentsstyle = "monokailight"
```


## <span class="section-num">5</span> Add metatags for SEO {#add-metatags-for-seo}

Found these suggestions [here](https://djangocas.dev/blog/hugo/tips-on-hugo-seo/).

Metadata is used by browsers (how to display content or reload page), search engines (keywords), and other web services. The &lt;meta&gt; tag defines metadata about an HTML document. &lt;meta&gt; tags always go inside the &lt;head&gt; element, and are typically used to specify character set charset, page description, keywords, author of the document, and viewport settings.


### Keywords {#keywords}

Use keywords meta tag to provide a list of search keywords of current the page.  This is done in Hugo by adding the keywords meta tag in the `partials/head.html`  template:

```hugo
{{ with .Param.Keywords }}
   <meta name="keywords" content="{{ delimit . "," }}">
{{ end }}
```

Then in _markdown_ use keywords to define a list of keywords:

```yaml
---
title: Tips on Hugo SEO
keywords:
- hugo
- seo
---
```


### Description {#description}

Use `description` tag to provide a short description of the page, this adds a  description meta tag to allow search engine used in the snippet shown in search results in some situations.

```hugo
{{ with .Param.Description }}
   <meta name="keywords" content="{{ . }}">
{{ end }}
```


### How to insert description and keyword metatags with ox-hugo {#how-to-insert-description-and-keyword-metatags-with-ox-hugo}

Unfortunately, there are no `#+HUGO_DESCRIPTION` or  `#+HUGO_KEYWORDS` tags for `ox-hugo`, but the following works:

```org
  #+HUGO_CUSTOM_FRONT_MATTER: :description My Hugo tips
  #+HUGO_CUSTOM_FRONT_MATTER: :keywords hugo ox-hugo emacs
```

When the right code in `=partials/head.html`, this becomes strings in the markdown front matter and eventually gets translated into `meta` tags:

```html
 <meta name="description" content="My Hugo tips">
 <meta name="keywords" content="hugo, ox-hugo, emacs">
```

[^fn:1]: Look here: <https://gohugo.io/content-management/syntax-highlighting/>
