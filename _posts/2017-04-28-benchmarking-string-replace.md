---
layout: post
title: Benchmarking fun with string.Replace method
date: 2017-04-28 00:11:29.000000000 -05:00 
tags: [Benchmarking, Optimization]
---

I was working on some code which gets a list of strings where each item holds a domain user name. This list is being rendered in an mvc view. For readability, we wanted to remove the repetitive domain name from the string value. The easy approach is using `String.Replace` method to replace the domain name with empty string   

{% highlight c# %}
var input = @"MyBoringDomainName\skrishnankutty";
return input.Replace(@"MyBoringDomainName\", "");
{% endhighlight %}

Could this be improved ? What if you know that the domain name is always same and it is at the start of your string ? We could get a substring which ignores the chars neeeded for the string we want to remove/replace.


{% highlight c# %}
var l = "MyBoringDomainName".Length;
var input = @"MyBoringDomainName\skrishnankutty";
return input.Substring(l);
{% endhighlight %}

How do we know our second approach is better? We need to measure the behavior of both the methods. I have been using Benchmark dot net lately for similar things and absolutely love it.

Here is the result when i ran benchmark. Substring method was set as the baseline method.


{% highlight c# %}
 |    Method |     Mean |     Error |    StdDev | Scaled | ScaledSD |
 |---------- |---------:|----------:|----------:|-------:|---------:|
 |   Replace | 83.59 ns | 1.8246 ns | 4.5775 ns |   5.19 |     0.42 |
 | Substring | 16.16 ns | 0.3839 ns | 0.9701 ns |   1.00 |     0.00 |

{% endhighlight %}

1 to 5.19. Wow! that is nice. Imagine dealing with thousands of instances of the above string. 

Substring is doing a linear traversal on the input char array. But Replace method which internally needs to search for the substring, has to do something even complex than that.

#### Conclusion

This does not mean that we need to replace every instance of `Replace` method with `SubString` in the code base. As i mentioned earlier, i felt it ideal to do a substring in my specific usecase where i knew the length and position of the domain name.

Readability  is also an important part of good code. So find the balance between performance and readability and use it sensibly.    

Cheers


