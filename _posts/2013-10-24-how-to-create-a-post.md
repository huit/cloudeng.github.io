---
layout: post
title: "How to Create a Post"
description: ""
category: "howto"
tags: [tfhartmann, howto]
---
{% include JB/setup %}

## How to Post to the blog

It seemed like a good idea to write up some direction for our team on how to post to this blog! Where better to host this doc? THE BLOG itself! 

Things you'll need installed: 

**1.**  Jekyll 

This should be as easy as `gem install jekyll`

**2.** [s3_website](https://github.com/laurilehmijoki/s3_website)

  * This should just be `gem install s3_website` but it requries a new-ish version of ruby so you may have some tinkerin with rbenv like I did. 
  * You'll need to set your AWS ID and secret as environment variables: `S3_ID` and `S3_SECRET`

**3.** Once you've got both of those gem's installed, checkout the blog repo

**4.** To Create a new post, from the root of the project run: 

- `rake post title="How to Create a Post"`

**5.** Open with your favorite editor and create some text!


```bash
$ rake post title="How to Create a Post"
Creating new post: ./_posts/2013-10-24-how-to-create-a-post.md
$ vim _posts/2013-10-24-how-to-create-a-post.md 
```


**6.**  **TEST your post!** 

I got bit by this a bunch due to slight differnece with markdown syntax. Test by running `jekyll serve  --trace` or `jekyll serve` then connect to (http://localhost:4000) and check to see if your page looks right! 

**7.** Update git, and publish your post with s3_website `s3_website push`


That should be about it! 

[@tfhartmann](https://github.com/tfhartmann)
