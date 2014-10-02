---
layout: post
title: Simple web scraping with Node.js
subtitle: herple derple herple derple herple derple
permalink: /blog/simple-web-scraping-node
excerptLong: herple derple herple derple herple derple herple derple herple derple herple derple herple derple herple derple herple derple herple derple 
excerptShort: herple derple
hero: NY.jpg
hero-adj: "background-position: center center;"
---

**Note:** updated on 10/1/2014 to account for changes on unsplash, introducing a new layer of complexity.

Writing your first basic scraper feels incredibly empowering, and it's a fantastic way to get acquainted with Node.  I love that Node has allowed front end devs familiar with javascript to start doing fancy magic with a substantially lower learning curve, and simple web scraping is a great way to get started.<BREAK>

### First steps

{% highlight javascript linenos=table %}
var request = require('request');
var cheerio = require('cheerio');
var url = require('url');
var exec = require('child_process').exec;

(function() {
    var dlDir = './downloads/';
    var host = 'http://unsplash.com';
    
    request(host, function(err, resp, body) {
        if (err) throw err;

        $ = cheerio.load(body);
        $('.photo a').each(function(){
            var dlPage = $(this).attr('href');

            request(host+dlPage,function(err,resp,body){
                //redirects image
                var fileUrl = resp.request.uri.href;
                var fileName = url.parse(fileUrl).pathname.split('/').pop().split('?').shift();
                //in curl we have to escape '&' from fileUrl
                var curl =  'curl ' + fileUrl.replace(/&/g,'\\&') +' -o ' + dlDir+fileName + ' --create-dirs';
                var child = exec(curl, function(err, stdout, stderr) {
                    if (err){ console.log(stderr); throw err; } 
                    else console.log(fileName + ' downloaded to ' + dlDir);
                });
            })
        });
    });
})();
{% endhighlight %}

