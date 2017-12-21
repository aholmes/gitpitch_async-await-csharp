# Async in C#
##### How to succeed with asynchronous programming in .NET
###### (and other fun facts)

+++

### Common issues and why they happen
Understand what happens with asynchronous programming in .NET
- TPL
- TAP |
- Task.Factory.StartNew |
- Task.Run |
- async and await |
+++
### A brief history of space and Task

		- TPL, TAP, & Task.Factory.StartNew
			- task parallel library
			- define TAP Task Based Asynchronous Programming
+++
		- .NET 4.5 added Task.Run to make life easier (defaults for Task.Factory.StartNew, and safer)
			- Task.Factory.StartNew(A, CancellationToken.None, TaskCreationOptions.DenyChildAttach, TaskScheduler.Default);
			- Task.Run(A);
+++
			- C# 5 added async/await keywords
			- explain why Microsoft did not remove old TAP methods, and why async/await is built on top of them
+++
		- Task.Run is parallel, but not necessarily asynchronous
			- define parallel and asynchronous: IO-bound work where thread can detach
				- BCL -> Overlapped IO -> OS -> IRP -> device driver
+++
	- deadlocks
		- .Result, .Wait(), .GetAwaiter().GetResult, any other sort of thread blocking
			- define thread blocking
			- safe at top level (like Main in console app)
+++
		- in ASP.NET
			- define sync context
			- synchronization context & flow diagram
+++
		- configureawait false
			- why not to rely on this
+++
- succeeding
	- concepts needed to use async/await successfully
+++
		- async and await keywords
			- state machine
+++
		- WhenAny/WhenAll
+++
		- continuewith
+++
		- cancellationtokens
+++
		- Task<T> extends Task
+++
		- AggregateException
			- Catching and handling this and other exceptions
				- (use `when()` - it's awesome)
+++
	- what to do in the monolith
		- async is possible in most areas
+++
			- if not, "why aaron keeps saying use Task.Run(Action).Result"
+++
	- use async everywhere, but be smart about it
+++
		- try to avoid async/await when using TPL or "threaded tasks"
+++
		- achieve parallelization when possible, use responsibly
			- don't exhaust resources like network ports or disk io with too many parallel operations
+++
		- always configureawait false in your library code
			- don't rely on context capturing in you need it (e.g. global HttpContext)
+++
		- consider adopting an "always use cancellation tokens" policy
+++
	- permutations on the use of Task/Task<T> and async/await
+++
		- Task pass-through
			- exceptions again
+++
		- Task[] and different Task<T> types
+++
		- using .Result after task completion (dangerous!)
			- exceptions again again
		- using await after task completes
+++
		- Unwrapping Task<Task> and Task<Task<T>>
+++
		- async in xUnit - do it! use supporting asserts
