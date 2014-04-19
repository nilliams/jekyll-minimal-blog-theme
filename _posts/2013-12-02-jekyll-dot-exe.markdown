---
layout: post
title: Building Jekyll.exe for Windows
subtitle: A portable Jekyll executable, thanks to OCRA
date: 2013-12-02 09:00:00
tags:
 - Ruby
 - Static Sites
 - Windows
---

The result is a single, zero-install (2.6mb) `jekyll.exe` file you can keep in your Dropbox or on a USB stick. This saves you the [potentially painful installation](http://www.madhur.co.in/blog/2011/09/01/runningjekyllwindows.html) that is otherwise required on Windows (i.e. setup of Ruby, DevKit & RubyGems). You can download jekyll.exe [here](https://github.com/nilliams/jekyll.exe).

> Note: This build currently doesn't provide 'pygments' highlighting. You must disable `pygments` in your site's `_config.yml`. See [this section](#pygments) for syntax highlighting alternatives.

The remainder of this post details how it was built. It's written step-by-step and is something of a catalogue of errors. If you want the short answer, skip to [the recap](#recap).

### Execute

The pain of getting Jekyll working on Windows got me wondering about the state of 'script-to-exe' tools in Ruby. I wasn't optimistic as I've tried similar things in Python before and the tooling wasn't great (presumably due to lack of demand). I've also played around with Ruby's [Shoes](http://shoesrb.com) toolkit which all but *rolls its own platform* for easy deployment (and I still had problems on Windows).

So I was pretty surprised when a few hours later I had a (basically) working `jekyll.exe` sat on my desktop. A bit more experimenting later and now it's hopefully ready enough that other people should find it useful. So here's how to build a portable Jekyll.

> Note: This is not to be confused with [this 'portable' Jekyll for Windows](http://www.madhur.co.in/blog/2013/07/20/buildportablejekyll.html) which clocks in at an unfortunate 216mb. This is a totally different approach which leads to the lighter, aforementioned **2.6**mb executable.

### OCRA

Enter the unassuming [OCRA](http://ocra.rubyforge.org/) (One-Click Ruby Application) package. It lets you turn a Ruby script into an executable like so:

{% prism bash %}
$ gem install ocra
$ echo 'print "Hello World!"' > hello.rb
$ ocra hello.rb

$ hello.exe
Hello World!
{% endprism %}

> Note: I'm using [MSYS](http://www.mingw.org/wiki/MSYS) for a Unix-like shell in Windows, which is why my commands look bash-like. But all of this should work in Command Prompt (just use backslashes)! I also explicity write `jekyll.exe` everywhere I execute jekyll for my own assurance I'm not accidently running my global jekyll - it will of course work fine without the `.exe`.

So let's try running OCRA on Jekyll.

{% prism bash %}
$ gem unpack jekyll
$ ocra jekyll-1.2.0/bin/jekyll

...
=== Finished building jekyll.exe (2149622 bytes)
{% endprism %}

It spits out an executable, but I'm not expecting it to be that easy. I'm not surprised when I try it out and it errors:

{% prism bash %}
$ jekyll.exe serve
Configuration file: c:/Users/.../slate/_config.yml
            Source: c:/Users/.../slate
       Destination: c:/Users/.../slate/_site
      Generating... You are missing a library required for Markdown.
      Please run: $ [sudo] gem install redcarpet
...
{% endprism %}

Helpful error though, and I should have expected this, as the OCRA docs tell me:

> Your program should ‘require’ all necessary files when invoked without arguments, so OCRA can detect all dependencies.

 So I simply open up `bin/jekyll` and add `require 'redcarpet'` at the top. Re-running OCRA & trying the new `jekyll.exe`, I actually find `jekyll build` works.

{% prism bash %}
$ jekyll.exe build
Configuration file: c:/Users/.../slate/_config.yml
            Source: c:/Users/.../slate
       Destination: c:/Users/.../slate/_site
      Generating... done.
{% endprism %}

This means OCRA can get us to a jekyll with a working `build` command by adding a single require statement. For some people this may be all you need for working on your static sites. Pretty cool I think.

I'll soldier on though and see what else we can get working. 

### The `serve` command

The `serve` command chokes, complaining about `webrick`.

{% prism bash %}
$ jekyll.exe serve
...
C:/Users/.../1.9.1/rubygems/custom_require.rb:36:in `require':
cannot load such file -- webrick (LoadError)
{% endprism %}

I try the same tactic as before, add `require 'webrick'` to the top of `bin/jekyll`, rebuild and try again. We get a different error:

{% prism bash %}

$ jekyll.exe serve
...
error: No such file or directory - C:/Users/.../AppData/Local/Temp/ocr7DD1.tmp/src/jekyll-1.2.0/lib/jekyll/mime.types. Use --trace to view backtrace
{% endprism %}

Turns out this is because the path is being resolved to look within OCRA's *temp directory* (this is explained in the OCRA docs). We need to pull the `mime.types` file into the executable so that it is available in this temp directory at runtime. To do this we add the path to the end of our `ocra` build command.

{% prism bash %}
ocra jekyll-1.2.0/bin/jekyll jekyll-1.2.0/lib/jekyll/mime.types
{% endprism %}

Success - `serve` now works and spins up a webrick server.

{% prism bash %}
$ jekyll.exe serve
Configuration file: c:/Users/.../slate/_config.yml
            Source: c:/Users/.../slate
       Destination: c:/Users/.../slate/_site
      Generating... done.
[2013-10-31 11:44:30] INFO  WEBrick 1.3.1
[2013-10-31 11:44:30] INFO  ruby 1.9.3 (2013-06-27) [i386-mingw32]
[2013-10-31 11:44:30] INFO  WEBrick::HTTPServer#start: pid=3484 port=4000
{% endprism %}

### The `--watch` option

The `--watch` option fails with the following error:

{% prism bash %}
$ jekyll.exe build --watch
C:/Users/.../ruby/site_ruby/1.9.1/rubygems/custom_require.rb:36:in `require': cannot load such file --
directory_watcher (LoadError)
{% endprism %}

Seems familiar. Adding `require 'directory_watcher'` to `bin/jekyll` fixes it, both `build` and `serve` now work with `--watch`.

### The `new` Command

Sufficed to say we're going to need to tell OCRA to pull in all files in the `lib/site_template` directory, so that when we run `jekyll new` it can spit them straight back out:

{% prism bash %}
$ ocra jekyll-1.2.0/bin/jekyll jekyll-1.2.0/lib/jekyll/mime.types jekyll-1.2.0/lib/site_template/**/* jekyll-1.2.0/lib/site_template/*
{% endprism %}

With that, `new` now works and we can replicate the jekyll quick-start flow found on the [jekyll homepage](http://jekyllrb.com).

{% prism bash %}
$ jekyll.exe new my-awesome-site
$ cd my-awesome-site
$ ../jekyll.exe serve
# => Now browse to http://localhost:4000
{% endprism %}

<a name="pygments"></a>
### No Pygments?

When you use `jekyll new` you may notice a lack of code syntax highlighting as pygments is disabled by default in `_config.yml`.

But before you run away screaming, don't worry - there are decent (and easy to setup) alternatives. Namely that you set up your blog to use client-side syntax highlighting instead. There are a couple of ways to do this:

 * Use the [prism plugin](https://github.com/gmurphey/jekyll-prism-plugin). Yes, plugins seem to work (simple ones at least)!
 * Use [google-code-prettify](https://code.google.com/p/google-code-prettify/). I have a gist which shows you how [here](https://gist.github.com/nilliams/7138983#file-gistfile1-md).

Pygments is a bit of an awkward dependency, and when using Jekyll on Windows prior to this, I always used client-side highlighting instead. I've been happy enough that I haven't seriously looked into how feasible it would be to get Pygments working. If anyone is really keen to do so I'd love to hear from them.

Another alternative could be to do what I believe [docco](http://jashkenas.github.io/docco/) *used to do* - use a [web-service version](http://pygments.appspot.com/) of Pygments if it is not available locally.

Personally, I rarely, if ever use the `new` command and would instead pull a site template from github which already has client-side syntax highlighting setup. For example, a quick start, using the theme for this blog:

{% prism bash %}
$ git clone https://github.com/nilliams/jekyll-minimal-blog-theme.git
$ cd jekyll-minimal-blog-theme
$ jekyll.exe serve
{% endprism %}

<a name="recap"></a>
### Recap

So all we did was ... 1. unpack jekyll

{% prism bash %}
$ gem unpack jekyll
{% endprism %}

then 2. add a bunch of require statements to the top of `bin/jekyll`

{% prism ruby %}
require 'redcarpet'
require 'jekyll'
require 'webrick'
require 'directory_watcher'
{% endprism %}

... and 3. build using OCRA like so, to pull in `mime.types` and `site_template`

{% prism bash %}
$ ocra jekyll-1.2.0/bin/jekyll jekyll-1.2.0/lib/jekyll/mime.types jekyll-1.2.0/lib/site_template/**/* jekyll-1.2.0/lib/site_template/*
{% endprism %}

I've been really impressed with [OCRA](http://ocra.rubyforge.org/) and I'm still pretty surprised this even works. Be sure to [check it out](http://ocra.rubyforge.org) and consider it for your own projects. Maybe we can make some more Ruby tools less painful on Windows.

If you have any questions or comments, hit me up on [twitter](http://twitter.com/nickwuh).