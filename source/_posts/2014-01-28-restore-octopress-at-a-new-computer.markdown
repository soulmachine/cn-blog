---
layout: post
title: "Restore Octopress at a new computer"
date: 2014-01-28 00:04
comments: true
categories: tools
---
OS: Ubuntu 12.04 64-bit

##1. Install ruby

###1.1 Install ruby via RVM

    $ \curl -sSL https://get.rvm.io | bash -s stable --ruby

###1.2 [Integrating RVM with gnome-terminal](https://rvm.io/integration/gnome-terminal)

###1.3 Give it a try
Exit current shell, and open a new shell,

    ruby -v

You have successfully installed ruby.


##2. Clone your blog to the new machine
First you need to clone the `source` branch to the local octopress folder.

    $ git clone -b source git@github.com:username/username.github.com.git octopress

<!-- more -->
Then clone the `master` branch to the `_deploy` subfolder.

    $ cd octopress
    $ git clone git@github.com:username/username.github.com.git _deploy 

Then run the rake installation to configure everything

    $ gem install bundler
    $ bundle install

NOW you've setup with a new local copy of your Octopress blog.

You don't need to run `rake setup_github_pages` any more.


##3. Blogging at more than one computer

###3.1 Pushing changes
If you want to blog at more than one computer, you need to make sure that you push everything before switching computers. From the first machine do the following whenever you’ve made changes:

    $ rake new_post["hello world"] 
    $ rake generate
    $ rake deploy

This will generate your blog, copy the generated files into `_deploy/`, add them to git, commit and push them up to the master branch, see [Deploying to Github Pages](http://octopress.org/docs/deploying/github/). 
Don't forget to commit the source for your blog.

    $ git add .
    $ git commit -am "Some comment here." 
    $ git push origin source  # update the remote source branch 

###3.2 Pull changes at another computer

    $ cd octopress
    $ git pull origin source  # update the local source branch
    $ cd ./_deploy
    $ git pull origin master  # update the local master branch

## Reference
[Clone Your Octopress to Blog From Two Places](http://blog.zerosharp.com/clone-your-octopress-to-blog-from-two-places/)

