---
layout: post
title: "Simple RSpec tests for Puppet Modules"
description: ""
category: "howto, puppet"
tags: [tfhartmann, hakamadare, puppet, rspec-puppet]
---
{% include JB/setup %}

### "Test yo code!"

  While this post isn't directly "cloud" related, to make our nodes and application patterns work, we rely heavily on configuration management tools like Puppet, Ansible, Chef, etc.  Since we rely so heavily on  automated configuration management, it seems like not only a good idea, but a requirement get as much testing done early and to automate it. Now, truth be told, Ruby and rspec are brave new lands for me personally so I've perhaps put out code **may not** have always been the best tested... I know it's unthinkable, but hey, good is better then perfect!  Or as per the Iron Triangle -  "Good, Cheap, Fast, pick two" 

  I spent part of this weekend hacking around on rspec-puppet, configuring some basic tests, and hooking the github repo into [travis-ci.](http://travis-ci.org) While there are lots of great blog posts out there on **how** to do it, I felt pretty discombobulated by all the copies of Rakefile, gemfile and spec_helper.rb there were running around, and found myself working in circles. Hopeful this post will help someone whose just starting our with Ruby and rspec/rspec-puppet testing and make things a little clearer. 

  I Decided to start with a module I wrote to deploy and manage [Splunk](https://github.com/huit/puppet-splunk) which while still in progress I thought would be a decent, simple place to start. 

### "Why Test?" 

  You might find yourself saying "I can run `puppet apply --verbose` and `puppet validate parser`  as well as the next guy..  why bother with all the fancy tests??!"  to which I reply - "Badges... we need Badges" ![Build Passing!]({{ site.url }}/assets/images/travis-ci-passing-badge.png) How awesome is it to know only know that a Module or project your may want to use.. works??!  Pretty darn awesome.  

  Although badges are awesome, you might not need no stinking badges... **sigh** as your modules get more and more complex, it becomes really useful to have some testing.  For a while now, I've been using 'puppet apply --verbose', 'puppet agent --test', 'puppet validate parser' tests in the <MODULE>/tests directory, which worked ok for a while. Then I discovered Vagrant and started spinning up vagrant boxes of different flavors to run tests against, which was/IS awesome but it's also a pain to keep spinning boxes. It also takes **TIME** something that I never have enough of.

  Automating your tests gives you the ability to pass parameters, Facter Facts,  and much more in a nice, fast, reusable format. I found a presentation by [Jan Vansteenkiste](http://vstone.eu/talks/puppet-module-testing/) about this where he hits the nail on the head!

### "The How" 

  I spend a good amount of time figuring out the "how" of getting rspec-puppet working. in My case, I had so many different versions of the base files, it was starting to make my head spin.  Let me save you a bunch of time. Use the [Puppet Labs Sec Helper Gem](https://github.com/puppetlabs/puppetlabs_spec_helper) This will save you a lot of time and Ruby frustration. You may find the [Puppet Labs Post about the spec_helper gem useful!](http://puppetlabs.com/blog/the-next-generation-of-puppet-module-testing) One of the really useful features of the puppetlabs_spec_helper gem is it's ability to pull in your dependency modules!

First install the Gems you'll need.  I found the easiest way to do this was with [bundler.](http://bundler.io/) Set up a Gemfile at the root of your module: 

Example Gemfile:

```ruby
source 'https://rubygems.org'

if ENV.key?('PUPPET_VERSION')
  puppetversion = "= #{ENV['PUPPET_VERSION']}"
else
  puppetversion = ['>= 2.7']
end

gem 'rake'
gem 'puppet-lint'
gem 'rspec-puppet'
gem 'puppetlabs_spec_helper'
gem 'puppet', puppetversion
```

Then using bundler just do a `bundle install` to get all the gems you'll need for testing!


Next setup your spec folders for testing, an easy way to do this is by running `rspec-puppet-init`. One word of caution though, to get puppetlabs_spec_helper working the way you want, you'll need to clean our the 'spec/fixtures/modules/<name>' directory of symlinks and delete the <modulename> directory.  The puppetlabs_spec_helper gem will install and remove the directory and links for you, and fail if the links and directory already exist. 

```bash
[alaric@dhcp-128-103-62-80 huit-test]$ rspec-puppet-init
 + spec/classes/
 + spec/defines/
 + spec/functions/
 + spec/hosts/
 + spec/fixtures/
 + spec/fixtures/manifests/
 + spec/fixtures/modules/
 + spec/fixtures/modules/test/
 + spec/fixtures/manifests/site.pp
 + spec/fixtures/modules/test/manifests
!! spec/spec_helper.rb already exists and differs from template
 + Rakefile
 ```

You can see from my test, it failed on spec_helper.rb since it already existed, don't worry about it, we are going to update that bad boy in just a moment :) 

Now let's set up our core config files. 

First, our Rakefile in the root of the module. puppetlabs_spec_helper lets us create a really simple Rakefile! For those of us with little to no Ruby skills.. this is quite nice. 

```ruby
require 'rubygems'
require 'puppetlabs_spec_helper/rake_tasks'

task :default => [:spec, :lint]
```

Next we'll set up .travis.yml in the root of the module. 

```yaml
rvm: 1.8.7
bundler_args: ''
notifications:
  email:
    - me@example.com
env:
  - PUPPET_VERSION=3.3.1
```

You'll also want a  .fixtures.yml file so that puppetlabs_spec_helper can pull in any dependancies  you might need. 

```yaml
fixtures:
  repositories:
    stdlib: git://github.com/puppetlabs/puppetlabs-stdlib.git
    initfile: git://github.com/puppetlabs/puppetlabs-inifile.git
  symlinks:
    splunk: "#{source_dir}"
```

So in the root directory of your module, you should have a Gemfile, a Rakefile, a .fixtures.yml, and a .travis.yml file. 

Now we can set up our spec_helper.rb file in the spec directory of your module. This is where puppetlabs_spec_helper really jumped out as awesome.  It let me just have a single-line spec_helper.rb! 

```ruby
require 'puppetlabs_spec_helper/module_spec_helper'
```

Nice, simple, and clean. 

Lastly, you can create a simple test! In my case I created a really simple test just to ensure that my module loaded, and ran puppet-lint against my code.  Put your tests in the directory that matches the type of object your testing, in my case I was testing the Splunk class, so created a test called  'puppet-splunk/spec/classes/splunk_spec.rb'. 

```ruby
require 'spec_helper'

describe 'splunk', :type => :class do

  describe "Splunk class with no parameters, basic test" do
    let(:params) { { } }

      it {
        should contain_package('splunkforwarder')
        should contain_service('splunk')
      }
  end
end
```

### Done and Done

So thats pretty much it! After much hand wringing, I was able to get some simple tests working. 
In addition to being able to integrate this with something like travis-ci, this also allows you to
run things like puppet-lint tests quickly over all of your classes in a module without having to set up 
a git pre-commit hook. 

Check out 'rake help' and 'rake lint' from the root of your module! Pretty awesome stuff!

```bash
[alaric@dhcp-128-103-62-80 puppet-splunk]$ rake help
rake build            # Build puppet module package
rake clean            # Clean a built module package
rake coverage         # Generate code coverage information
rake help             # Display the list of available rake tasks
rake lint             # Check puppet manifests with puppet-lint
rake spec             # Run spec tests in a clean fixtures directory
rake spec_clean       # Clean up the fixtures directory
rake spec_prep        # Create the fixtures directory
rake spec_standalone  # Run spec tests on an existing fixtures directory
```


#### Useful links and Citations

* http://puppetlabs.com/blog/the-next-generation-of-puppet-module-testing
* https://github.com/puppetlabs/puppetlabs_spec_helper
* http://puppetlabs.com/blog/testing-modules-in-the-puppet-forge
* http://bombasticmonkey.com/2012/03/02/automatically-test-your-puppet-modules-with-travis-ci/
* https://github.com/vStone/puppet-testing-example

[@tfhartmann](https://github.com/tfhartmann)
