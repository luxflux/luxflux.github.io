---
layout: post
title: "Gem: Version Reader"
date: 2012-11-25 15:16
comments: true
categories: [Ruby, Gem, version_reader]
redirect_from: '/blog/2012/11/25/gem-version-reader/'
---

If you are using a ```VERSION```-file to give your new (or old) cool
software tool a version, you maybe also want to display it in the footer
of the website or in the ```help``` command option of your CLI.

Instead of writing stuff like this
{% highlight ruby %}
def version
  @version ||= File.open('VERSION', 'rb') { |f| f.read }.strip
end
{% endhighlight %}

you could use [version_reader](https://github.com/luxflux/version_reader)
(written by me) which reads the ```VERSION```-file and does all the
stripping and formatting for you.

<!-- more -->

Version Reader is just a small wrapper around the ```VERSION```-file.

Let's do a short example. Imagine that your ```VERSION```-file is in ```~/MyApp/```
with the content ```0.4.2\n```. Load the version with the following code:

{% highlight ruby %}
require 'version_reader'

version_reader = VersionReader.new('~/MyApp/VERSION')
{% endhighlight %}

Now, you can display a nicely formatted version
{% highlight ruby %}
version_reader.normal # Output: 0.4.2
{% endhighlight %}

If you don't like this output, just write a flavor to add a
different one. There is already a Rails-flavor which adds some
additional output formats:

{% highlight ruby %}
version_reader.extend VersionReader::Flavor::Rails
version_reader.rails_env # Output: 0.4.2-development
{% endhighlight %}

Check the [Readme](https://github.com/luxflux/version_reader) for more
details.

By the way, if you add this Gem in the ```Gemfile``` of your Rails-Application,
it will automatically define ```MyApp::Application.version``` with a
Rails-flavored instance of ```VersionReader```.


