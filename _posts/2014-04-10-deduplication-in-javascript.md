---
layout: post
title: Deduping arrays in javascript
subtitle: herple derple herple derple herple derple
permalink: /blog/dedupe-arrays
excerptLong: herple derple herple derple herple derple herple derple herple derple herple derple herple derple herple derple herple derple herple derple 
excerptShort: herple derple
hero: photo-chicago-train-chicago-and-midland-illinois-steam-locomotive-602-and-tender.jpg
hero-adj: "background-position: center center;"
---

Knowing how to deduplicate arrays is pretty useful, and it can be handled in a variety of ways.  This is my favorite.  It uses two loops, many random "Archer" references, and a cool characteristic of objects to get the job done.<BREAK>

{% highlight javascript linenos=table %}
function dedupe(array) {
  //declares dedupe object
  var dedupeObj = {}
  for (var i = 0; i < array.length; i++) {
    //stores each array element
    dedupeObj[array[i]] = true
  }
  //empties array
  array = [];
  //refills array with the object keys
  for (var key in dedupeObj) {
    array.push(key);
  }
  return array;
}
{% endhighlight %}

First, the function maps each item in the array to an object key, and sets the key's value equal to true.  We could just as easily set that value to false or 5 or 'blue' or whatever we wanted, all that matters is we're storing the array as keys.  Object keys are always unique.

{% highlight javascript linenos=table %}
var array = ['archer','lana','pam','lana','cyril']

for (var i = 0; i < array.length; i++) {
  dedupeObj[array[i]] = true
}

//Result: {archer: true, lana: true, pam: true, cyril: true}
//Notice only one LAAAANAAAAAAAAAAAA
{% endhighlight %}

So when we loop through the array setting elements to object keys, if two elements in the array are both 'lana', the object will get a key dedupeObj.lana set to true, and then the loop will set dedupeObj.lana to true again.  Uniqueness!

To finish up, we empty the array, and use a simple for-in loop to reload the array with the object's keys, yielding an array with only one of each element.

###what about an array of objects?

If you want to deduplicate an array of objects, we'll need to do something slightly different.  If we used the above method, objects in the array ALL get stored in the deduplicator as <code style="font-size: 60%; padding: 5px;">[object Object]</code>.  So the deduplication doesn't work, and the data or archer references stored in that object will be lost.

We need to add in conditionals to account for this:

{% highlight javascript linenos=table %}

var example = [{"name":"barry","email":"barry@kgb.ru"},
              {"name":"barry","email":"otherbarry@kgb.ru"},
              {"name":"nikolai","email":"nikolai@kgb.ru"},
              {"name":"nikolai","email":"nikolai@kgb.ru"},
              7,
              'asdf',
              'asdf',
              8];

function dedupe(array) {
  var dedupeObj = {}
  var itemKey;
  for (var i = 0; i < array.length; i++) {
      //make a key of the object's own keys and values
      if(typeof array[i] === 'object'){
          itemKey = array[i].name + "|" + array[i].email;
          dedupeObj[itemKey] = array[i];
      } else {
          dedupeObj[array[i]] = true
      }
  }
  array = [];
  for (var key in dedupeObj) {
      //rebuild the object in the returned array
      if(typeof dedupeObj[key] === 'object'){
          array.push(dedupeObj[key]);
      }
      else {
          array.push(key);
      }
  }
    return array;
}

dedupe(example);

{% endhighlight %}

First we use <code style="font-size: 60%; padding: 5px;">typeof</code> to test if the array element is an object.  If so, the object's key in dedupeObj is a concatenation of that objects own keys and values, to make sure it's unique.  We need to have a conditional in the second loop as well, because this time we need to push a value to the rebuilt array, not a key.

One thing to keep in mind when deduping an array that contains objects, is you need a sort of schema.  You need to know what to expect in each object in order to build the deduplicating key <code style="font-size: 60%; padding: 5px;">itemKey = array[i].name + "|" + array[i].email</code>.

If you have a better or different way of doing things (perhaps with the <code style="font-size: 60%; padding: 5px;">array.filter()</code> method?) leave them in the comments below!
