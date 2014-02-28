--- 
layout: post
title: Lets do some LaTeX
location: Manipal
---

Here is a [template](https://github.com/tsudot/scribbings) that I have made for
my mid-term report. Go ahead mod it if you want and send me a pull request.

A few pointers if you are new to LaTeX.  There are 2 files which you'll need
from the above link. One is ```mid-report.tex``` and the other is
```myrefs.bib```.  Install all the packages mentioned in the .tex file. If you
are on an Ubuntu machine, just install the package ```texlive-full```.  There
is a way in which you parse latex files which need citations from the .bib
file. Do the following in order.


{% highlight bash %}
latex mid-report.tex
bibtex mid-report.aux
latex mid-report.tex
latex mid-report.tex
{% endhighlight %}


There you go, enjoy making your report!
