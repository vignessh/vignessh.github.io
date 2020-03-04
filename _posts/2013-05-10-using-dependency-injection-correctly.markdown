---
layout: post
title: "Using dependency injection correctly"
date: 2013-05-10 16:33
comments: true
share: true
categories: [c#, tdd]
---
Even though dependency injection is a better understood concept now than in the past, there is still code being written that does not keep the principle in mind. The problem seems to be around recognizing which dependencies are needed when constructing the object and which ones need resolution at runtime.

<!-- more -->
To better explain the problem, here is a sample piece of code.

``` c#
public class BusinessProfileFinder
{
  private readonly IEnquiryService enquiryService;
  private readonly Parameter parameter;
  private readonly string businessReferenceCode;

  public BusinessProfileFinder(IEnquiryService enquiryService, Parameter parameter, string businessReferenceCode)
  {
    this.enquiryService = enquiryService;
    this.parameter = parameter;
    this.businessReferenceCode = businessReferenceCode;
  }

  public BusinessProfile Find()
  {
    return enquiryService.Find(new BusinessProfileRequest {Parameter = parameter, ReferenceCode = businessReferenceCode});
  }
}
```

So this finder is just a wrapper over the actual service `EnquiryService` that is responsible for retrieving the `BusinessProfile` given certain parameters. At first glance, there is nothing really wrong with the way the above code has been written. But if you look closely, there are some issues. The constructor is taking arguments that are required for both construction as well as for the operation of this class.

Because of this, the caller of this class cannot express this as a dependency when configuring the DI container. Instead it will need to instantiate it at runtime by itself. Thereby we lose the ability to wire up dependencies, as well lose the ability to test it without resorting to hacks.

The same code could be re-written in a more test/dependency injection friendly way as below:

``` c#
public class BusinessProfileFinder
{
  private readonly IEnquiryService enquiryService;

  public BusinessProfileFinder(IEnquiryService enquiryService)
  {
    this.enquiryService = enquiryService;
  }

  public BusinessProfile Find(Parameter parameter, string businessReferenceCode)
  {
    return enquiryService.Find(new BusinessProfileRequest {Parameter = parameter, ReferenceCode = businessReferenceCode});
  }
}
```

Notice the change to the arguments to the constructor and the `Find` implementation. Looking at example, it becomes evident as to how writing test friendly code can also help with how dependencies are expressed and therefore help with design of the class under concern. It is important to understand the difference between dependencies that can be resolved during instantiation time vs. those that can only be resolved during run time.
