# Async/Await, TPL, and Multi-Threading in C#
## A Guide with Examples and Best Practices

## Table of Contents
- **Async/Await Basics**
- **Task Parallel Library (TPL)**
- **Multi-Threading with Thread Class**
- **Advanced Async/Await Scenarios**
- **Fixing Deadlocks**
- **Using ConfigureAwait(false)**
- **Async Void vs. Async Task**
- **Submission Guidelines**
- **Key Takeaways**

---

## Async/Await Basics
### Task 1: Download Content Asynchronously
**Goal:** Use async/await to keep the app responsive during I/O-bound operations.

```csharp
using System;
using System.Net.Http;
using System.Threading.Tasks;

class Program
{
    static async Task Main(string[] args)
    {
        string url = "https://jsonplaceholder.typicode.com/posts/1";
        string content = await DownloadContentAsync(url);
        Console.WriteLine(content);
    }

    static async Task<string> DownloadContentAsync(string url)
    {
        using (HttpClient client = new HttpClient())
        {
            HttpResponseMessage response = await client.GetAsync(url);
            response.EnsureSuccessStatusCode();
            return await response.Content.ReadAsStringAsync();
        }
    }
}
```

**Key Things to Know:**
- Use `async` to mark a method as asynchronous.
- `await` pauses execution until the awaited task completes without blocking the main thread.
- Always dispose of `HttpClient` correctly (e.g., with `using`).

---

## Task Parallel Library (TPL)
### Task 2: Parallelize CPU-Bound Work
**Goal:** Use `Parallel.ForEach` to speed up operations.

```csharp
using System;
using System.Linq;
using System.Threading.Tasks;

class Program
{
    static void Main()
    {
        int[] numbers = Enumerable.Range(1, 10000).ToArray();
        Parallel.ForEach(numbers, number =>
        {
            Console.WriteLine($"{number} squared is {number * number}");
        });
    }
}
```

**Key Things to Know:**
- `Parallel.ForEach` splits work across multiple threads for CPU-bound tasks.
- Avoid thread-unsafe operations (e.g., writing to shared variables without locks).

---

## Multi-Threading with Thread Class
### Task 3: Run Operations Concurrently
**Goal:** Use `Thread` to execute methods in parallel.

```csharp
using System;
using System.Threading;

class Program
{
    static void Main()
    {
        Thread thread1 = new Thread(Method1);
        Thread thread2 = new Thread(Method2);

        thread1.Start();
        thread2.Start();

        thread1.Join(); // Wait for threads to finish
        thread2.Join();

        Console.WriteLine("All threads completed.");
    }

    static void Method1() => Console.WriteLine("Method1 running...");
    static void Method2() => Console.WriteLine("Method2 running...");
}
```

**Key Things to Know:**
- `Join()` blocks the calling thread until the target thread finishes.
- Threads are heavyweight—use TPL for most scenarios.

---

## Advanced Async/Await Scenarios
### Task 4: Multi-Layered Async Operations
**Goal:** Chain async methods with CPU-bound and I/O-bound work.

```csharp
using System;
using System.Net.Http;
using System.Threading.Tasks;

class Program
{
    static async Task Main()
    {
        string result = await MethodC();
        Console.WriteLine(result);
    }

    static async Task<string> MethodA()
    {
        await Task.Delay(1000); // Simulate CPU-bound work
        return "DataFromA";
    }

    static async Task<string> MethodB()
    {
        string data = await MethodA();
        using (HttpClient client = new HttpClient())
        {
            return await client.GetStringAsync($"https://example.com?data={data}");
        }
    }

    static async Task<string> MethodC()
    {
        string response = await MethodB();
        return $"Processed: {response}";
    }
}
```

**Key Things to Know:**
- Use `Task.Run()` to offload CPU-bound work to a thread pool thread.
- Avoid mixing async/await with `Task.Wait()` or `Task.Result` to prevent deadlocks.

---

## Fixing Deadlocks
### Task 5: Deadlock Prevention
**Problematic Code:**
```csharp
// Deadlock!  
var result = DeadlockMethod().Result;  
Console.WriteLine(result);  
```
**Solution:**
```csharp
static async Task Main()  
{  
    string result = await DeadlockMethod();  
    Console.WriteLine(result);  
}  

static async Task<string> DeadlockMethod()  
{  
    await Task.Delay(1000);  
    return "Hello, World!";  
}  
```

**Why It Works:**
- `async Main` allows `await` without blocking.
- Never use `.Result` or `.Wait()` in async code.

---

## Using ConfigureAwait(false)
### Task 6: Optimize Continuations
**Goal:** Avoid unnecessary context switching.

```csharp
using System;
using System.Threading;
using System.Threading.Tasks;

class Program
{
    static async Task Main()
    {
        await MethodB();
    }

    static async Task MethodA()
    {
        Console.WriteLine($"Before await: {Thread.CurrentThread.ManagedThreadId}");
        await Task.Delay(1000).ConfigureAwait(false);
        Console.WriteLine($"After await: {Thread.CurrentThread.ManagedThreadId}");
    }

    static async Task MethodB()
    {
        await MethodA();
        Console.WriteLine("MethodB completed.");
    }
}
```

**Key Things to Know:**
- Use `ConfigureAwait(false)` in library code to avoid capturing the synchronization context.
- Not needed in UI apps (e.g., WinForms/WPF) where context matters.

---

## Async Void vs. Async Task
### Task 7: Exception Handling Differences
**Danger of async void:**
```csharp
static async void VoidMethod()  
{  
    await Task.Delay(1000);  
    throw new Exception("This crashes the app!");  
}  

static async Task TaskMethod()  
{  
    await Task.Delay(1000);  
    throw new Exception("This can be caught!");  
}  

// Usage:  
try { VoidMethod(); } // Exception NOT caught!  
catch { }  

try { await TaskMethod(); } // Exception caught!  
catch { }    
```

**Key Things to Know:**
- Never use `async void` except for event handlers.
- Exceptions in `async Task` methods are wrapped in `Task.Exception`.

---

## Submission Guidelines
- **Code Quality (40%)**: Follow C# conventions, use clear comments.
- **Correct Implementation (40%)**: Avoid blocking calls, use `HttpClient` properly.
- **Git Repository (20%)**: Organize code into folders, include `.gitignore`.

---

## Key Takeaways
- **Async/Await**: Use for I/O-bound work.
- **TPL**: Use `Parallel.ForEach` for CPU-bound work.
- **Deadlocks**: Never block async code with `.Result` or `.Wait()`.
- **Thread Safety**: Use `lock` or thread-safe collections.

Happy Coding!

