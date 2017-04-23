---
layout: default
title:  "Downloading all of your Instagram photos"
date:   2017-04-22 4:20:11 -0700
categories: writings
---

## I needed to backup Instagram photos

My fiancée was complaining how she wasn't able to easily download her own
Instagram photos, and none of the online services I tried wanted to really work
for me, or they were unverified applications that could only download the first
few photos. My first attempt was to check Github and came across a
[promising python repo](https://github.com/norm/instagram-backup/), but
that seemed to have the same limitation in the app not being verified.

---

## It's a somewhat manual process

#### Ok let's begin:

1. Head over to your Instagram page (`https://instagram.com/USERNAME`) and open
your developer tools. All image urls come from a request to a `query` endpoint
2. Check out the response from a request and you'll see a few `display_src`
keys. Sweet! This is just what we need. Without doing anything else, scroll
down the entire Instagram page so that the browser requests all of your images
3. Search for `"query"` in the network search bar to filter just the data we
want to save and right click any `query` request and then choose
`Save as HAR with content`. Open the file in your favorite text editor
4. Use your editors magic and retrieve all of the image urls (`display_src`)
from the file. Atom users, beware it's size is most-likely enough to crash 😞
5. Put all image urls into an array, and create a new file `instagram-backup.rb`

## Let's get downloading

This is the easy part. Copy the below script into the `instagram-backup.rb` file
you just created and replace `IMAGE_URLS` with the array of urls you saved
from step 5.

{% highlight ruby %}
require "open-uri"

IMAGE_URLS = [] # insert image urls here

IMAGE_URLS.each do |image_url|
  File.open("#{image_url.tr("/","_")}", 'wb') do |f|
    f.write open(image_url).read
  end
end
{% endhighlight %}

Save and run the script:
`$ ruby instagram-backup.rb`

Your images will download and my fiancée is happy 🎉


{% include analytics.html %}
