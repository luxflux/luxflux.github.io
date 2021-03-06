---
layout: post
title: "Javascript tests with PhantomJS"
date: 2013-01-19 17:57
comments: true
categories: ['Ruby', 'Rails', 'Test', 'RSpec', 'PhantomJS', 'Capybara']
redirect_from: '/blog/2013/01/19/javascript-tests-with-phantomjs/'
disqus_identifier: 'http://blog.yux.ch/blog/2013/01/19/javascript-tests-with-phantomjs/'
---

Today I wanted to write some javascript tests for my [Moviezz](https://github.com/luxflux/Moviez) project.

Maybe two years ago, I've written javascript tests for a project at [nine.ch](http://nine.ch).
I ended up by using Cucumber, Capybara and Selenium for this. The result was okay, but it took you a very long time to run the tests
and you could not do anything else than watching your webbrowser clicking links and filling in text.

Some weeks ago, I heard about PhantomJS and I waited since for the right chance to try it out.
PhantomJS enables you to run your javascript tests headless, means no browser windows is opened and used to click and fill.
This also gives you the ability to run your javascript tests via Jenkins or Travis.

<!-- more -->

At first, I verified that ```capybara``` is in my ```Gemfile```.

{% highlight ruby %}
gem 'capybara', group: :test
{% endhighlight %}

It also has to be loaded by ```RSpec```, so I required it in ```spec_helper```.
{% highlight ruby %}
require 'capybara/rspec'
require 'capybara/rails'
{% endhighlight %}

After this, I went on and wrote the first javascript test with RSpec.
Note the ```js: true``` on line 7, this switches the Capybara driver to the javascript adapter.

{% highlight ruby %}
require 'spec_helper'

describe 'search for a movie' do

  let(:tmdb_movie) { tmdb_result }

  it 'searches for the given title', js: true do
    visit new_movie_path
    page.should_not have_content('Star Wars Episode VII')
    fill_in 'title', with: 'Star Wars'
    page.should have_content('Star Wars Episode VII')
  end
end
{% endhighlight %}

When I ran the test, Capybara opened a new Firefox window and did my defined steps as expected.

To use PhantomJS with Capybara, you have to switch to another adapter.
There is an adapter called Poltergeist which enables you to use PhantomJS.
So I went on and added Poltergeist to my Gemfile.

{% highlight ruby %}
gem 'poltergeist', group: :test
{% endhighlight %}

As soon as I required ```capybara/poltergeist``` and added ```Capybara.javascript_driver = :poltergeist``` in ```spec_helper.rb```, my javascript test passed without opening a window.

You cannot use transactions, which is the default, to cleanup your database after each javascript test, better explained [here](https://github.com/jnicklas/capybara#transactions-and-database-setup).
To solve this, you can use ```DatabaseCleaner``` with the truncation strategy instead of transactions.

But there is a caveat, the truncation strategy is slower.
As I like my tests run fast, I wrote the following RSpec configuration which just enables the truncation strategy
for tests with ```js: true``` set.

{% highlight ruby %}
RSpec.configure do |config|

  config.before :suite do
    DatabaseCleaner.strategy = :transaction
    DatabaseCleaner.clean_with :truncation
    Capybara.javascript_driver = :poltergeist
  end

  config.before :each do
    if example.metadata[:js]
      DatabaseCleaner.strategy = :truncation
    else
      DatabaseCleaner.strategy = :transaction
      DatabaseCleaner.start
    end
  end

  config.after :each do
    DatabaseCleaner.clean
  end

end
{% endhighlight %}

Note, that I also moved ```Capybara.javascript_driver = :poltergeist``` into this config file as it belongs to the javascript configuration.

After this, I just had to disable the default transactional feature in ```spec_helper.rb```
{% highlight ruby %}
RSpec.configure do |config|
  config.use_transactional_fixtures = false
end
{% endhighlight %}

Now, I have headless javascript tests and the fastest possible database cleanup strategy depending on the test type.

All the code above can also be found [in a gist](https://gist.github.com/4573914).
