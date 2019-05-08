# Error Handling 
Error handling in the Selawik project is an important component in making sure applications and services are reliable. The system is two-pronged, having seperate mechanisms for detecting programmer errors and also dynamic failure. 

## Abandonment
Abandonment is Selawik's response to programmer bugs. Known elsewhere as fail-fast, abandonment tears down the entire process, refusing to run any user code while doing so. Since processes in Selawik are lightweight, small, and completely isolated, tearing down a whole process in response to a bug is okay and promotes reliable design.

Abandonment can occur after any of the following operations:
- Invalid cast.
- Array access out-of-bounds.
- Divide by zero.
- Stack overflow or out-of-memory.
- Arithmetic overflow or underflow.
- Contract or assertion failures.
- Explicitly abandoning.

While it makes sense to abandon when the operations above occur unintentionally, sometimes it is reasonable to want to explicitly recover from these. Many of these APIs will have 'try' versions that propagate errors via [dataflow](#resultt). It will also be possible to `try` to allocate an object, which would return `null` instead of abandoning on failure.
```cs
Int32[]? buffer = try new Int32[1_000_000];
```

## Design By Contract
Contracts are Selawik's main construct for preventing programmer errors. They define the 'contract' for a method or type - conditions that must hold true in order for execution of the program to continue. If a contract is found to be false at runtime, abandonment is triggered. The compiler is also free to issue compile time errors or elide runtime checks if it can prove such facts about the contracts at compile time. 

Using contracts leads to much more concise and obvious error handling code than, for example, exceptions. Take this sample from the `FileStream.Read` method in corefx:
```cs
public override Task<Int32> ReadAsync(Byte[] buffer, Int32 offset, Int32 count, CancellationToken cancellationToken)
{
    if (buffer == null)
        throw new ArgumentNullException(nameof(buffer), SR.ArgumentNull_Buffer);
    if (offset < 0)
        throw new ArgumentOutOfRangeException(nameof(offset), SR.ArgumentOutOfRange_NeedNonNegNum);
    if (count < 0)
        throw new ArgumentOutOfRangeException(nameof(count), SR.ArgumentOutOfRange_NeedNonNegNum);
    if (buffer.Length - offset < count)
        throw new ArgumentException(SR.Argument_InvalidOffLen /*, no good single parameter name to pass*/);

    // ...
}
```

Using contracts, we can shorten this down to: 
```cs
public override Task<Int32> ReadAsync(Byte[] buffer, Int32 offset, Int32 count, CancellationToken cancellationToken)
    requires buffer is not null
    requires offset >= 0
    requires count >= 0
    requires buffer.Length - offset < count
{ /* ... */ }
```
> NOTE: Using some more other features of Selawik, this can be simplified to
> ```cs
> public override Task<int> ReadAsync(Byte[] buffer, CancellationToken cancellationToken)
> { /* ... */ }
> ```
> See (TODO) slices and non-nullable types.


There are three kinds of contract definitions:
- `requires`: Defines a precondition, or a condition that must be true before the the method begins executing.
- `ensures`: Defines a postcondition, or a condition that must be true after the method finishes executing.
- `invariant`: Defines an invariant, or a condition that must be true before *every* method starts executing.

#### Preconditions and Postconditions

Preconditions and postconditions are defined per method.
```cs
public void Clear()
    ensures Count == 0
{ /* ... */ }
```

In postconditions, using the `return` keyword retrieves the value returned from the method.
```cs
public Int32 Increment(Int32 i)
    ensures return == i + 1
{ /* ... */ }
```

Using `old(...)`, postconditions can capture the state of variables before the method begins executing.
```cs
public void Register(String name)
    requires !String.IsNullOrEmpty(name);
    ensures registry.Count - old(registry).Count == 1
{ /* ... */ }
```

#### Invariants
Invariants are defined per type.
```cs
class MyDisposable
{
    private Boolean _disposed;
    private invariant _disposed == false;
}
```
The invariant is checked before execution of every applicable method. It is comparable to a precondition applied to each method. The visibility of the invariant denotes which methods will be checked. Any method with visibility equal to or higher than the stated visibility of the invariant will be checked. 

This `private` invariant will be checked on every method invoked within `MyDisposable`. A `public` invariant will only check on invocations of `public` methods, allowing invariants to sometimes be temporarily violated in private code.

#### Assertions
An assertion is a manual check in the code for a condition, abandoning the process if it is false. 
```cs
assert(myList is not null);
```
Although an assertion is not a kind of contract, many ideas are shared between the two. Like contracts, all assertions will always be checked, and will be used liberally across the entire codebase. The compiler will also be able to aggressively eliminate or issue compile time errors about assertions.

## Typed Exceptions
Contracts are a useful tool for preventing programmer-induced bugs. However, in some situations — for example, I/O and text parsing — errors are not the programmer's fault. For these situations, we use 'typed exceptions.' Like contracts, these are also integrated into the type system, but serve a different purpose.

Any method that throws an exception must be marked with the `throws` effect.

```cs
public async String DownloadText(Uri uri) throws
{ /* ... */ }
```

At the callsite of a `throws` method, you must use the `try` keyword. Note that the keyword has no effect except making it more clear which callsites can raise exceptions.

```cs
var text = try client.DownloadText("example.com");
```
Since this snippet does not handle exceptions from `DownloadText`, the enclosing method would have to be marked `throws`. Of course, if all exceptions are handled, there is no need to apply the `throws` effect.
```cs
try
{
    var text = try client.DownloadText("example.com");
}
catch (ex)
{
    Log(ex);
    // recover somehow
}
```

Using a singular `throws` represents a single failure mode — that is, no specific typing information is applied to the exception (since it isn't necessary). For most APIs, this single failure mode is enough and is understood by the consumer. However, when multiple failure modes are necessary, it is possible to specify multiple exception types:

```cs
public void Predict(Model m) throws PredictionFailedException, AbortException 
{ /* ... */ }
```
Typed exceptions integrate well with the rules of the Selawik platform. Standard subtyping rules apply; for example, a method that overrides a `throws` method does not itself have to be `throws`. In addition, collapsing unneeded information (or adding more specificity about) a method's failure mode(s) is easy. 
```cs
var myFunc = throws () => Predict(myModel); 
// specific exception information is lost, but we didn't need it for what we're doing anyway
```

These aspects, combined with the fact that typed exceptions will not be the dominant error handling construct, make the system much more palatable and reliable, compared to a system like Java's (or one without checking at all, like C#).

### `Result<T>`
On occasion, a more dataflow oriented solution is preferred to imperative-like `try` and `catch` blocks. This syntax makes use of the `Result<T>` union type and the `try ... else` expression.
```cs
Result<String> result = try client.DownloadText("example.com") else catch;
return result switch
{
    String s => s,
    Exception e => e.Message
}
```

One is free to also replace the `catch` with any expression, making the feature work similarly to null coalescing in C#.
```cs
return try client.DownloadText("example.com") else "Error";
```

### Aborts
The abort pattern makes use of exceptions and unforgeable tokens to make it easy to exit a workflow (without tearing everything down like abandonment would). 

Here is the definition of `AbortException`:
```cs
public immutable class AbortException : Exception 
{
    public AbortException(immutable object token);
    public void Reset(immutable object token);
    // etc.
}
```
`AbortException` is special, in that it is re-raised at the end of any `catch` block unless it has been reset (in other words, it is *undeniable*). The exception must be reset using the same token that was provided to it during construction — if it doesn't match, abandonment occurs.

Aborts are useful when a workflow's result can be invalidated midway through its process. For example, imagine the UI library is doing a layout recalculation. If this calculation is invalidated (e.g. the window is resized) before it has finished processing, it would be a waste of time to continue with the calculation. Thus, the UI library can utilize aborts to quickly move the context back to the top of the stack so that the calculation can start fresh.

```cs
class MyComponent
{
    const object token = new object();
    const AbortException abort = new AbortException(token);

    void Abort() throws AbortException => throw abort;

    void Loop()
    {
        while (true) 
        {
            try 
            {
                // here go the operations that may end up calling our Abort()
            }
            catch (AbortException ex)
            {
                ex.Reset(token);
            }
        }
    }
}
```

## Keepers
A keeper is an error handling pattern that makes use of capability objects to recover from errors 'in situ.' Rather than throwing an exception, code that makes use of keepers refers to an object given to them by the consumer to figure out what to do next. Here is an example definition of a keeper, perhaps used for some sort of network client:
```cs
public async interface NetworkKeeper
{
    async String BadRequest(Uri uri) throws;
    async Credentials Unauthorized(Credentials creds, Uri uri) throws;
    async String Forbidden(Uri uri) throws;
    async String RequestTimeout(Uri uri) throws;
    // etc.
}
```

On the consumer side, one would implement this interface using their error recovery method of choice:
```cs
public class AskForCredsKeeper : NetworkKeeper
{
    public async Credentials Unauthorized(Credentials creds, Uri uri)
    {
        return await RequestForNewCredentials(uri);
    }
    
    // more implementations...
}
```

When the network client gets returned a 401 Unauthorized status code, instead of raising a possibly costly exception, it can refer to the keeper that was given to it to figure out what to do next. In this case, it pops up a dialog that requests for credentials.

### See Also
> TODO:
> - Non-nullable types
> - Capabilities
> - Processes/async model
