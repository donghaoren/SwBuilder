SwBuilder - Static Website Builder
====

SwBuilder (Static Website Builder) is a SConstruct script to build static webpages from page contents, partials and templates.
It works mainly by substituting page contents and partials into templates,
so you don't need to update every page when you modify your template or partials.

* Author: Donghao Ren
* License: GPLv3

Features
----

* Blog support.
* Markdown support.
* Javascript/CSS minify.
* YAML metadata.
* Mustache templates.
* Various small utilities.

Concepts
----

* Templates: HTML templates are style files for your website. They are
  incomplete HTML files with placeholders for content, partials and other stuff.
    - File extension: `.template` or `.blog_template`.
    - Place content: `{{content}}`.
    - Place partials: `{{partial: partial_name}}`.
* Partials: Contents that are common to many pages, for example sidebars, menus.
* Metadata: Key-Value pairs (nesting is allowed) of strings.
  Including global metadata and per-page metadata. User defined metadata are allowed. For example:
    - `build_time`: Time the build script run.
    - `build_time_iso`: ISO format of `build_time`.
    - `blog`: Special entry for blog pages.

Documentation
----

To use SwBuilder, install the following dependencies first:

* SCons
* Python Modules:
  * json, yaml: JSON/YAML support.
  * pystache: Mustache rendering.
  * pytz: For timezone definitions.
  * PIL: For image manipulation.
* Commandline Tools:
  * LaTeX: For LaTeX rendering support.
  * ImageMagick: For image manipulation.
  * multimarkdown: Markdown rendering.
  * uglifyjs, cleancss: JS/CSS minification.

Not all the dependencies are required, it depends on what feature you use.

### Getting Started

1. Put the SConstruct file in the directory of your website sources.
   The output files will be generated in the `deploy` folder, and temporary files in the `temp` folder.

2. Create templates along with CSS files and Javascript files.

3. Write the `SConstruct` file to define your webpages.
   Remember that this is a python file, so feel free to use other python expressions to ease your configuration.

4. Run `scons` to build your website.

### Creating Templates

A template is just a text file with extension `template` or `blog_template`.
You need to define the place to put page content by writing `{{content}}` in the template.

For example:

    <!DOCTYPE html>
    <html>
    <head>
      <!-- Use {{metadata-key}} to reference page metadata. -->
      <title>{{title}} - My Website</title>
    </head>
    <body>
      <h1>{{title}}</h1>
      <!-- Put page content here -->
      {{content}}
    </body>
    </html>

Each page has its contents and metadata, you can use `{{key}}` to refer them in your template or pages.

### Creating Pages

A page can be defined by a html file or a markdown file.

For example a html file:

    <!--{
    title: Page Title
    key: value
    nested:
      key: value
    }--->

    <p>This is the page content</p>
    <!-- yields 'value' -->
    <p>{{nested.key}}</p>

    <!-- Use {{ref: path}} to reference another file. -->
    <a href="{{ref: folder/page.html}}">link</a>
    <img src="{{ref: folder/image.png}}" />

    <!-- Use {{js: path}} to include Javascript -->
    <script type="text/javascript" src="{{js: script.js}}"></script>

Use `<!--{ yaml contents }-->` to define YAML metadata for the page, it can be referred from the page or the templates.

If your website is not directly under a domain, it's difficult to reference pages from a template.
Static Website Builder provides a useful `{{ref: path}}` directive, as well as `{{js: path}}`, they will be replaced with
the actual path (relative to current page) in the build process. For example, `{{ref: a.html}}` will be replaced as `a.html` in `b.html`, but `../a.html` in `folder/b.html`.

### SConstruct

First you should import SwBuilder. Copy `site_scons` to your project,
and write the following code at the beginning of your SConstruct.

    from SwBuilder import *

Note that the targets files are relative to the `deploy` folder.

Define a partial:

    Partial("name", "file.html")
    Partial("name", "file.md")

Define a page:

    Page("target.html", "source.html", "mytemplate.template")
    Page("target.html", "source.md", "mytemplate.template")
    Page("target.html", "source.html", "mytemplate.template", title = "mytitle")

Define an image or a binary file:

    Image("target.png", "source.png")
    Binary("target.png", "source.png")

Use TargetList and Find to defined targets by wildcard matching.

    TargetList(Image, "images", Find("*.jpg", "images"))

Define a concatenated and minified Javascript/CSS file:

    Javascript("target.js", ["source1.js", "source2.js"])
    CSS("target.css", ["source1.css", "source2.css"])

### Blog Support

To define blog posts, first call BlogInit:

    BlogInit("blog") # Desired blog directory: 'blog'

Then, pass definition of tags as an yaml:

    - name: Tag Display Name
      key: tag_key

Call BlogTags to load the yaml:

    BlogTags("blog/tags.yaml")

Then call BlogPost to add blog posts:

    BlogPost("blog/article.md")
    # Use FindFiles to add all matched files conveniently.
    SourceList(BlogPost, FindFiles("blog/*.md"))

Each blog post should contain yaml metadata as follows:

    title: Blog Title
    blog:
        permlink: permanent-link-of-the-blog
        summary: short summary, not required.
        date: 2009-01-01 10:12:14 Asia/Shanghai
        tags: [ tag1, tah2 ]

Next, you may add images for blog:

    TargetList(Image, "blog/images", Find("*.jpg", "blog/images"))
    TargetList(Image, "blog/images", Find("*.png", "blog/images"))

Call BlogFinalize when all the article files are added.

    BlogFinalize()

Finally, generate articles and lists:

    # Generate articles with given template:
    BlogGenerateArticles("template.blog_template")

    # Generate article list with given template and list template.
    # Mustache templated should be used in blog/index.html to render lists.
    BlogGenerateList("blog/index.html", "template.blog_template")
    BlogGenerateTags("blog/index.html", "template.blog_template")

### Math Support

Use HTML comments to create math formulas:

<div data-hljs-lang="none"></div>

    <!-- math: latex_code -->: in a $ ... $ environment.
    <!-- lmath: latex_code --> : in a displaymath environment.
