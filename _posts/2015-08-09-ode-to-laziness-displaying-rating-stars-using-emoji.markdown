---
layout: post
published: true
title:  "Ode to laziness - displaying rating stars ⭐️ using emoji 😍"
date:   2015-08-09 11:10:00
categories: andilabs blog objective-c python django rest maps ratings stars emoji
---

I am wokring on [dogspot](http://dogspot.eu/), which is an application for reviewing dog friendly spots.
One of the feature is ability to rate the spots by users.

Now I create read-only app - a kind of radar for finding dog-friendly spots nearby and showing it on map.

![emoji-ratings](/assets/emoji-rating/demo.png)

ODE:
----

Backend:
My python/django api is returning float representing friendly_rate.

IOS:
There is a lot of cool libs for ratings, e.g:

[EDStarRating](https://github.com/erndev/EDStarRating), 

[DYRateView](https://github.com/dyang/DYRateView), 

[HCSStarRatingView](https://github.com/hugocampossousa/HCSStarRatingView)

**BUT**: This have to be read-only app for now. So I decided to mocke rating with static stars. First idea: use static star images. Better idea: **use emoji** 😍!
I am using [Google Maps iOS SDK](https://developers.google.com/maps/documentation/ios/). I was afraid it will have problems with rendering snippet containg emoji but it worked like a charm!

I am rounding float, and want display only integer rating. Then I need only display ⭐️ as many times as value of rating.
{% highlight objective-c %}
[@"" stringByPaddingToLength:[@"⭐️" length]*lroundf([marker[@"friendly_rate"]floatValue])
                  withString: @"⭐️" startingAtIndex:0]
{% endhighlight %}
Emoji `[@"⭐️" length]` is 2 not 1!

Another aproach, prepare text stars on backend in django-rest-frameworks serializers `to_represenation` method
{% highlight python %}
ret['friendly_rate_stars'] = '*'*int(round(instance.friendly_rate))
{% endhighlight %}

and then just make simply replacment:
{% highlight objective-c %}
[marker[@"friendly_rate_stars"] stringByReplacingOccurrencesOfString:@"*" withString:@"⭐️"]
{% endhighlight %}