The scraper grabs the first ten photos off [unsplash.com](http://www.unsplash.com) (a fantastic resource for attribution free photos, like this post's hero image), so you don't have to visit the site and grab images individually, you can just grab that week's images in bulk.  The following code is available right here on my [github](https://github.com/elifitch/unsplash-scraper), and I recommend cloning my repo and following along.

This post assumes a very basic familiarity with the command line, but if you're not comfortable this is a [fantastic post](http://lifehacker.com/5633909/who-needs-a-mouse-learn-to-use-the-command-line-for-almost-anything) to get you started.

The first thing to do before we get started is to install node if you have not already.  You can [download it here](http://nodejs.org/download/) for whatever OS you use.

### Modules

{% highlight javascript linenos=table %}
var request = require('request');
var cheerio = require('cheerio');
var url = require('url');
var exec = require('child_process').exec;
{% endhighlight %}

After you have node downloaded and installed, we can dig into what's going on here.  The first block of code is a few require statements.  This tells node that you need certain modules.  Assuming you cloned my repo, cd into the directory and run the command <code style="font-size: 60%; padding: 5px;">npm install</code>.  Since I listed the dependencies in package.json, npm (node package manager) will download and install them for you.  How nice!

<hr>

<code style="font-size: 60%; padding: 5px;">Request</code> is a module that enables super simple http calls.

<code style="font-size: 60%; padding: 5px;">Cheerio</code> gives you all the magic of jQuery DOM traversal.

<code style="font-size: 60%; padding: 5px;">URL</code> gives you a really nice toolbox to parse and manipulate URLs.

<code style="font-size: 60%; padding: 5px;">Child_process.exec</code> (initialized here as exec) gives you the ability to run shell commands in your code. Neat!

<hr>

### Making a request

{% highlight javascript linenos=table %}
(function() {
    var dlDir = './downloads';
    var host = 'http://unsplash.com/';
    request(host, function(err, resp, body) {
        if (err) throw err;
        $ = cheerio.load(body);

        //Scraping function here

    });
})();
{% endhighlight %}

Now we're getting into the meat and potatoes of what the script actually does.  After determining where the images will be saved, and the host (in this case, unsplash), we make our request.  We feed request the host we want to hit, and there's a [callback](http://docs.nodejitsu.com/articles/getting-started/control-flow/what-are-callbacks) with three arguments: error, response, body.  Error gets thrown if there are any problems, response is the entire reply from the host (which is more than we need for this), and body is the returned HTML.  We load body into cheerio, and bind that to <code style="font-size: 60%; padding: 5px;">$</code> so it works just like jQuery.

### Ready set scrape

{% highlight javascript linenos=table %}
$('.photo a').each(function(){
    var dlPage = $(this).attr('href');

    request(host+dlPage,function(err,resp,body){
        //redirects image
        var fileUrl = resp.request.uri.href;
        var fileName = url.parse(fileUrl).pathname.split('/').pop().split('?').shift();
        //in curl we have to escape '&' from fileUrl
        var curl =  'curl ' + fileUrl.replace(/&/g,'\\&') +' -o ' + dlDir+fileName + ' --create-dirs';
        var child = exec(curl, function(err, stdout, stderr) {
            if (err){ console.log(stderr); throw err; } 
            else console.log(fileName + ' downloaded to ' + dlDir);
        });
    })
});
{% endhighlight %}

We know from checking out unsplash.com that we can access the images we want at <code style="font-size: 60%; padding: 5px;">$('.photo a')</code> using cheerio to allow jQuery syntax, and run a function on each one.  The full size image is stored on another page so we need to make a second request.  We append the href from the image link onto the host, and make a request to that location.

Unfortunately, there's one more step before we can get at the images we need.  The href from the image link on unsplash.com doesn't link directly to the image, it redirects first.  We use <code style="font-size: 60%; padding: 5px;">resp.request.uri.href</code> to find the redirected location, in this case the image itself, and set that value to <code style="font-size: 60%; padding: 5px;">fileUrl</code>.


The next step in saving our image is knowing what to save it as.  <code style="font-size: 60%; padding: 5px;">fileName</code> uses the URL module to get extract the name from the file url.  First we use <code style="font-size: 60%; padding: 5px;">url.parse()</code> to break the URL up into pieces that can be manipulated with other methods.  We then grab <code style="font-size: 60%; padding: 5px;">.pathname</code>, which is the portion of the URL that comes after the host, for example <code style="font-size: 60%; padding: 5px;">/assets/img/image.jpg</code>.  We split pathname into an array divided by '/' <code style="font-size: 60%; padding: 5px;">['assets','img','image.jpg']</code>, and pop the the last array item off.  This will always be the file name and extension!

Next we build a string to feed child_process as a command and assign it to <code style="font-size: 60%; padding: 5px;">var curl</code>.  Curl is a command line utility for making requests, downloading files, and [much more](http://curl.haxx.se/).  We start the command with <code style="font-size: 60%; padding: 5px;">'curl '</code>, nothing fancy there.  The next chunk, <code style="font-size: 60%; padding: 5px;">fileUrl.replace(/&/g,'\&')</code> escapes all the ampersands in <code style="font-size: 60%; padding: 5px;">fileUrl</code>'s query string, which includes vital a authorization.  Without escaping the ampersands, curl doesn't consider the query string as part of the url, and makes an unauthorized request, which obviously fails.  <code style="font-size: 60%; padding: 5px;">' -o '</code> is a flag that tells curl to write the the data to a file, <code style="font-size: 60%; padding: 5px;">dlDir+fileName</code> tells curl to save the file with the defined name within our downloads directory, and <code style="font-size: 60%; padding: 5px;">' --create-dirs'</code> forces curl to create the download directory if it doesn't already exist.

<blockquote>

If you like, you can type curl command into your terminal (don't forget to replace the variables with real values) just to see how it works individually.

</blockquote>

Exec will run the command string we just defined, and give us a callback to handle errors and outputs that are useful for debugging.  We wrap up by logging out the file names and download location as reference.

### Run it!

You run the script by entering the command <code style="font-size: 60%; padding: 5px;">node unsplash-scraper.js</code> in the root directory.  You should see the images downloading one by one, and have ten pretty pictures in your specified downloads folder.

Don't stop here!  Use this as a base for writing your own node scrapers, and post them in the comments!

If you have any thoughts, questions or suggestions, leave them in the comments or hit me up on [twitter](http://www.twitter.com/elifitch).