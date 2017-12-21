# Async in C&#35; 
##### How to succeed with asynchronous programming in .NET
###### (and other fun facts)

+++

# Common issues and why they happen
Understand what happens with asynchronous programming in .NET

<div style="width:100%;text-align: left;">Will touch on:</div>
- History and baggage of Task |
- Async state machine |
- Thread management |
- Asynchronous vs Parallel |

+++

# A brief history of space and Task
- Task Parallel Library (TPL)
	.NET >= 4.0
	- Task
	- Task.Factory.StartNew
- Task-based Asynchronous Programming (TAP)
	.NET >= 4.5
	- Task.Run
	- async and await (C# 5)

+++

# Task Parallel Library (TPL)
- Provides high-level methods to simplify using threads for parallel processing and  concurrency |
- Introduced Task and friends (.Result(), .Wait(), et al) |
- Introduced Task.Factory.StartNew() |
- Stuck using (and blocking) threads |
	- Wastes CPU cycles waiting for IO

+++

# Task-based Asynchronous Programming (TAP)
<ul>
<li class="fragment">
.NET 4.5 added Task.Run to make life easier
</li>
<li class="fragment">
Equivalent to <br/>	
<span style="font-size:20px;">
```
Task.Factory.StartNew(A, CancellationToken.None, TaskCreationOptions.DenyChildAttach, TaskScheduler.Default);
```
</span>
</li>
<li class="fragment">
Who wants to remember this?<br/>

Instead use <span style="font-size:20px;">Task.Run(A);</span>
</li>
</ul>

+++

#C&35; 5

- Added async/await keywords |
	- Built on top of existing Task concepts
		- Continuations |
		- Task API |
		- Thread management |

+++

# Task.Run

Task.Run is parallel, but not necessarily asynchronous
Context-specific definitions:
- Parallel: executing concurrently via threads |
- Asynchronous: executing on hardware after releasing thread |
	- Releasing thread: sending the thread back to the management threadpool

+++

# Asynchronous magic

The async/await flow uses the Base Class Library (BCL) to send work to the OS and release threads to the threadpool

- Application |
	- BCL |
		- Overlapped IO |
			- OS |
				- IRP | &lt;-- "continuation"
					- Device driver | &lt;-- thread is finally released

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
