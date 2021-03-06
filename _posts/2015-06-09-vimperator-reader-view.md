---
layout:     post
title:      "Vimperator + Reader View"
date:       2015-06-09 15:34
comments:   true
---

Firefox 38.0.5 introduced [Reader View](https://support.mozilla.org/en-US/kb/enjoy-clutter-free-web-pages-reader-view) 
which strips out everything except the main content of a page for easier 
reading.  I use [vimperator](http://www.vimperator.org/vimperator/) and wanted 
to add a key mapping to quickly go into reader view.  To save others the trouble
of figuring it out, here's how: 

Basically all the little reader button does is prepend `"about:reader?url="` to 
the current URL so all we need to do is write a command that does just that. 
Turns out it's easier said than done.  I hunted through the vimperator 
documentation, experimenting with copying the URL with `y` (but there's no way 
to paste), using `<url>` in the command specification (doesn't work), and 
creating macros (inelegant and the `@` key is too far away) to no avail.  

Finally I found the `:execute` command and `content.location.href` which isn't 
documented at *all* but expands to the current URL.  It can then be concatenated
into a command with `+` in the argument of `:execute`.  

Without further ado, execute the following in vimperator: 
{% highlight vim %}
:command reader :execute "open about:reader?url=" + content.location.href
:map gr :reader<CR>
:mkvimperatorrc
{% endhighlight %}
Or equivalently, add the following to your `.vimperatorrc`: 
{% highlight vim %}
command reader :execute "open about:reader?url=" + content.location.href
map gr :reader<Return>
{% endhighlight %}

This can cause strange behavior if you call it on a page that's already in 
reader mode.  It will get 'stuck' and `H` to go back doesn't work but `2H` will.

There is a way to prevent this using javascript's conditional operator
`condition ? expr1 : expr2`.  This evaluates to *`expr1`* if the condition is 
true and *`expr2`* if the condition is false.  The *`condition`* will be 
whether we're already in reader view (whether the URL starts with 
`"about:reader"`), if we are, go back to normal mode (remove the 
`"about:reader"?url="`, and if not, go to reader mode same as before.  
Thus: 
{% highlight vim %}
:command! reader :execute ((content.location.href.indexOf("about:reader?url=") !== -1) ? ("open " + content.href.substring(17)) : ("open about:reader?url=" + content.location.href))
:mkvimperatorrc!
{% endhighlight %}
or in `.vimperatorrc`: 
{% highlight vim %}
command reader :execute ((content.location.href.indexOf("about:reader?url=") !== -1) ? ("open " + content.location.href.substring(17)) : ("open about:reader?url=" + content.location.href))
{% endhighlight %}
