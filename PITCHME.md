# Async in C&#35; 
##### How to succeed with asynchronous programming in .NET
###### (and other fun facts)

An informative rant by Aaron Holmes

+++

GitPitch slideshow: https://gitpitch.com/aholmes/gitpitch_async-await-csharp

GitPitch repository: https://github.com/aholmes/gitpitch_async-await-csharp

+++

#### A brief history of space and ~~time~~ Task
- Task Parallel Library (TPL)
	.NET >= 4.0
	- Task
	- Task.Factory.StartNew
- Task-based Asynchronous Programming (TAP)
	.NET >= 4.5
	- Task.Run
	- async and await (C# 5)

+++

#### Task Parallel Library (TPL)
- Provides high-level methods to simplify using threads for parallel processing and  concurrency |
- Introduced Task and friends (.Result(), .Wait(), et al) |
- Introduced Task.Factory.StartNew() |
- Stuck using (and blocking) threads |
	- Wastes CPU cycles waiting for IO

+++

#### Task-based Asynchronous Programming (TAP)
<ul>
<li class="fragment">
.NET 4.5 added Task.Run to make life easier
</li>
<li class="fragment">
Equivalent to
<span style="font-size:20px;">
```
Task.Factory.StartNew(A, CancellationToken.None, TaskCreationOptions.DenyChildAttach, TaskScheduler.Default);
```
</span>
</li>
<li class="fragment">
Who wants to remember this?<br/>

Instead use
<span style="font-size:20px;">
```
Task.Run(A);
```
</span>
</li>
</ul>

+++

#### C&#35; 5 and async/await

- Added async/await keywords |
	- Built on top of existing Task concepts
		- Continuations |
		- Task API |
		- Thread management |

+++

#### Task.Run

Task.Run is parallel, but not necessarily asynchronous
<div style="width:100%;text-align: left;">Context-specific definitions:</div>
- Parallel: executing concurrently via threads |
- Asynchronous: executing on hardware after releasing thread |
	- Releasing thread: sending the thread back to the managed threadpool

+++

#### Asynchronous

The async/await flow uses the Base Class Library (BCL) to send work to the OS and release threads to the threadpool

- Application |
	- BCL |
		- Overlapped IO |
			- OS |
				- IRP &lt;-- "continuation" |
					- Device driver &lt;-- thread released |

+++

#### Common issues and why they happen
Understand what happens with asynchronous programming in .NET

<div style="width:100%;text-align: left;">Will touch on:</div>
- History and baggage of Task |
- Async state machine |
- Thread management |
- Asynchronous vs Parallel |

+++

#### Common problems
###### Deadlocks

- Caused by blocking a thread while another thread attempts to continue
- Culprits: .Result, .Wait(), .GetAwaiter().GetResult(), any other sort of thread blocking
	- TARE: Thread-Abuse Resistence Education. Just say no!

+++
#### Common problems
###### Deadlocks in ASP.NET

- Caused by SynchronizationContext when blocking threads
	- One executing thread per request (SyncContext gating)
	- Continuation tries to execute two threads

+++
#### Common problems
###### Relying on ConfigureAwait(false)

> Using ConfigureAwait(false) to avoid deadlocks is a dangerous practice. You would have to use ConfigureAwait(false) for every await in the transitive closure of all methods called by the blocking code, including all third- and second-party code. Using ConfigureAwait(false) to avoid deadlock is at best just a hack).

https://blog.stephencleary.com/2012/07/dont-block-on-async-code.html

+++

#### Succeeding with Async
###### Concepts needed to use async/await successfully

+++
#### Succeeding with Async

- One big state machine
	- MSBuild generates IL |
	- State machine contains continuations |
	- Manages async Task object |

+++
#### Understanding async/await
Never block! Instead, use:
- await |
	- Unwraps Tasks and returns internal result
- WhenAny |
	- Returns when any task in an IEnumerable<Task> completes
	- Returns immediately when any Task fails
- WhenAll |
	- Returns when all tasks in an IEnumerable<Task> complete
	- Failures do not cause an early return
+++

#### Understanding async/await
ContinueWith()

- Set up your own continuations |
- Easy way to handle a result immediately after a Task finishes |
	- One place .Result is safe
	- ContinueWith(t => t.Result);

+++
#### Understanding async/await
Cancellation Tokens

- Stop in-progress work |
- Respond in Tasks when something happens |

+++
#### Understanding async/await

Task<T> extends Task

+++
#### Understanding async/await
##### AggregateException
- Catching and handling this and other exceptions
	- (use `when()` - it's awesome)

+++
#### Understanding async/await
##### What to do in the monolith
- async is possible in most areas
	- if not, "why aaron keeps saying use Task.Run(Action).Result"

+++
#### Understanding async/await
##### use async everywhere
###### but be smart about it
- try to avoid async/await when using TPL or "threaded tasks"

+++
#### Understanding async/await
##### achieve parallelization when possible
###### use responsibly
- don't exhaust resources like network ports or disk io with too many parallel operations

+++
#### Understanding async/await
- always configureawait false in your library code
- don't rely on context capturing in you need it (e.g. global HttpContext)

+++
#### Understanding async/await
- consider adopting an "always use cancellation tokens" policy

+++
#### Understanding async/await
- permutations on the use of Task/Task<T> and async/await
	
+++
#### Understanding async/await
- Task pass-through
	- exceptions again

+++
#### Understanding async/await
- Task[] and different Task<T> types
	
+++
#### Understanding async/await
- using .Result after task completion (dangerous!)
	- exceptions again again
- using await after task completes

+++
#### Understanding async/await
- Unwrapping Task<Task> and Task<Task<T>>
	
+++
#### Understanding async/await
- async in xUnit - do it! use supporting asserts
