+++
title = "Using ox-Hugo"
author = ["zhi"]
date = 2026-01-17T00:00:00-05:00
tags = ["ox-hugo"]
categories = ["tutorial"]
type = "posts"
draft = false
weight = 1003
+++

[Hugo](https://gohugo.io/) is a static website generator written with GO, providing easily
capability to convert **markdown** files into a **website**. For those who use
[emacs](https://www.gnu.org/software/emacs/) [Org mode](https://orgmode.org/), one can use [ox-hugo](https://ox-hugo.scripter.co/) to convert Org files into markdown files
that will be compatible with Hugo. Hence the workflow is:

\\[\mathrm{Org} \\\ \xrightarrow{\text{ox-hugo}} \\\ \mathrm{markdown} \\\ \xrightarrow{\text{Hugo}}  \\\ \mathrm{HTML}\\]

For me, I use it to create my research journal and personal website.

Here I describe different ways of writing things **in Org mode** that can be
re-interpreted by `ox-hugo` to generate the corresponding `md` files that
can be used by `Hugo` for my own records.


## Setup {#setup}

There are two types of [blogging workflow](https://ox-hugo.scripter.co/doc/blogging-flow/) using `ox-hugo`.

1.  One Org file per post
2.  One Org subtree per post

It is generally preferred to do it the second way,
i.e. use one Org file for multiple posts where we have
one Org subtree per post.

Now `Hugo` introduces the idea of [page bundles](https://gohugo.io/content-management/page-bundles/) for better content management.
There are two types:

1.  leaf bundle: uses the name _index.md_ to store the text
2.  branch bundle: ues the name _\_index.md_ to store the text

See [here](https://ox-hugo.scripter.co/doc/hugo-bundle/) to see how `ox-hugo` exports files as bundles.

My personal workflow is to use branch bundle as _sections_ and leaf bundle for
specific entries. For example, my typical structure goes like:

```nil
content-org/
├── posts/
│   ├── _index.md
│   ├── post1/
│   │   ├── index.md
│   │   └── img1.png
│   └── post2/
│       ├── index.md
│       └── img2.png
└── about/
    └── _index.md
```

where I use branch bundle, using _\_index.md_, to symbolize a section.
And then use leaf bundles, using _index.md_, for each individual post.


## Typographical Emphasis and Horizontal Line {#typographical-emphasis-and-horizontal-line}

Different text formatting are available

1.  **bold text** via `* *`
2.  _italic text_ via `/ /`
3.  ~~crossout text~~ via `+ +`
4.  `highlight text` via `~ ~` or `= =`

A horizontal line can be created for formatting via
5 consecutive dashes.

```nil
-----
```

which looks like

---


## Footnote {#footnote}

You can create footnotes via the same [Org-mode](https://orgmode.org/manual/Creating-Footnotes.html) syntax.
Put the footnote as `[fn:#]` where # is the footnote number.
Then create the definition of footnote separately via
`[fn:#] footnote definition`[^fn:1]


## Quotes and Code {#quotes-and-code}

You can create a quote block via

```nil
#+begin_quote
This is a greate quote by me.

  -- Me
#+end_quote
```

which looks like

> This is a greate quote by me.
>
> -- Me

You can also create code blocks via

```nil
#+begin_src language
Some language code
```

\#+end_src

For example python, we can do

```python
import numpy as np

class Cat:
    def __init__(self, name, numWhiskers, weight):
        self.name = name
        self.numWhiskers = numWhiskers
        self.weight = weight

    def talk(self):
        print("Meow!")

myCat = Cat("Nian", 20, 10)
myCat.talk()
```


## Hyperlink and Online Image {#hyperlink-and-online-image}

See [here](https://ox-hugo.scripter.co/doc/image-links/) for more information.

`ox-hugo` converts hyperlinks using the standard
[Org mode hyperlink syntax](https://orgmode.org/guide/Hyperlinks.html):

```nil
[[LINK][DESCRIPTION]]
```

The `DESCRIPTION` part is optional.
If it is omitted, the `LINK` itself is displayed on the website.
If both `LINK` and `DESCRIPTION` are provided, the website displays
`DESCRIPTION`, and clicking on it navigates to `LINK`.
The `DESCRIPTION` can be plain text or an image.

For example, here is a hyperlinked text to [wikipedia](https://www.wikipedia.org/).

To display image on the website, we typically want to format it somehow.
The common syntax are the following:

```nil
#+NAME: IMG_NAME
#+ATTR_HTML: :width 100%
#+CAPTION: Here is a caption
[[IMG_LINK][DISPLAY_IMG_LINK]]
```

`#+NAME` is the name used to reference the image via `[[IMG_NAME]]`.
`#+ATTR_HTML` sets the attributes for html. See [here](https://orgmode.org/worg/org-tutorials/images-and-xhtml-export.html) for more info.
And the main one to use is `:width`, which controls the width of the
image displayed. Lastly we have `#+CAPTION`, which writes the caption
under the displayed image. Again, `DISPLAY_IMG_LINK` is optional,
and if it is omitted, image from `IMG_LINK` gets displayed.
If it is not omitted, then the website shows image from `DISPLAY_IMG_LINK`,
and clicking on it navigates to `IMG_LINK`.

For example, Figure [1](#figure--fig:carina) is an image via an external link.

<a id="figure--fig:carina"></a>

{{< figure src="https://cdn.esawebb.org/archives/images/wallpaper4/weic2205a.jpg" caption="<span class=\"figure-number\">Figure 1: </span>Carina Nebula by James Webb Telescope" width="100%" >}}

Figure [2](#figure--fig:pillar) is where both `IMG_LINK` and `DISPLAY_IMG_LINK` are
set to the external image link. Hence clicking on this image
navigates to the original website.

<a id="figure--fig:pillar"></a>

{{< figure src="https://cdn.esawebb.org/archives/images/wallpaper5/pillarsofcreation_composite.jpg" caption="<span class=\"figure-number\">Figure 2: </span>Pillar of Creation by James Webb Telescope" width="100%" link="https://cdn.esawebb.org/archives/images/wallpaper5/pillarsofcreation_composite.jpg" >}}


## Local Images and Videos {#local-images-and-videos}

We often want to display local images instead of online images.
Hence we need to link the local images.
Let's first discuss how `Hugo` handles it, and then talk about
how `ox-hugo` can be integrated into the workflow along with Org mode.

`Hugo` generally has two options for storing images that will
be accessible and linked by `md` files stored in `content/`.
See [here](https://github.com/gohugoio/hugo/issues/1240#issuecomment-753077529) for some good explanation.

1.  Store all images in `static/` directory.
2.  Use [leaf bundles](https://gohugo.io/content-management/page-bundles/#leaf-bundles) and store images within the directory that contains
    the `index.md` that links the images.

By default, the html files that `Hugo` generates live in the `public/`
directory, and that serves as the root directory.
`Hugo` also copies all subdirectories that lives in `static/` to
`public/`, so the _conventional_ way of storing images is to
store them in `static/`, or perhaps create a subdirectory, say `images/`
and put images there. Suppose the `Hugo` base directory is named as _hugo_,
then the structure goes like

```nil
hugo/
├── other subdirs
└── static/
    └── images/
        └── img1.png
```

Then the image can be accessed via the following (see [here](https://ox-hugo.scripter.co/doc/image-links/#references-to-files-in-the-static-directory) for more information),
which is to basically access via _absolute path_ after the website is built,
where it is assumed that the `public/` directory acts as the _root directory_ for
the website.

```nil
[[/images/img1.png]]
```

i.e.

{{< figure src="/images/nian.jpg" caption="<span class=\"figure-number\">Figure 3: </span>This is Nian Nian" width="50%" >}}

This has two major downsides:

1.  Images for every posts lives here. You can imagine it being difficult to manage.
    Although you can create subdirectories to hold images for each post.
2.  For Org mode users who uses ox-hugo, the hyperlinked image cannot be opened
    or accessed within the org file. This is because the link is neither a valid
    absolute link nor relative to where org file lives.

Now the second way of accessing images, using [leaf bundles](https://gohugo.io/content-management/page-bundles/#leaf-bundles) is my preferred way.
A _leaf bundle_ is basically a post that uses the name _index.md_ to store the text.
Now we can store images in the same directory as _index.md_ and then we will be
able to access it via _relative link_.

For example, if the structure goes like:

```nil
hugo/
├── other subdirs
└── content/
    └── posts/
        └── post1/
            └── index.md
            └── img1.png
```

Then we can access it in _index.md_ via

```nil
![image](img1.png)
```

However, for Org mode users, we typically only want to work with `content-org/`
directory, and have `ox-hugo` to auto generate everything to `content/`.
This means that we wish to work with a structure like:

```nil
hugo/
├── other subdirs
├── content
└── content-org/
    └── posts/
        └── post.org
        └── post1/
            └── img1.png
```

where `post.org` holds all possible post entries. Suppose that we have
a single post named as `post1`, and it uses image `img1.png`. Now we can access
this image in the Org file using the relative link.

```nil
[[file:post1/img1.png]]
```

However, `Hugo` won't recognize this since `img1.png` does not belong to the same
directory as _index.md_ which will be auto generated by `ox-hugo`.
To solve this issue, `ox-hugo` has a unique feature which will auto copy local attachments
with extensions listed in `org-hugo-external-file-extensions-allowed-for-copying`.
(see [here](https://ox-hugo.scripter.co/doc/images-in-content/) for more information).
For example, I have

```elisp
(setq org-hugo-external-file-extensions-allowed-for-copying
    '("JPG" "PNG" "JPEG" "GIF" "SVG" "PDF" "MP4"
      "jpg" "png" "jpeg" "gif" "svg" "pdf" "mp4"))
```

Now if the Org post is _leaf bundle_, i.e. for the Org subtree,
we set the output file name to be _index.md_ and set the directory where
the bundle lives, i.e. _post1_.

```nil
:EXPORT_FILE_NAME: index
:EXPORT_HUGO_BUNDLE: post1
```

Now we can simply reference the image as following,
and `img1.png` will then be auto-copied to `content/post/post1/img1.png`.

```nil
[[file:post1/img1.png]]
```

i.e.

{{< figure src="nian1.JPG" caption="<span class=\"figure-number\">Figure 4: </span>Nian Nian raising her hand (feet)" width="50%" >}}

**Note** that using the prefix `file:` is needed to use local
link for `DISPLAY_IMG_LINK` for local image files in order
to properly display the image. It is optional for `IMG_LINK`
but it doesn't hurt. So generally speaking, always start
your local image link with `file:`.


## Math Expressions and LaTeX {#math-expressions-and-latex}

There are different ways to write math expressions. See [here](https://ox-hugo.scripter.co/doc/equations/) for more info.

1.  **Inline math** via `$ EXPRESSION $`, e.g. $ e=mc^2$
2.  **Math block** via `$$ EXPRESSION $$`, e.g.
    \\[ i\hbar \frac{\partial}{\partial t} \Psi(\mathbf{r}, t) = \hat{H} \Psi(\mathbf{r}, t) \\]
3.  Math block via **LaTex**:
    ```latex
    \begin{equation}
      \label{eq:euler} \tag{1}
      \frac{\partial }{\partial t}\left( \rho \vec{U} \right) + \nabla \cdot \left(\rho \vec{U} \otimes \vec{U} \right) + \nabla p = 0
    \end{equation}
    ```
    producing:

\begin{equation}
\label{eq:euler}
\frac{\partial}{\partial t}\left( \rho \vec{U} \right) + \nabla \cdot \left(\rho \vec{U} \otimes \vec{U} \right) + \nabla p = 0
\end{equation}

If you wish to reference the equation, you should use LaTeX environment and
you can reference it via `\ref{label}`, where `label` is the label you set via `\label{}`.
For example, here is Eq. \ref{eq:euler}.

**Note** that to get this working, the `ams` tag needs to be present
in your **MathJax** setting, which is typically stored in `math.html`.
My `math.html` looks like:

```html
<script id="MathJax-script" async src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-chtml.js"></script>
<script>
  MathJax = {
      tex: {
          tags: 'ams',
          displayMath: [['\\[', '\\]'], ['$$', '$$']],  // block
          inlineMath: [['\\(', '\\)'], ['$', '$']]      // inline
    }
  };
</script>
```

To preview compiled LaTeX within Org mode, one can use command `C-x C-l`.

[^fn:1]: The footnote definition appears at the bottom of the page.
