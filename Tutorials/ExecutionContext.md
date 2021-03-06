During execution sometimes one needs information that are cross cutting the application. 
Such as who is the currently logged in user, current culture, what is the current tenant and other custom meta
data you want is relative to the environment.

In Bifrost we have formalized this through something called execution context. 
To get the current execution context, you can either take a dependency directly to an interface
called IExecutionContext. 

	using Bifrost.Execution;

	public class MySystem
	{
		public MySystem(IExecutionContext executionContext)
		{
			// Put your code here
		}
	}


This is very useful if your system has a transient lifecycle, or in a web context; per web request. 
With a singleton lifecycle, this is not very useful at all, as you would get the execution context
that is valid at the first creation time. Any changes after that would not be reflected.
If you have a singleton, you can take a dependency to something called IExecutionContextManager

	using Bifrost.Execution;

	public class MySystem
	{
		public MySystem(IExecutionContextManager executionContextManager)
		{
			var current = executionContextManager.Current;

			// Put your code here
		}
	}

On the execution context you will find the following properties:

	IPrincipal	Principal;
	CultureInfo	Culture;
	string System;
	ITenant Tenant;
	dynamic Details;

The principal is populated by whatever resolver is configured for the system to resolve it. 
In the Configure instance, you will find a Security property that contains the type that will used
for resolving it.

Moving on to Culture, which is resolved through what the current thread has set. So whatever your
application is configured with.

System represents a string that just gives a name of your application - the currently running system.
It exists as a property on the configure object of Bifrost, also called System. During configuration
you simply set that property.

Tenant is resolved by the tenancy system, we will not go into details about that here.

The last property is a dynamic property called Details - it can only be populated by placing a class
implementing ICanPopulateExecutionContextDetails anywhere, in fact you can have multiple. 
Bifrot will discover all your implementations and call the Populate() method whenever the 
execution context needs to be populated.

	using Bifrost.Execution;

	public class MyPopulator : ICanPopulateExecutionContextDetails
	{
		public Populate(IExecutionContext executionContext, dynamic details)
		{
			details.SomeMetaData = "Hello world";
		}
	}


