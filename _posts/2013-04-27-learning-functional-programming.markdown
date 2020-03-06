---
layout: post
title: "Learning functional programming - Part 1"
date: 2013-04-27 16:03
comments: true
share: true
tags:
- csharp
- clojure
- "functional programming"
- linq
---
Ever since I saw Rich Hickey's [Simple Made Easy](http://www.infoq.com/presentations/Simple-Made-Easy) presentation, I have been wanting to learn [Clojure](http://clojure.org). For the simple fact that it was designed by someone who has spent enough time thinking about problems faced with current generation of programming languages and the limitations they impose. <!--more-->

I've been intrigued by functional programming paradigm ever since my good friend enticed me to go through the book [**The Little Schemer**](http://mitpress.mit.edu/books/little-schemer). (If you have not read it yet, please do, it is a fascinating intro into understanding recursion). That journey started off a frantic rush which then tapered off after a while owing to various reasons. Now after going through Rich's presentation and after having spent a good part of the last 5 yrs building .NET based enterprisey apps, I'm beginning to feel the need to jump back to learning functional programming principles better. The experiments with C#'s Linq helped understand the principles to a great extent.

For example, instead of

{% highlight csharp linenos %}
var tasks = new List<Task>();
foreach(var user in users)
{
	var userTasks = Tasks.For(user.Id);
	tasks.AddRange(userTasks);
}
return tasks;
{% endhighlight %}

prefer something like

{% highlight csharp linenos %}
return users.Select(user => Tasks.For(user.Id)).SelectMany(task => task).ToList();
{% endhighlight %}

This might seem like syntactic sugar at first glance. That is because the functional paradigm have been transplanted onto a object oriented language like C# using Linq. In a pure functional language like [Clojure](http://clojure.org) or [Haskell](http://www.haskell.org), the syntax would feel more natural. Interestingly, the Linq implementation is purely via extension methods and not by extending the `IEnumerable<T>` interface. We'll explore this bit in a follow up post.
