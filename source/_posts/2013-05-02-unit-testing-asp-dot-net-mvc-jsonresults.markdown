---
layout: post
title: "Unit Testing ASP.Net MVC JSONResults"
date: 2013-05-02 17:40
comments: true
categories: [ASP.Net, MVC, Unit Testing]
---
Let's say we have a controller action that returns a `JsonResult` with an anonymous type:

```c#
public JsonResult Customer(int id)
{
    var customer = new { Id = id, Name = "John" };

    return Json(customer, JsonRequestBehavior.AllowGet);
}
```
<!--more-->
We'd like to test the response (using [NUnit](www.nunit.org)).  We have a number of options here.

##The 'Correct' (AKA long) way
This approach is how the MVC framework tests work.  You can browse though it [here](http://aspnetwebstack.codeplex.com/SourceControl/latest#test/System.Web.Mvc.Test/Test/JsonResultTest.cs).  Here we are using [Moq](https://code.google.com/p/moq/) to mock out the `HttpResponseBase` and the `ControllerContext`.  This allows us to grab what would normally be written to the `HttpContext.Response` and redirect into a `StringBuilder` object which we can interrogate later.

```c#
[Test]
public void GetCustomer_WhenGivenAnId_ReturnsValidCustomer()
{
    var controller = new CustomerController();

    var result = controller.Customer(1);

    var sb = new StringBuilder();
    var mockResponse = new Mock<HttpResponseBase>();
    mockResponse.Setup(s => s.Write(It.IsAny<string>())).Callback<string>(c => sb.Append(c));

    var mockControllerContext = new Mock<ControllerContext>();
    mockControllerContext.Setup(x => x.HttpContext.Response).Returns(mockResponse.Object);

    result.ExecuteResult(mockControllerContext.Object);

    Assert.AreEqual(@"{""Id"":1,""Name"":""John""}", sb.ToString());    
}
```

We can make this a little easier to read and maintain by adding a serializer:
```c#
[Test]
public void GetCustomer_WhenGivenAnId_ReturnsValidCustomer()
{
    var serializer = new JavaScriptSerializer();
    var controller = new CustomerController();

    var result = controller.Customer(1);

    var sb = new StringBuilder();
    var response = new Mock<HttpResponseBase>();
    response.Setup(s => s.Write(It.IsAny<string>())).Callback<string>(c => sb.Append(c));

    var controllerContext = new Mock<ControllerContext>();
    controllerContext.Setup(x => x.HttpContext.Response).Returns(response.Object);

    result.ExecuteResult(controllerContext.Object);

    var dynamicResponse = serializer.Deserialize<dynamic>(sb.ToString());

    Assert.AreEqual(1, dynamicResponse["Id"]);
    Assert.AreEqual("John", dynamicResponse["Name"]);
}
```

##Utilising a serialiser
We are getting back a `JsonResult.Data` object.  It is is still an anoymous object and has not yet been serialized by the `HttpResponseBase` object.  The object looks like this:

```c#
{ Id = 1; Name = "John" }
```
rather than this:

```javascript
{ "Id" : "1"; "Name" : "John" }
```

Two approaches here.  This first approach (much like the first test above) results in a pretty unreadable/unmaintainable test even providing your `JsonResult.Data` is small.  It is very easy to get the string output wrong and cause a false failing test.  It is quick and easy though.

```c#
[Test]
public void GetCustomer_WhenGivenAnId_ReturnsValidCustomer()
{
    var serializer = new JavaScriptSerializer();
    var controller = new CustomerController();

    var response = controller.Customer(1);

    var serializedData = serializer.Serialize(response.Data);

    Assert.AreEqual(@"{""Id"":1,""Name"":""John""}", serializedData);
}
```

This test is slightly better in that it is more readable but requires us to serialise the `response.Data` into a string, cast into a dynamic then deserialize back to a dynamic.

```c#
[Test]
public void GetCustomer_WhenGivenAnId_ReturnsValidCustomer()
{
    var serializer = new JavaScriptSerializer();
    var controller = new CustomerController();

    var response = controller.Customer(1);

    var obj = serializer.Serialize(response.Data) as dynamic;
    var customer = serializer.Deserialize<dynamic>(obj);

    Assert.AreEqual(1, customer["Id"]);
    Assert.AreEqual("John", customer["Name"]);
}
```

##Reflection
This test uses `Reflection` to get to the `JsonResult.Data`.  The code here could obviously be abstracted away into an extension method or some other such way but the bones of it are as follows.

```c#
[Test]
public void GetCustomer_WhenGivenAnId_ReturnsValidCustomer()
{
    var controller = new CustomerController();

    var response = controller.Customer(1);
    var dataType = response.Data.GetType();

    Assert.AreEqual(1, dataType.GetProperty("Id").GetValue(response.Data));
    Assert.AreEqual("John", dataType.GetProperty("Name").GetValue(response.Data));
}
```
    
##RouteValueDictionary
We can so it using a much easier approach using `RouteValueDictionary`.  I  quite like this approach although it is a little counter intuitive to use the `RouteValueDictionary` class for accessing an anonymous type.

```c#
[Test]
public void GetCustomer_WhenGivenAnId_ReturnsValidCustomer()
{
    var controller = new CustomerController();

    var response = controller.Customer(1);
    var customer = new RouteValueDictionary(response.Data);

    Assert.AreEqual(1, customer["Id"]);
    Assert.AreEqual("John", customer["Name"]);
}
```

##dynamic and `InternalsVisibleTo`
---
Finally, we can use `dynamic`.  I much prefer the testing syntax of this approach but (there is always a 'but') there is an issue.  `JsonResult.Data` is an anonymous type and these are, by default, declared as `internal` to their assembly and most tests are declared in a seperate project so no access to internal anonymous types in the controller assembly.  By the time we get access to the `JsonResult.Data` we see it as an object.  

If we run our test below, we will get a `RuntimeBinderException` exception `'object' does not contain a definition for 'Id`.

```c#
[Test]
public void GetCustomer_WhenGivenAnId_ReturnsValidCustomer()
{
    var controller = new CustomerController();

    var response = controller.Customer(1);
    dynamic customer = response.Data;

    Assert.That(1, customer.Id);
    Assert.That("John", customer.Name);
}
```

Adding `[assembly: InternalsVisibleTo("MyTestProject")]` attribute on the web project `AssemblyInfo.cs` class (in the `Properties` folder) will allow our test to access the `JsonResult.Data` anonymous type.  Re-run the test and all green.