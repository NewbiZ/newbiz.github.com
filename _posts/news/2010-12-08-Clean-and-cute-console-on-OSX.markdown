---
layout: page_post
title: Clean and cute console on OSX
categories: news
---
This article will help you customize the Console application to have cute colors and prompt.
It is a sequel to the very nice article provided by Todd Werth [on his blog](http://blog.infinitered.com/entries/show/6).

Let’s begin this article by directly showing you the expected result:

![Result console 1](/files/cute_console1.png "Result console 1")

As you noticed, we change the color and informations displayed at the prompt, as well as the main colors of the console.
These changes are reflected in other terminal applications such as vim.

![Result console 2](/files/cute_console2.png "Result console 2")

Okay, so the first thing to locate, is your `.bash_profile` file in your home directory. If the file does not exists, just create it.

This file contains command lines that are executed when launching a new Console window. So you can just start by typing them directly
in the terminal to test it before adding it to you `.bash_profile`

Bash configuration
-------------------
We are now going to activate some nice features of bash. For instance, we would like to have completion working case insensitive.
{% highlight bash %}
    bind "set completion-ignore-case on"     # Ignore case during command line completion
    bind "set bell-style none"               # Disable bell
    bind "set show-all-if-ambiguous on"      # Show all possibilities with a single tab
{% endhighlight %}

Custom promt
------------
Now it’s time to set up a clean prompt. The only informations we need is the current folder, and login.
{% highlight bash %}
    export PS1="\e[40;32m\u \e[40;34m\w \e[40;37m➤ \e[0m \[${COLOR_NC}\]"
{% endhighlight %}

There are a number of options for configuring a nice prompt. Here are some escaped sequences and their equivalents:

* `\a` : an ASCII bell character (07)
* `\d` : the date in “Weekday Month Date” format (e.g., “Tue May 26”)
* `\D{format}` : the format is passed to strftime(3) and the result is inserted into the prompt
* `\e` : an ASCII escape character (033)
* `\h` : the hostname up to the first ‘.’
* `\H` : the hostname
* `\j` : the number of jobs currently managed by the shell
* `\l` : the basename of the shell’s terminal device name
* `\n` : newline
* `\r` : carriage return
* `\s` : the name of the shell, the basename of $0 (the portion following the final slash)
* `\t` : the current time in 24-hour HH:MM:SS format
* `\T` : the current time in 12-hour HH:MM:SS format
* `\@` : the current time in 12-hour am/pm format
* `\A` : the current time in 24-hour HH:MM format
* `\u` : the username of the current user
* `\v` : the version of bash (e.g., 2.00)
* `\V` : the release of bash, version + patch level (e.g., 2.00.0)
* `\w` : the current working directory, with $HOME abbreviated with a tilde
* `\W` : the basename of the current working directory, with $HOME abbreviated with a tilde
* `\!` : the history number of this command
* `\#` : the command number of this command
* `\$` : if the effective UID is 0, a #, otherwise a $
* `\nnn` : the character corresponding to the octal number nnn
* `\\` : a backslash
* `\[` : begin a sequence of non-printing characters, which could be used to embed a terminal control sequence into the prompt
* `\]` : end a sequence of non-printing characters

Enabling colors
---------------
In the quest of having nice colors in the Console application on Leopard, you’ll need to install SIMBL.
Once done, just add the following line to enable colors:
{% highlight bash %}
    export CLICOLOR=1;   # Enable colors in terminal
{% endhighlight %}
