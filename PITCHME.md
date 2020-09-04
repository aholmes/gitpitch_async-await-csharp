# Async in C&#35; 
##### Succeeding with async programming in C&#35;
###### (and other fun facts)

A passion project by Aaron Holmes

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
- Other
	- Asynchronous Programming Model (APM)
	- Event-based Asynchronous Programming Model (EAP)

+++

#### Task Parallel Library (TPL)
@ol
- High-level methods to simplify using threads for parallel processing and  concurrency
- `Task` and friends (`.Result()`, `.Wait()`, et al)
- `Task.Factory.StartNew()`
- Stuck with blocking threads
@olend

+++

#### Task-based Asynchronous Programming (TAP)
<ul style="list-style: none;">
<li class="fragment">
.NET 4.5 added Task.Run
<br />
Equivalent to
<span style="font-size:20px;">
```
Task.Factory.StartNew(Action, CancellationToken.None, TaskCreationOptions.DenyChildAttach, TaskScheduler.Default);
```
</span>
</li>
<li class="fragment">
Who wants to remember this?<br/>

Instead use
<span style="font-size:20px;">
```
Task.Run(Action);
```
</span>
</li>

<li class="fragment">Can we do better?</li>
</ul>

+++

#### C&#35; 5 and async/await

@ol
- Added async/await keywords<br/>Built on top of existing Task concepts
	- Continuations
	- Task API
	- Thread management
@olend

+++

#### Task.Run

Task.Run is _parallel_<br/>not always _asynchronous_
<div style="width:100%;text-align: left;">Definitions:</div>
- Parallel: concurrency via threads
- Asynchronous: no thread, hardware interrupts
+++

#### Asynchronous

Async/await tells the OS to send some work to hardware, then releases a thread

<div style="width:100%;text-align: left;">Flow:</div>
- Application |
	- BCL |
		- Overlapped IO |
			- OS |
				- IRP (Input Output Request) &lt;-- "continuation" |
					- Device driver &lt;-- thread released |

+++

#### Common issues and why they happen
What makes asynchronous programming in C# hard?

- History and baggage of Task
- Async state machine
- Thread management
- Asynchronous vs Parallel

+++

#### Common problems
###### Deadlocks

TARE: Thread-Abuse Resistence Education.<br/>Just say no!

- Blocking a thread when another thread receives an IRP
- Culprits:
	- .Result
	- .Wait()
	- .GetAwaiter().GetResult()
	- any other sort of thread blocking

+++
#### Common problems
###### Deadlocks in ASP.NET

- SynchronizationContext
	- One executing thread per request (SyncContext gating)
	- Continuation tries to execute two threads

No longer a problem in ASP.NET Core!

+++
#### Common problems
###### Relying on ConfigureAwait(false)

<blockquote style="font-size:15px; line-height:1px; display:block;">
<p>
Using ConfigureAwait(false) to avoid deadlocks is a dangerous practice. You would have to use ConfigureAwait(false) for every await in the transitive closure of all methods called by the blocking code, including all third- and second-party code. Using ConfigureAwait(false) to avoid deadlock is at best just a hack).
<p>
https://blog.stephencleary.com/2012/07/dont-block-on-async-code.html
</blockquote>

+++

#### Succeeding with Async
###### How to use async/await successfully

+++
#### Succeeding with Async

- One big state machine
	- Compiler generates IL |
	- State machine contains continuations |
	- Manages async Task object |

+++
#### Understanding async/await
Never block! Instead, use:
- await |
	- Returns the result of a `Task`
- WhenAny |
	- Returns when any many `Task`s complete
	- Returns immediately when any `Task` fails
- WhenAll |
	- Returns when all `Task`s complete
	- Failures do not cause an early return
+++

#### Understanding async/await
ContinueWith()

- Set up your own continuations |
- Useful to transform a Task result |
- One place .Result is safe |
	- ContinueWith(t => t.Result);

+++
#### Understanding async/await
Cancellation Tokens

- Stop in-progress work |
- Respond in Tasks when something happens |

+++
#### Understanding async/await

`Task<T>` extends `Task`

+++
#### Understanding async/await
`AggregateException`
- Catching and handling this and other exceptions
	- (use `when()` - it's awesome)

+++
#### Understanding async/await
##### What to do in old non-async code
- `async` is possible in most areas
	- New code should be async
- if `async` can't be used, understand why Aaron keeps saying use `Task.Run(Action).Result`

+++
#### Understanding async/await
##### use async everywhere
###### but be smart about it
- Try to avoid async/await when using TPL or "threaded `Task`s"
- Avoid "sync-over-async"

+++
#### Understanding async/await
##### achieve parallelization when possible
###### use responsibly
- `async` can exhaust IO<br/>(sockets, file handles, disk r/w, bandwidth)
	- Use a custom `TaskScheduler`

+++
#### Understanding async/await
- always `ConfigureAwait` false in your library code
- Don't rely on context capturing if you need it (e.g. global HttpContext)
	- Use a wrapper class!

+++
#### Understanding async/await
- Consider adopting an "always use cancellation tokens" policy

+++
#### Understanding async/await
- Task pass-through
	- This changes exceptions!

```
Task DoSomething()
{
	return SomethingAsync();
}
```

+++
#### Understanding async/await
- `Task[]` and `Task<T>` polymorphism

```
var tasks = new Task[]
{
	GetStringAsync(), // returns Task<string>
	GetIntAsync() // returns Task<int>
};

await Task.WhenAll(tasks);
```
	
+++
#### Understanding async/await
- Using .Result after `Task` completion (dangerous!)
	- This changes exceptions!
- using await after `Task` when task is started elsewhere

```
var task = DoSomethingAsync();
// wait for task to complete
return task.Result; // this is technically safe but has risk
```

```
var task = DoSomethingAsync();
// do something else
return await task;
```

+++
#### Understanding async/await
- Unwrapping `Task<Task>` and `Task<Task<T>>`
	
```
	// some wrapped task
	Task<Task<string>> task = await GetStringAsync() =>

	return await task.Unwrap(); // get the string
```

+++
#### Understanding async/await
- Async in unit and integration tests
	- xUnit has `async` asserts

+++
# Thank you!
	await Questions();
