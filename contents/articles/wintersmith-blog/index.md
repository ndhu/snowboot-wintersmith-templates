title: Wintersmith notes
abstract: Moving my Blog to Wintersmith. A quick introduction to Wintersmith, setup of environment in sublime text, primer on how to customize the wintersmith generator using jade templates.
author: Andy Hulstkamp
date: 2012-10-10 12:20
template: article.jade
tags: tools
type: post
status: published

My blog is running on an outdated version of wordpress - an update is desperately needed. Since updating a customized WP-Version can be a pain and I don't really need the bunch of features WP offers, I decided to switch to a static site generator. I had a look at a couple of options and settled for Wintersmith. It is straightforward to setup and easy to customize.

These are my notes on how I customized my blog using Wintersmith.

<span class="more"></span>

### Getting started

Setting up Wintersmith is easy. [Check the guide](https://github.com/jnordberg/wintersmith) to get started.

Once Wintersmith is set up, invoke...

```dos
	cd [working-directory]
	wintersmith preview
```

...in order to see the changes while editing blog entries or customizing templates.

### Customizing

Customization in Wintersmith revolves around

  + Jade-templates
  + page meta-data
  + configuration files
  + the content tree

#### Custom Templates & Meta-Data

Wintersmith uses [Jade](http://jade-lang.com/) for templates.

Layout.jade is the main template of my Blog. It is my basic template, where all blocks and default content is specified - more specific templates inherit from layout.jade to
provide custom LAF on specific pages.

Each page in Wintersmith specifies which template to use for rendering.
For instance, at the top of the index.md page for this blog entry, I've got  

<pre>
  <code class="no-highlight">
    title: Wintersmith notes
    abstract: Some abstract...
    author: Andy Hulstkamp
    date: 2012-10-10 12:20
    template: article.jade
    tags: tools
    type: post
    status: published
  </code>
</pre>

which tells wintersmith to use article.jade to render this page. The template will also get access to all the other variables like title, tags etc. I specified here.

article.jade is just

<pre>
  <code class="no-highlight">
extends layout
block content
  a.back(href='/') &larr; BACK TO INDEX
  br
  br
  h2
    = page.title
  article.single.post
    section.content
      != page.html
    br
    a.back(href='/') &larr; BACK TO INDEX
  </code>
</pre>

it ihnerits from layout.jade that defines the header-, navigation- and footer structure. 

When a page is rendered, the template gets a reference to the _page_ instance. The page is available as _page_ in the template context. The content tree is also available as _contents_ and the config.locals object as _locals_. underscore.js is available as ___ to provide some utility functions. Custom variables specified on the page may be accessed by _page.metadata_.

Page-Title and Page-Content are accessed in a template like this
```
h2
  = page.title
  article.single.post
    section.content
      != page.html
```

#### Navigation & Configuration Files

An important file is _config.json_. This is where I put global data like the data for my navigation

```Json
{
  "locals": {
    "url": "http://localhost:8080",
    "name": "Andy Hulstkamp",
    "title": "Andy Hulstkamps blog",
    "owner": "Andy Hulstkamp",
    "description": "stuff",
    "index_articles": 5,
    "nav" : [
      {
        "name": "Home",
        "cssClass": "home",
        "description": "Home Page",
        "url": "/"
      },
      {
        "name": "Blog",
        "cssClass": "blog",
        "description": "Blog",
        "url": "/blog"
      },
      {
        "name": "About",
        "cssClass": "about",
        "description": "About ",
        "url": "/about"
      }
    ]  
  }
}
```

Data provided in config.json can be accessed in templates via _locals_. 
I use the _nav_ array specified in config.json to create the navigation by looping over the entries in a template.

<pre>
  <code class="no-highlight">
ul.nav.link-list
  each item in locals.nav
    li
      span.pikto 
        =item.cssClass
      a(href=item.url)= item.name
  </code>
</pre>

#### Content Tree

The content tree is a graph of all json-, page- or fileobjects in the contents folders (and subfolders).

For instance
```javascript
contents.tags['spring.json']
```
holds the reference to a file named _spring.json_ within the _tags_-folder in the _contents_ directory.

if _spring.json_ is something like...

```json
{
  "id": "spring",
  "name": "Spring",
  "cssClass": "spring",
  "description": "Spring JEE Framework"
}
```

...I can access the cssClass property like

```javascript
contents.tags['spring.json'].cssClass
```

...which is how I did the tags for the site.

At the top of each page I added a list of string to tag a page:

```json
	tags: flex, java, junit
```

In a template I can fetch this metadata through _page.metdata_ and using underscore to turn it into an array.
I can then loop over the array and build a list.

<pre>
  <code class="no-highlight">
  - var tagsarray = _.toArray(page.metadata.tags.split(','))
        ul
          each tag in tagsarray
              li= tag
  </code>
</pre>

#### Mixin and data references

I can also reference other variables using the tag as a key to load data from the content tree

<pre>
  <code class="no-highlight">
 div.tagindex
    h1 TAGINDEX
    ul.tagindex
      - for (var k in tags)
        li.tag 
          mixin tag(k)
          ul
            - var groupedValues = _.values(tags[k])
            - for (var ki in groupedValues)
              li.tagged.row
                h4.title
                 a(href=groupedValues[ki].obj.url)= groupedValues[ki].title
                p.abstract.seven.column= groupedValues[ki].abstract
  </code>
</pre>

This loops over an array of tags and uses a mixin to access data in the content tree...

<pre>
  <code class="no-highlight">
mixin tag(tagName)
  - var tag = contents.tags[tagName + '.json']
    if tag
      span.pikto J
        a(href=tagName)
          = tag.metadata.name
    else
      = tagName
  </code>
</pre>

A mixin is simply defined in a seperate jade template file. This one here is defined in a file called tag.jade under the templates folder.

A .json file in the content tree is dynamically accessed using the tagName. 
For instance within my contents/tags folder I have a file ux.json, that holds some data in context to the ux tagName

```JSON
{
  "id": "ux",
  "name": "UX, Usability, Interface Design",
  "cssClass": "ux",
  "description": "all user-centric things"
}
```

The mixin looks up the data in this file and renders some stuff accordingly.

### Good to know

#### Directories, Filenames, Html-Content

Again, the content tree is just the tree of json-, page- and fileobjects under the contents folder.

<pre>
  <code class="no-highlight">
 - var directories = _.map(contents.articles._.directories, function(item) {
  -                                    return item
  - })
  </code>
</pre>

will get all directories. 

Loop over directories:

<pre>
  <code class="no-highlight">
  ul
  each dir in directories
    li
      strong dir.filename (dir): 
      span #{dir.filename}
      br
      strong  dir.index.filename: 
      span #{dir.index.filename}
      br
      strong dir.index.metadata.tags
      span #{dir.index.metadata.tags}
  </code>
</pre>

include html from page in a template:


<pre>
  <code class="no-highlight">
!= page.html
  </code>
</pre>

#### extending blocks in templates 

Any block defined in a template can be overwritten or elements can be appended or prepended.
For the detail page I need specific javascript and styles, that can be linked in the header
by appending to the head block of the parent template.

<pre>
  <code class="no-highlight">
block append head
  link(rel='stylesheet', href="stylesheets/ahimpr.css")
  script(type='text/javascript', src='js/ah.js')
  script(type='text/javascript', src='js/modernizr.js')
  script(type="text/javascript")
    $(document).ready(function () {
      AHUI.setupDiashow($(".photobox"),$(".keyControlHolder"));
    });
  </code>
</pre>

#### Use Mixins for repeated customized content ####

I have a couple of older Flash-stuff I wanted to move over to the static site.
Here's a way to deal with the swf-objects includes, that are required for flash content.

Mixin defined in parent template

<pre>
  <code class="no-highlight">
extends layout
mixin useFlash(file)
  style
    html, body  { height:100%; }
    body { 
      margin:0; padding:0; overflow:auto; text-align:left; 
      background-color: #fafafa; 
    }
    #flashContent { display:none; 
    }
  script(type="text/javascript", src="swfobject.js")
  script(type="text/javascript", src="history/history.js")
  link(rel="stylesheet", type="text/css", href="history/history.css")
  script
    var swfVersionStr = "10.0.0";
    <!-- To use express install, set to playerProductInstall.swf, otherwise the empty string. -->
    var xiSwfUrlStr = "playerProductInstall.swf";
    var flashvars = {};
    var params = {};
    params.quality = "high";
    params.bgcolor = "#fafafa";
    params.allowscriptaccess = "sameDomain";
    params.allowfullscreen = "true";
    var attributes = {};
    attributes.id = "#{file}";
    attributes.name = "#{file}";
    attributes.align = "middle";
    swfobject.embedSWF("#{file}.swf", "flashContent", "100%", "100%", swfVersionStr, xiSwfUrlStr, flashvars, params, attributes);
    <!-- JavaScript enabled so display the flashContent div in case it is not replaced with a swf object. -->
    swfobject.createCSS("#flashContent", "display:block;text-align:left;");
block append head
 mixin useFlash(page.metadata.swf)
block content
  article.post
    section.content!= page.html
    a.back(href='./') &larr; BACK TO POST
  </code>
</pre>

we can pass objects defined on the page to the mixin, therefore generalizing repeated parametrized content. 

### Toubleshooting

#### Migration

While moving html pages from my old blog to wintersmith, simply coping and pasting the html structures into
wintersmith might cause trouble. Sometimes jade would just ignore script or link tags. This could usually be
corrected by reintending the tags properly.

#### Speeding up builds

Wintersmith builds a content tree based on the files located in the contents folder (and subfolders).
While developing templates or writing new articles it might be useful to temporarely move old articles 
out of the contents folders and back in when you choose to make the final build.

#### Updates

Updates on config.json are not visible in wintersmith preview. A restart of wintersmith preview is necessary.

### Tooling

I use Sublime Text 2 for blog entries. To setup a build process put 

```JSON
{
    "cmd": ["wintersmith.cmd", "build", "-v"],
    "path": "c:/data/blog-2012",
    "working_dir": "{project_path}"  
}
```
under Tools/Build System/New Build System.

This will invoke wintersmith build in the my blog directory.

#### Firefox auto reload addon

Tools/AutoReload Preferences/ 

-> create http://localhost:8080/
-> add when local file changes -> add directory where the blog structure lies

### Jade Key Point

[jade-blocks]:https://github.com/visionmedia/jade#template-inheritance

+ Templating engine inspired by HAML  (Javscript, Node)
+ Define default template in layout.jade
+ layout.jade defines the default structure using default blocks for header, head, content etc [see jade blocks][jade-blocks]
+ custom layouts created by extending layout.jade and prepending, appending or replacing the block elements in layout.jade
  +  _block header_ to replace the header block
  +  _block append header_ to append to the header block
  + _block prepend header_ to prepend to the header block
+ Each content page refers to a layout, set at the top of each content page (template: article.jade)
+ look at article.jade, it extends layout.jade and replaces title, header and content blocks.
+ supports vanilla javascript. In Wintersmith underscore _ is provided as well.

  
