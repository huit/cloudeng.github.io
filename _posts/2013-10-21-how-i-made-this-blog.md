---
layout: post
title: "How I made this Blog"
description: "How I made this Blog - A Drama in Many Parts"
category: ""
tags: [tfhartmann]
---
{% include JB/setup %}

## _How I made this Blog - A Drama in Many Parts._

 
- [The Quick and Dirty](#The Quick and Dirty)
- [Install s3_website](#Install s3_website)

This post may get a little long and ramble a bit, so let me skip directly to the payoff - quick and easy content managment on crazy robust and reliable infrastructure. Here's the commands I need to add and post new content:

```bash
$ vim _posts/2013-09-27-hello-world.md 
$ jekyll serve 
$ s3_website push
```

and _Behold - The power of S3 and CloudFront!__  

===

## The Quick and Dirty

Steps:

1. Make sure you have a new-ish version of ruby, if not, install [rbenv](https://github.com/sstephenson/rbenv#homebrew-on-mac-os-x)
2. Install or configure Jekyll ( I used Jekyll bootstrap to start with since I'm new to it )
3. Install [s3_website](https://github.com/laurilehmijoki/s3_website)
4. Configure `_config.yml` and `s3_website.yml`
5. Create Your first post under _posts
6. Check to see if your site works locally by running `jekyll serve`
7. Create your AWS **Stuff** `s3_website cfg apply`
8. Point a CNAME at your CloudFront bucket
9. Publish your Site to your s3 bucket: `s3_website push`
10. Repeat steps 5,6 and 9 for fun and profit!

===

### Install s3_website

On my shiny new Mac, I wasn't able to install s3_website due to failed dependency

- (https://github.com/laurilehmijoki/s3_website)

As follows:

```bash
alaric@dhcp-128-103-209-223 splunk-searches]$ gem install -b s3_website
ERROR:  While executing gem ... (Gem::FilePermissionError)
    You don't have write permissions into the /Library/Ruby/Gems/1.8 directory.
[alaric@dhcp-128-103-209-223 splunk-searches]$ sudo gem install -b s3_website
Password:
ERROR:  Error installing s3_website:
  nokogiri requires Ruby version >= 1.9.2.
```

However thanks to brew I was able to use a newer version of Ruby and get it installed!

```bash
$ brew install ruby
$ sudo gem install -b s3_website
Password:
```

After proving that I know nothing about Ruby paths, @hakamadare turned me on to [rbenv](https://github.com/sstephenson/rbenv#homebrew-on-mac-os-x)

```bash

$ brew install rbenv
==> Downloading https://github.com/sstephenson/rbenv/archive/v0.4.0.tar.gz
######################################################################## 100.0%
==> Caveats
To use Homebrew's directories rather than ~/.rbenv add to your profile:
  export RBENV_ROOT=/usr/local/var/rbenv

To enable shims and autocompletion add to your profile:
  if which rbenv > /dev/null; then eval "$(rbenv init -)"; fi
==> Summary
🍺  /usr/local/Cellar/rbenv/0.4.0: 31 files, 152K, built in 2 seconds

$ brew install ruby-build
==> Installing dependencies for ruby-build: autoconf
==> Installing ruby-build dependency: autoconf
==> Downloading http://ftpmirror.gnu.org/autoconf/autoconf-2.69.tar.gz
######################################################################## 100.0%
==> ./configure --prefix=/usr/local/Cellar/autoconf/2.69
==> make install
🍺  /usr/local/Cellar/autoconf/2.69: 69 files, 2.0M, built in 22 seconds
==> Installing ruby-build
==> Downloading https://github.com/sstephenson/ruby-build/archive/v20130923.tar.gz
######################################################################## 100.0%
==> ./install.sh
🍺  /usr/local/Cellar/ruby-build/20130923: 84 files, 372K, built in 2 seconds

```


And in my case, I had to source my `.bash_profile` again after switching ruby versions with `rbenv`, (So for those keeping score, three (ahhahah) versions of ruby now) to get the gem to show up in my path (two versions of s3_website..). SEtup the environment via via `source .bash_profile`
 
 
## AWS Creds

Since we are all agile and stuff, multiple people will be updating the blog - so I didn't want to put AWS credentials into the repo itself.. plus we *are* publishing them on github, so I thought i'd source them from ENV variables as per the instructions on the s3_website docs... but I source in my AWS Creds this way in my bash_profile file

```bash
export AWS_CONFIG_FILE=~/aws_creds.txt 
```

AND! --- FAIL! 

```bash
$ s3_website cfg apply
Applying the configurations in s3_website.yml on the AWS services ...
/Users/alaric/.rbenv/versions/2.0.0-p247/lib/ruby/gems/2.0.0/gems/configure-s3-website-1.4.0/lib/configure-s3-website/http_helper.rb:62:in `digest': no implicit conversion of nil into String (TypeError)
from /Users/alaric/.rbenv/versions/2.0.0-p247/lib/ruby/gems/2.0.0/gems/configure-s3-website-1.4.0/lib/configure-s3-website/http_helper.rb:62:in `create_s3_digest'
from /Users/alaric/.rbenv/versions/2.0.0-p247/lib/ruby/gems/2.0.0/gems/configure-s3-website-1.4.0/lib/configure-s3-website/http_helper.rb:6:in `call_s3_api'
from /Users/alaric/.rbenv/versions/2.0.0-p247/lib/ruby/gems/2.0.0/gems/configure-s3-website-1.4.0/lib/configure-s3-website/s3_client.rb:34:in `enable_website_configuration'
from /Users/alaric/.rbenv/versions/2.0.0-p247/lib/ruby/gems/2.0.0/gems/configure-s3-website-1.4.0/lib/configure-s3-website/s3_client.rb:12:in `configure_website'
from /Users/alaric/.rbenv/versions/2.0.0-p247/lib/ruby/gems/2.0.0/gems/configure-s3-website-1.4.0/lib/configure-s3-website/runner.rb:4:in `run'
from /Users/alaric/.rbenv/versions/2.0.0-p247/lib/ruby/gems/2.0.0/gems/configure-s3-website-1.4.0/bin/configure-s3-website:12:in `<top (required)>'
from /Users/alaric/.rbenv/versions/2.0.0-p247/bin/configure-s3-website:23:in `load'
from /Users/alaric/.rbenv/versions/2.0.0-p247/bin/configure-s3-website:23:in `<main>'
$
```

But thats ok, it's a quick hack! 

EPIC FAIL AGAIN!!! 

Why? Because sometime I assume I'm smarting then the instructions.... and am mostly wrong. the envirment variables they want you to use are

```bash
export S3_ID=<ID>
export S3_SECRET=<SECRET>
```

NOT `aws_access_key_id`, and `aws_secret_access_key` which I, having read the AWS docs (ok, skimmed) tried first! 

_AND THEN!_

```bash
$ s3_website cfg apply
Applying the configurations in s3_website.yml on the AWS services ...
Created bucket cloudhacks.huit.harvard.edu in the US Standard Region
Bucket cloudhacks.huit.harvard.edu now functions as a website
Bucket cloudhacks.huit.harvard.edu is now readable to the whole world
No redirects to configure for cloudhacks.huit.harvard.edu bucket
Do you want to deliver your website via CloudFront, the CDN of Amazon? [y/N] n
```

and BAM!! a freaking awesome bucket! 

```bash
$ s3_website push
Deploying _site/* to cloudhacks.huit.harvard.edu
Calculating diff ... done
Uploading 19 new file(s)
Upload archive.html: Success!
Upload 404.html: Success!
Upload 2013/09/27/hello-world/index.html: Success!
Upload assets/themes/tom/css/screen.css: Success!
Upload assets/themes/tom/images/rss.png: Success!
Upload assets/themes/tom/css/syntax.css: Success!
Upload assets/themes/twitter/bootstrap/css/bootstrap.2.2.2.min.css: Success!
Upload assets/themes/twitter/bootstrap/img/glyphicons-halflings-white.png: Success!
Upload assets/themes/twitter/bootstrap/img/glyphicons-halflings.png: Success!
Upload categories.html: Success!
Upload atom.xml: Success!
Upload assets/themes/twitter/css/style.css: Success!
Upload Genfile: Success!
Upload index.html: Success!
Upload History.markdown: Success!
Upload pages.html: Success!
Upload rss.xml: Success!
Upload sitemap.txt: Success!
Upload tags.html: Success!
Done! Go visit: http://cloudhacks.huit.harvard.edu.s3-website-us-east-1.
                                             amazonaws.com/index.html
```

OK - that was pretty sweet... but I can totall do that on my own... it's just S3 right??  Lets CDN this bad larry! - So I'll run `s3_website cfg apply` again and see what happens!! 

```bash
$ s3_website cfg apply
Applying the configurations in s3_website.yml on the AWS services ...
Bucket cloudhacks.huit.harvard.edu now functions as a website
Bucket cloudhacks.huit.harvard.edu is now readable to the whole world
No redirects to configure for cloudhacks.huit.harvard.edu bucket
Do you want to deliver your website via CloudFront, the CDN of Amazon? [y/N]
y
  The distribution E2KIUNT4276WHO at d99xfp4zz9wie.cloudfront.net now delivers 
  the bucket cloudhacks.huit.harvard.edu
    Please allow up to 15 minutes for the distribution to initialise
    For more information on the distribution, see 
        https://console.aws.amazon.com/cloudfront
  Added setting 'cloudfront_distribution_id: E2KIUNT4276WHO' into s3_website.yml

$ jobs
[1]+  Stopped                 vim _config.yml
[2]-  Stopped                 vim s3_website.yml

$ fg 2
vim s3_website.yml
```

Now I have d99xfp4zz9wie.cloudfront.net as a target for our awesome blog! I was totally ready for my sweet sweet vicotry! 

So All I needed to do now was to create the CNAME in DNS with our DNS Admin tool! Click-Click Click... and 

BAAA-- wait... --- no BAM??!

I get this??!!


---

## ERROR

The request could not be satisfied.
Generated by cloudfront (CloudFront)

---


OK- Back to reading the [AWS CloudFront Docs](http://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/CNAMEs.html) and apprently I need to add an alternate domain name to the CloudFront Config, Luckly s3_website lets me do that with just a bit of editing to the config file! 

```yaml
cloudfront_distribution_config:
   aliases:
     quantity: 1
     items:
       CNAME: cloudhacks.huit.harvard.edu
```

After updating the config, I gave s3_website another spin - 

```bash
$ s3_website cfg apply
Applying the configurations in s3_website.yml on the AWS services ...
Bucket cloudhacks.huit.harvard.edu now functions as a website
Bucket cloudhacks.huit.harvard.edu is now readable to the whole world
No redirects to configure for cloudhacks.huit.harvard.edu bucket
Detected an existing CloudFront distribution (id E2KIUNT4276WHO) ...
  Applied custom distribution settings:
    aliases:
      quantity: 1
      items:
        CNAME: cloudhacks.huit.harvard.edu
```

In the AWS Console, I can see my new Domain added, and the distribution is listed as "In Progress" 

And with that - after waiting about 10 minutes for the content to update, I got my much anticipated "BAM!" and vicotry dance! Welcome to the [Cloud Hacks blog!](http://cloudhacks.huit.harvard.edu/) 

To Push Posts we just use the `s3_website push` command!

```bash
Deploying _site/* to cloudhacks.huit.harvard.edu
Calculating diff ... done
No new or changed files to upload
Done! Go visit: 
    http://cloudhacks.huit.harvard.edu.s3-website-us-east-1.amazonaws.com/index.html
Invalidating Cloudfront items...
  /
succeeded
```

After All that, I thought I was golden, however I couldn't for the life of me get links working! 

```bash
# This is the default format. 
# For more see: http://jekyllrb.com/docs/permalinks/
#permalink: /:categories/:year/:month/:day/:title 
```

The End

[@tfhartmann](https://github.com/tfhartmann)
