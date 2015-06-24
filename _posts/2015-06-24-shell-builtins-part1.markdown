---
title:  "Exploring extglob"
date:   2015-06-24 17:18:00
description: Concise pattern matching in unix for the lazy
---

Take the following hypothetical directory structure:

{% highlight ruby %}
example/
  - a/
  - b/
  - c/
  ...
  - z/
{% endhighlight %}

In this scenario, there several files and subdirectories in each of the sibling directories in `example/`. Now let's say that you want to perform an operation on every one of these directories except `a/`, and because you are lazy, you want to do it in one command (sans pipes, `awk`, `grep`, or other compound, obscure one-liners).

## Some background
Globs already provide a solid set of inclusive, pattern matching operators. Here is a quick review of special characters that are already available by default in bash.

|* | wildcard matching (zero or more)|
|? | match exactly one character|
|[ ] | match any character in the brackets|

With these tools at your disposal, one could concievably `mv` every directory except for `a/` one directory up by simply enumerating all of the single letter directory names inside of brackets.

{% highlight ruby %}
$ mv [bcdefghijklmnopqrstuvwxyz]/ ../
{% endhighlight %}

While this may work in a few limited cases, what if the directory names are more than a single character long? What if the objects you want to move are better defined by exclusion than inclusion? What about more complex pattern-matching requirements?

## Hack
After reading this whole post, you'll never need to write something like this again.

{% highlight ruby %}
$ ls | grep -v a | while read dname; do mv "$dname" ../; done
{% endhighlight %}

There must be a better way, and in fact, there is.

## Enter extglob
Fortunately, bash has an extended set of glob operators that bring the full power of regular expression pattern matching to a terminal near you! First, investigate what options your shell has preconfigured.

{% highlight ruby %}
$ shopt
...
dotglob         off
execfail        off
expand_aliases  on
extdebug        off
extglob         off
extquote        on
failglob        off
force_fignore   on
...
{% endhighlight %}

Because of the rich set of capabilities extglob provides, it's typically turned off by default to prevent unexpected behavior. Let's turn it on.

{% highlight ruby %}
$ shopt -s extglob
{% endhighlight %}

Courtesy of the bash man page, below is a brief overview of your new toolbox.

?(pattern-list) | Matches zero or one occurrence of the given patterns
*(pattern-list) | Matches zero or more occurrences of the given patterns
+(pattern-list) | Matches one or more occurrences of the given patterns
@(pattern-list) | Matches one of the given patterns
!(pattern-list) | Matches anything except one of the given patterns

You may notice that the meaning of `?` has now changed, which could cause issues later, but for now, let's tackle the problem at hand: moving everything except for one thing.

{% highlight ruby %}
$ mv !(a) ../
{% endhighlight %}

Voilà! When it comes to bash pattern matching, your expressions can be both simple and powerful, while eliminating the need for syntactically horrible bash loops. If you want to play it safe, just disable extglob when not needed.

{% highlight ruby %}
$ shopt -u extglob
{% endhighlight %}

For more documentation on shell options and globs, `man bash`. For more examples using globs and extglobs, check out [Greg's Wiki](http://mywiki.wooledge.org/glob).
