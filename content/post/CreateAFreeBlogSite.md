---
title: "Create a Free Web/Blog Site Using Github Pages and Hugo"
date: 2020-08-26T13:09:42-06:00
draft: true
image: "images/github-pagesLightBlue.jpeg"
tags: []
categories: []
---

Want to create your own web site, but don't want to get tied into any of the big hosted solutions or subscription publishing sites? This article describes how to use Github Pages with Hugo to create, maintain, and host your own web site.
<!--more-->

## Overview

> Prerequisites:
>
> 1. A basic knowledge of git and Github. For significant customizations beyond what's available natively in Hugo and the chosen Hugo theme, a basic understanding of HTML and CSS will be required.
> 2. Git version 2.24.3 or higher
> 3. Mac development machine
>    1. Windows and Linux are also supported. Much of what's described here is applicable to both of these. But the exact steps I followed were performed on a Mac.

This article is a fairly detailed tutorial on how to create a full featured website/blog using Hugo and Github Pages. If you just want a quick overview of how to use Hugo I recommend going to the [Hugo Quickstart page](https://gohugo.io/getting-started/quick-start/). If you want a quickstart on setting up a Hugo/Github Pages site I recommend going to the [Hugo Github Pages quickstart page](https://gohugo.io/hosting-and-deployment/hosting-on-github/).

At its core Github Pages is a combination of a Github repository and a published web site. One consequence of this is that any commit to the master branch of the repository is immediately published. One way to create a Github Pages website is to also create an associated "workspace" Github respository. This is what is described here.

Hugo is an Open Source static website generator. It is quite powerful and the websites that can be created with Hugo can be quite sophisticated. Hugo is the base engine used to interpret how to create a website based on a predefined template. Hugo calls these templates themes. Skins are another word for themes. A theme is a combination of HTML and CSS. The HTML is used for site layout. CSS is used to define colors, fonts, font size, and other non-layout related attributes that define how the website looks. This combination is quite powerful. The website's page definitions themselves, i.e., the website's content, are defined using standard Markdown syntax.

Hugo also comes with an HTML server that can be used during development to provide immediate feedback on any changes that are made to web pages. Hugo will also generate a complete static website based on a theme's layout, CSS, and the markdown files that define the content. ***TALK ABOUT HUGO DEV VS. GENERATE VS PUBLISH MODES?***

The Hugo themes as well as the website's page definitions are stored in what I above called the "workspace" repository. This is the primary authoring space. The Hugo web server operates against this repository. Through the magic of Git Submodules, the final set of committed changes to the master branch in the workspace respository are immediately made available in the Github Pages respository.

> Brief segue on [Git Submodules](https://github.blog/2016-02-01-working-with-submodules/):
>
> Git Submodules are a way of including one, **hosted**, Github respository in another, **hosting**, repository. The entire content of the hosted respository is available in the hosting repository as if it was just a normal subdirectory in the hosting respository. Changes to the hosted repository can be made in the context of the hosting respository. Committing any such changes are immediately available in the hosted respository.

In the context of Submodules, the Github Pages repository is the **hosted** repository. It contains the published content of the website. In contrast, the respository containing the Hugo theme(s) and the development versions of the content is the **hosting** repository. When the website content is ready for publishing Hugo is used to generate the static website. Hugo generates the content into the submodule associated with the, **hosted**, Github Pages respository. When ready, the changes are committed in the **hosting** respository and they become immediately available, and thereby published, in the Github Pages respository.

## Initial Github setup

The two respositories, the Github Pages respository and the hosting development respository, need to be created. There are 2 options for the Github Pages respository:

1. Personal or Organizational
2. Project

See the [Github Pages help documentation](https://help.github.com/articles/user-organization-and-project-pages/#user--organization-pages) for more details about the differences. This article describes how to create a Personal Github Pages respository.

Creating the Github Pages respository is fairly straightforward, but there is one very important restriction on its name. The respository's name must be of the form `<github_username>.github.io`. For example, `youngkin.github.io`. This will be the URL of the website. So the Github pages respository is just a normal Github respository, just with a very specific name. See the [Github Pages quickstart guide](https://pages.github.com/) if you need more details on how to create a respository.

The second step is to create the respository where the website development/authoring will take place. This is just a normal Github respository. See the [Github documentation](https://docs.github.com/en/github/creating-cloning-and-archiving-repositories/creating-a-new-repository) if you need more details about how to create a respository.

## Install Hugo

There are a number of ways to install Hugo:

* There are binary releases available on the [Hugo downloads page](https://github.com/gohugoio/hugo/releases).
* On a Mac using Homebrew is the simplest, `brew install hugo`.
* On Linux, with Homebrew installed, `brew install hugo`.
* There are a couple of ways to install on Windows using Chocolatey:
  * choco install hugo[-extended] -confirm
  * scoop install hugo[-extended]

Docker images are also available. For more details on installing Hugo see the [Hugo documentation](https://gohugo.io/getting-started/installing/).

## Create a basic site

***Specify names for the 2 github repos and use those names in the rest of the doc just to keep things easy to understand?***

### Initialize the new site

The first thing to do is to clone the development/authoring respository on to your local machine.

With the Github respositories setup and Hugo installed it's time to initialze the website. Hugo provides this capability through the `hugo` command line:

1. Change directories into the root of the development/authoring respository
2. Run `hugo new site .`. 
   1. `hugo new site` is telling Hugo to create a new site. The final parameter in the command, the `.`, is telling Hugo which directory to create the new site. In this case we're going to create it in our development/authoring respository.

`hugo new site <dirname>` creates the following directory structure:

```
site_root
  |--layout
  |--content
  |--static
  |--data
  |--themes
  |--config.toml
  +--archetypes
```

The directories and file that we're interested in for this article are:

* `content` - this is where content under development is located, i.e., the various markdown files for a site's pages.
* `static` - content that doesn't change, such as images
* `themes` - Where the site's theme(s) are located
* `config.toml` - The basic configuration for the site, e.g., the site's name
* `archetypes` - Template(s) used by the `hugo new <contentdir/contentfile.md>` command, used to create new content markdown files

### Pick and install the site's theme

The next step is to choose a theme to use. See the [Hugo themes page](https://themes.gohugo.io/) and select the one that looks best to you. The Hugo [Ananke theme](https://themes.gohugo.io/gohugo-theme-ananke/) is a popular one. I chose the [Clean White theme](https://themes.gohugo.io/hugo-theme-cleanwhite/). Once you've picked a theme you need to add it to the development/authoring repo. Like with the Github Pages repo this is accomplished via a git submodule:

```
git submodule add <theme's-github-url> <target-directory>
```

I chose a customized version of the Clean White theme:

```
git submodule add git@github.com:youngkin/hugo-theme-cleanwhite-rsy.git themes/hugo-theme-cleanwhite-rsy
```

The `<target-directory>` should be a subdirectory of `themes`. `themes` is where Hugo expects loaded themes to be located.

> I'll discuss my reasons for customizing the Clean White theme, as well as describe what I did, later in this article.

Some basic customization of a Hugo generated website is specified in the `config.toml` file. For now we'll just tell Hugo to use our chosen theme.

```
echo 'theme = "<theme-name>"' >> config.toml
```

In my case I used

```
echo 'theme = "hugo-theme-cleanwhite-rsy"' >> config.toml
```

The `theme-name` is from the name of the directory where you placed the theme.

### Test the new site locally

Now it's time to create some content and test everything we've done so far. The first step is to create some content:

```
hugo new post/testpage.md
```

The new page will be located at `content/post/testpage.md`, `post` being the parent directory given in the previous command. This command places all content in the `content` directory. `content` is also the directory Hugo uses as the source of content when generating the static web site. The file will look something like this:

```markdown
---
title: "Testpage"
date: 2020-08-27T13:43:09-06:00
draft: true
---

```

You can inspect the generated file. The generated content comes from the `archetypes/default.md` file which is the template used for all new content. The content between the `---` lines is metadata for the page. Text after the 2nd `---` line is the body of the web page.

The next step is to test the installation by starting the Hugo web server and navigating to the development site:

```
hugo server -D
```

Near the bottom of the logged output you'll see a line like this:

```
Web Server is available at http://localhost:1313/ (bind address 127.0.0.1)
```

Copy the URL into a browser and you should see the home page of the new site. Depending on the theme chosen, you should see something like this:

<img style="border:2px solid black" src="/images/My_New_Hugo_Site.png" align="left"><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br>

## Authoring a site

Now we're ready to start creating the actual site that we want to publish. The first thing you might want to do is delete the `content/post/testpage.md` file created previously. This was created just to test the installation and is no longer needed.

### Configuration

As mentioned above, the basic configuration for a Hugo website is specified in the `config.toml` file. Hugo's [Getting Started documentation](https://gohugo.io/getting-started/configuration/) covers configuration in detail. Here is what a basic, but functional, configuration looks like:

```
baseurl = "https://<your-github-id-here>.github.io"
title = "Rich Youngkin"
theme = "hugo-theme-cleanwhite-rsy"
languageCode = "en-us"
paginate = 5000 #frontpage pagination

[markup]
  [markup.goldmark]
    [markup.goldmark.renderer]
      unsafe = true

[params]
  header_image = "images/main_go_wideandtall_darklayer.jpeg"
  title = "Software Engineering"
  slogan = "$ grep -rni \"The how's and why's\" ."
  SEOTitle = "Rich Youngkin Blog"
  keyword = "Rich, Youngkin, Raspberry, Pi, Docker, Kubernetes, Go, Golang, Microservice"
 
  # Sidebar settings
  sidebar_about_description = "Father, Husband, Skier, Cyclist, and, oh yeah, Software Developer"
  sidebar_avatar = "images/MeAtJackson.jpeg"
  about_me = true

  featured_tags = true 
  featured_condition_size = 2 # How many posts have to have a given tag to be featured (greater-than)

  image_404 = "images/404-bg.jpg"
  title_404 = "We couldn't find what you were looking for..."

  omit_categories = false

  [[params.addtional_menus]]
  title =  "ABOUT"
  href =  "/top/about/"

  [params.social]
  rss            = true
  email          = "rich.youngkin@gmail.com"
  linkedin       = "https://www.linkedin.com/in/richard-youngkin-0749763"
  github         = "https://github.com/youngkin"
  medium         = "https://medium.com/@RichYoungkin"
  stackoverflow  = "https://stackoverflow.com/users/2646870/rich"
  reddit         = "https://www.reddit.com/user/elevation5280"
 ```

 Taken line-by-line, here's what each one means:

 * `baseurl` - This is the URL of the published web site. For a Github Pages site it must be of the form `https://<your-github-id-here>.github.io`, e.g., `https://youngkin.github.io`.
 * `title` - This is the text displayed in the top left, header, area of every page on the site. It is a button control that will take you to the home page. Typically it's your name.
 * `theme` - The theme used by the site
 * `language` - The [IETF language tag](https://en.wikipedia.org/wiki/IETF_language_tag) specifying the primary language of the website. The language code, e.g., `en`, can have a suffix specifying an optional country code or region to subdivide the language into various dialects. `en-us` means United States English. `en-gb` means Great Britian English. 
 * `paginate` - how many blog entry summaries to display on the home page
 * `[markup]` - this is a section that specifies the Markup/down syntax used. It's optional. The Goldmark renderer is used by default. Blackfriday was the previous renderer and can still be used if specified. Goldmark has more capability than Blackfriday, but by default it won't render HTML tags due to security concerns. I wanted to use HTML tags in my pages, primarily for image scaling and placement. I had originally swapped out Goldmark for Blackfriday but later decided that I might like the extra functionality provided by Goldmark. To get the HTML tag support I needed, I had to add this `[markup]` section and set `unsafe = true`. See the [Hugo Configure Markup document](https://gohugo.io/getting-started/configuration-markup) for more details.
 * `[params]` - These are the basic parameter settings available
   * `header_image` - the background image to use as the default in the headers of all pages. It is the only image that will be used for the home and about pages
   * `title` - This is the text used in the title, or banner, displayed at the top of every web page.
   * `slogan` - This is a subtitle displayed just below the title in the header of all web pages
   * `SEOTitle` & `keyword` - These have something to do with search engine optimization. I included them in the hope that this would make my site more visible. More [information can be found in this tutorial](https://moonbooth.com/hugo/seo/).
   * `sidebar_about_description`, `sidebar_avatar`, & `about_me` - The sidebar is a concise "about me" entry. You can use the description to provide a short bio and the avatar to provide an image. `about_me` controls whether the sidebar is shown.
   * `featured_tags` & `featured_condition_size` - These are used to specify whether you want to display tags used to describe pages to show up in a tag cloud on each page. `feature_condition_size` specifies the minimum number of articles, that must have a given tag before that tag shows up in the tag cloud. It is important to note that tags will only be displayed when `feature_condition_size + 1` articles with a given tag exist.
   * `image_404` & `title_404` - These specify how a 404 Not Found page looks. One is the image, the other is the textual title.
   * `omit_categories` - Categories are like a higher level tag. For example, New Mexico, Colorado, Utah, Wyoming, Idaho, and Montana might all be tags for articles pertaining to those U.S. states. There might be a category "Best Ski Areas in the World" that could be used to group those tags into a higher level grouping. Defined Categories may be included in the navigation bar at the top of each page. `omit_categories` controls whether categories will be displayed in the top navigation bar.
   * `[[params.additional_menus]]` - This allows the specification of additional menu items to be put in the top navigation bar, for example an "About" page.
   * `[params.social]` - This entire sub-section is used to put badges associated with different social media sites on the Hugo generated site. There are other social media sites available such as twitter and facebook.

### Archetypes

Archetypes are templates used by Hugo when new web pages are created via the  `hugo new <some-post-file>` command. They are useful because there is a metadata section at the top of each page describing some basic characteristics like the page title, description, and date. Without archetypes this metadata would have to be manually typed or copied on each new web page. While Hugo installs a default archetype at `archetypes/default.md`, you'll probably want to create your own. I have a file located at `archetypes/post.md`. The name `post.md` is meaningful to Hugo. It is the archetype Hugo will use, if present, when generating posts. There are other archetype file names and locations that are meaningful to Hugo. You can find more detail about archetypes at the [Hugo Archetypes](https://gohugo.io/content-management/archetypes/) page.

My `archetypes/post.md` file looks like this:

```markdown
---
title: "{{ replace .Name "-" " " | title }}"
#description: <descriptive text here>
date: {{ .Date }}
draft: true
image: ""
tags: []
categories: []
---

# Descriptive text here...
<!--more-->
```

Here's what each line means:

* `---` - these lines delineate the metadata section of the page
* `title: "{{ replace .Name "-" " " | title }}"` - specifies that the title will be replaced with the file name given to the new page after replacing dashes ('-') with spaces. This is a quick, handy, way to generate titles automatically. For example, `My-New-Post.md` becomes "My New Post" in the page's title.
* `description` - specifies text to be used in an article's summary on the home page. This text will not be displayed in the article. It is related to the `# Descriptive text here...` line a little lower in the file.
* `date` - replaced with the date the post was created. It can be changed if you want to modify it, for example after updating the page. There are more sophisticated things you can do with dates, such as published date, last modified date, expiration date, etc. Those all have special meanings. See the [Hugo documentation](https://gohugo.io/getting-started/configuration/#configure-dates) for more details.
* `draft` - specifies whether a page is in draft or publish state. Pages in draft state can be seen when viewing the site running on the Hugo web server, but they will not be visible from a published web site such as Github Pages.
* `image` - specifies the image to display at the top of the page. If not specified, the `header_image` in `config.toml` will be used.
* `tags` - specifies a low level subject grouping. Taking my example from above, U.S. states might be an appropriate subject grouping for pages pertaining to U.S. states.
* `categories` - specifies a high level subject grouping. Again, taking my example from above, some U.S. states can be put in the category of "Best Ski Areas in the World".
* `# Descriptive text here...` & `<!--more-->` - These are used as an alternative to the `description` line in the metadata section. This can be used if the amount of descriptive text desired is more than can be comfortably put in the `description` line. `<~--more..>` is used to indicate the end of this form of descriptive text. Unlike `description` above, this text does appear in the article. An alternative to both the `description` line and these lines is to not specify either. In this case the first 70 words of the article will be used in the article summary.

### Writing

Now that we have our respositories setup, Hugo installed, our initialized Hugo development repository, and a basic understanding of Hugo website and page configuration, we can start authoring content. All content is written using Markup syntax. See the [Hugo Markdown Guide](https://www.markdownguide.org/tools/hugo/) for more information and syntax details. As noted in the Configuration section above, using the Goldmark `unsafe` option allows you to use standard HTML tags in the content if desired.

## Publishing content to Github Pages

At some point you're going to be ready to make your site live. To do this you'll want to generate the static content to be published from the development content. This development content is generally found in the `content` and `static` directories that were created when the site was initialized. The output of the generation process is generally placed in the `public` directory, but this can be modified. For our purposes we're just going to let Hugo take this default behavior.

### Linking the development respository to the Github Pages respository

Before you can publish to your Github Pages site, you need to link your development respository to your Github Pages respository, `<your-github-account-name>.github.io`. You only need to do this once. To do the linking we'll use Git Submodules as discussed previously. The command to use is:

```bash
git submodule add -b master git@<your-github-account-name>.github.io.git public
```

In my case I used:

```bash
git submodule add -b master git@youngkin.github.io.git public
```

The important parts of the command are:

* `-b master` - This links to the hosted repository's `master` branch
* `public` - This links the hosting, i.e., this, repository's `public` directory with the hosted respository. So any changes in the `public` directory, when committed, will be pushed to the Github Pages respository, `<your-github-account-name>.github.io.git`.

To illustrate the linking of the development respository to the Github Pages respository do the following:

```bash
git remote -v
```

The output should look something like this:

```bash
origin	https://github.com/<your-github-account-name>/<your-github-account-name>.github.io.git (fetch)
origin	https://github.com/<your-github-account-name>/<your-github-account-name>.github.io.git (push)
```

In my environment it looks like this:

```bash
origin	https://github.com/youngkin/youngkin.github.io.git (fetch)
origin	https://github.com/youngkin/youngkin.github.io.git (push)
```

### Publishing the site

When you're ready to publish the site you'll need to generate the static content. This is done by running:

```bash
hugo -D
```

As mentioned above, the default behavior is to generate the content into the `public` directory, the directory we linked to the Github Pages respository. At this point there's only one more thing left to do, we need to go through the usual `git add ...`, `git commit ...`, and `git push origin master` steps to publish to Github Pages.

## The finished product

This web site! The source code is available on Github for:

* [The blog source code](https://github.com/youngkin/hugoblog)
* [The modified Clean White theme](https://github.com/youngkin/hugo-theme-cleanwhite-rsy.)
* [The published Github Pages repo](https://github.com/youngkin/youngkin.github.io)

## References
* https://www.markdownguide.org/tools/hugo/
* https://gohugo.io/hosting-and-deployment/hosting-on-github/ 
* https://gohugo.io/getting-started/ 
* https://guides.github.com/features/pages/ 
* https://themes.gohugo.io/ ## Customizations

I used the [Clean White theme](https://themes.gohugo.io/hugo-theme-cleanwhite/). I wanted a different look for:

* Title and Description (looking for "green screen" feel)
* Tags - too dim, wanted to darken
* Wanted to change some fonts

Here's what I changed:

* I created my own "fork" of the [Clean White theme](https://github.com/youngkin/hugo-theme-cleanwhite-rsy)
  * I modified `themes/hugo-theme-cleanwhite-rsy/static/css/hux-blog.min.css` 
      * **Note:** I "pretty printed" this file with the Atom `css-clean` package to make it easier to visualize in this article, but it needs to be unchanged in the blog source in order for the web site to render properly

### Change the Blog Header:

![Blog Header](/images/BlogHeader.png)

"Green screen" look for blog title, change `color: #00FA9A`

* Snippet:

```css
.intro-header .page-heading,
.intro-header .post-heading,
.intro-header .site-heading {
  padding : 85px 0 55px;
  color   : #00FA9A;
}
```

"Green screen" for Navigation bar at top of page, change `color: #00FA9A`

* Snippet:

```css
  .navbar-custom .navbar-brand {
    padding     : 20px;
    color       : #00FA9A;
    line-height : 20px;
  }

  ...

    .navbar-custom .nav li a {
    padding : 20px;
    color   : #00FA9A;
  }
```

Terminal font and yellow color for blog subheading (slogan), change `color: #FFD700` and `font-size: 25px`, Add `font-family: Courier New, monospace;`

* Snippet:

```css
.intro-header .page-heading .subheading,
.intro-header .site-heading .subheading {
  display     : block;
  margin      : 10px 0 0;
  color       : #FFD700;
  font-family : -apple-system,
                "Helvetica Neue",
                Arial,
                "PingFang SC",
                "Hiragino Sans GB",
                STHeiti,
                "Microsoft YaHei",
                "Microsoft JhengHei",
                "Source Han Sans SC",
                "Noto Sans CJK SC",
                "Source Han Sans CN",
                "Noto Sans SC",
                "Source Han Sans TC",
                "Noto Sans CJK TC",
                "WenQuanYi Micro Hei",
                SimSun,
                sans-serif;
  font-family : Courier New,
                monospace;
  font-size   : 25px;
  font-weight : 300;
  line-height : 1.7;
  line-height : 1.1;
}
```

### Change the Sidebar

<img style="border:2px solid black" src="/images/BlogSideBar.png" align="left"><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br>

Get a darker gray for the "About Me" description and the "social" badges background color, change  `.sidebar-container {...}`, and `.sidebar-container a {...}` `color` element to `#555555`
* Snippet:

```css
.sidebar-container {
  color     : #555555;
  font-size : 14px;
}

.sidebar-container h5 {
  padding-bottom : 1em;
  color          : gray;
}

.sidebar-container h5 a {
  color           : gray!important;
  text-decoration : none;
}

.sidebar-container a {
  color : #555555!important;
}
```

Get a darker gray for "Featured Tags" in the sidebar to improve readability `.sidebar-container .tags a {...}` `color` element to `#555555`, `.tags a {... color: ... border:...}` 
* Snippet:

```css
.sidebar-container .tags a {
  border-color : #555555;
}

...

.tags a {
  display         : inline-block;
  margin          : 0 1px;
  margin-bottom   : 6px;
  padding         : 0 10px;
  color           : #FFD700;
  border          : 1px solid rgba(255,215,0,0.8);
  border-radius   : 999em;
  text-decoration : none;
  font-size       : 12px;
  line-height     : 24px;
}

```

## GIMP

I used [GIMP](https://www.gimp.org/) to manipulate the blog images to:

* Resize the images to fit within the page
* Add a dark layer over the top of the images to increase contrast with the blog header elements (i.e., navigation bar, title, subtitle)
  * Layer colors to darken images - 4d4d4d and 373737 and generally an opacity of 80%

