---
layout: post
title: "Using Castle TypeFactory facility"
date: 2013-04-28 17:40
comments: true
share: true
tags:
- csharp
- "castle windsor"
- tdd
---
Recently we ran into a problem with unit testing a piece of C# code because of the way the objects were created within the class under test. We were using Castle Windsor to inject the dependencies already but this particular situation was troublesome for us.
<!--more-->

Consider the following sample code:

{% codeblock lang:csharp AccountFinder.cs %}
public class AccountFinder
{
	private IAccountRepository repository;

	public AccountFinder(IAccountRepository repository)
	{
		this.repository = repository;
	}

	public Account Find(SearchCriteria criteria)
	{
		var searchStrategy = new AccountSearchStrategyBuilder(repository).Create(criteria);
		return searchStrategy.Find(criteria);
	}
}
{% endcodeblock %}

The `AccountFinder` depends on the following bit of code to create the right kind of strategy.

{% codeblock lang:csharp AccountSearchStrategyBuilder.cs %}
public class AccountSearchStrategyBuilder
{
	private IAccountRepository repository;

	public AccountSearchStrategyBuilder(IAccountRepository repository)
	{
		this.repository = repository;
	}

	public AccountSearchStrategy Create(SearchCriteria criteria)
	{
		if (critera.CustomerId != null)
			return new AccountSearchByCustomerIdStrategy(repository);
		return new DefaultAccountSearchStrategy(repository);
	}
}
{% endcodeblock %}

As you can see in the `AccountSearchStrategyBuilder` implementation, we end up creating the instances of the strategies themselves because we can decide on the type of the strategy to create only after we inspect the `SearchCriteria`. So, that implies testing the `AccountSearchStrategyBuilder` would mean creation of the various implementations even though we do not really need them. And the builder needs to be injected with all the dependencies that are required by the individual strategy implementations. Yikes!!

So how do we solve the problem? Castle Windsor has a [facility](https://github.com/castleproject/Windsor/blob/master/docs/typed-factory-facility.md) specifically to address this particular problem. The idea is we delegate the creational responsibility to the container whilst still being able to select a particular implementation based on the input available to us at runtime.

The new implementation of the StrategyBuilder looks like

{% codeblock lang:csharp AccountSearchStrategySelector.cs %}
public class AccountSearchStrategySelector : DefaultTypedFactoryComponentSelector
{
	protected override Type GetComponentType(MethodInfo method, object[] arguments)
	{
		var searchCriteria = (SearchCriteria) arguments[0];
		if (searchCriteria.CustomerId != null)
			return typeof(AccountSearchByCustomerIdStrategy);
		return typeof(DefaultAccountSearchStrategy);
	}
}
{% endcodeblock %}
This way, we are separating the responsibility clearly - the creational logic of wiring in the right dependencies is not expressed in our code. It stays with Windsor. But how do we use this in our `AccountFinder` code? Here is how

First we need an interface that Windsor can implement for the `AccountSearchStrategySelector`

{% codeblock lang:csharp IAccountSearchStrategyBuilder.cs %}
public interface IAccountSearchStrategyBuilder
{
	IAccountSearchStrategy Create(SearchCriteria criteria);
}
{% endcodeblock %}

This interface will not have an implementation in our codebase. It is purely used by Castle Windsor to generate a proxy which can be used in our `AccountFinder` implementation.

Our Castle Windsor configuration now looks like below:

{% codeblock lang:csharp %}
var container = new WindsorContainer();
container.AddFacility<TypedFactoryFacility>();
container.Register(Component.For<IAccountSearchStrategyBuilder>().AsFactory(f => f.SelectWith<AccountSearchStrategySelector>()));
container.Register(Component.For<AccountSearchStrategySelector, ITypedFactoryComponentSelector>());
{% endcodeblock %}

We are basically registering the `TypedFactoryFacility` with the container and then telling Windsor to use that factory to select an implementation type using the `.AsFactory` extension method when registering the `IAccountSearchStrategyBuilder`.

The newer version of `AccountFinder` looks like below

{% codeblock lang:csharp AccountFinder.cs %}
public class AccountFinder
{
	private IAccountRepository repository;
	private IAccountSearchStrategyBuilder searchStrategyBuilder;

	public AccountFinder(IAccountRepository repository, IAccountSearchStrategyBuilder searchStrategyBuilder)
	{
		this.repository = repository;
		this.searchStrategyBuilder = searchStrategyBuilder;
	}

	public Account Find(SearchCriteria criteria)
	{
		var searchStrategy = searchStrategyBuilder.Create(criteria);
		return searchStrategy.Find(criteria);
	}
}
{% endcodeblock %}

This way, we can test our individual classes easily by supplying mock/stubs instead of dealing with live instances that might require complicated setup.

#### Summary

Use this facility when you need to choose amongst a bunch of implementations based on input received at runtime. Next time you end up [Yak shaving](http://www.urbandictionary.com/define.php?term=yak+shaving), understand why you ended up doing it and see what can be done to ease the pain. And yeah, reading the manual helps too!
