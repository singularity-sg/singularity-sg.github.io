---
layout: post
title: "Rails Development using Vim"
description: "Speed up your Rails development with the best editor out there"
category: "Software Development Tools" 
tags: ['ruby on rails', 'vim']
---
{% include JB/setup %}

Finding a suitable editor for Ruby on Rails development can be quite long journey. I started out trying Aptana's Studio but later found it simply too resource intensive and heavy. The main benefit it brought was the ability to perform debugging on a Windowing environment (with the inclusion of the ruby-debug-ide gem. In the end, I decided to go with Vim. Many developers may recoil in horror at the thought of using something as arcane as Vim to do development work but I soon found out that my earlier misgivings were all uncalled for as Vim proved itself to be as powerful and much more nimbler than Aptana Studio.

<!--more-->

What made the development such a joy was the inclusion of the Vim plugins that are customized for Rails development. These plugins are 

1. [vim-ruby](https://github.com/vim-ruby/vim-ruby)
2. [vim-rails](http://www.vim.org/scripts/script.php?script_id=1567)
3. [vim-rspec](https://github.com/thoughtbot/vim-rspec)

If you are using pathogen to manage your Vim plugins, you can easily install the 3 plugins into your Vim environment. It's amazing how productive you can suddenly become when you start development using a lightweight tool. I find myself relying on the mouse less and coding more. If you are using vim-ruby and you'd like to enjoy the autocompletion feature, make sure you get a copy of vim that is compiled with the ruby interface. You could also use ctags to generate an index of method calls in your project. The only gripe I have with Vim is the lack of ability to invoke shell commands without leaving the explore mode buffer screen. 

I'm still at an early stage of Vim exploration for Ruby development but I'm very satisfied with my experience so far.
