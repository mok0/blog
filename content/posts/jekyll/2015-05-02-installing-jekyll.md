+++
title = "Installing Jekyll"
date = 2015-05-02T20:56:00+02:00
tags = ["jekyll", "ruby", "gem", "python3"]
categories = ["Jekyll"]
draft = false
author = "Morten Kjeldgaard"
+++

If you are using Python 3 as a standard, the [Jekyll](http://jekyllrb.com) installation from [homebrew](http://brew.sh) might give you problems, because the Jekyll installation out-of-the box uses the syntax higlighter [Pygments](http://pygments.org), which does not work under Python 3. # more

This is the rather mysterious error I got:

```bash
$ jekyll serve --trace
Configuration file: /Users/mok/github/mok0.github.io/_config.yml
            Source: /Users/mok/github/mok0.github.io
       Destination: /Users/mok/github/mok0.github.io/_site
      Generating...
  Liquid Exception: No header received back. in _posts/2015-05-02-welcome-to-jekyll.markdown
```

Instead, use the Ruby highlighter `rouge`. In `_config.yml` add:

```yml
highlighter: rouge
```

and install the rouge highlighter gem:

```shell
gem install rouge
```

Now the installation should go without problems.
