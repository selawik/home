# Permission, Ownership, and Intra-Process Parallelism
Permission and ownership are two core components embedded in Selawik's type system. These annotations define various properties about the type that define its ownership and mutability semantics. The primary purpose of these annotations is to ensure the safety of intra-process parallelism, described below.

Permission and ownership annotations are components of the type they are defined alongside. It can be helpful to think of the type system as a two-dimensional lattice, with one permission being the object's type and the other being its permission/ownership, demonstrating that any object can be converted to the type `readonly Object`.

![Permission lattice](https://i.imgur.com/s3pflFn.png)


*A variant of this system has been [formally verified](https://www.microsoft.com/en-us/research/publication/uniqueness-and-reference-immutability-for-safe-parallelism).*

## Permission
A permission defines what kind of actions one can perform on an object reference:
- `mutable`: No immutability; the object can be mutated as usual.
- `readonly`: Indicates that the referrer cannot mutate the referent object.
- `immutable`: Guarantees that the target object graph will never be mutated.
- `init`: See [leaky constructors](#leaky-constructors).

`mutable` and `immutable` have a subtyping relationship with `readonly`. An object graph with either of these permissions can be implicitly converted to `readonly`. Following the principle of least authority, objects are declared as `readonly` by default (excluding primitives, which are `immutable`). Permissions are 'deep,' and apply to their entire object graph (unlike a `readonly` field in C#).

```cs
class Point
{
    public Single X { get; set; }
    public Single Y { get; set; }
}


mutable var m = new Point(3, 3);

immutable var i = new Point(3, 4); // can never be mutated
i.Y--; // error: cannot mutate immutable object graph.

var r = m; // ok - can convert mutable to readonly
r.X++; // error: cannot mutate readonly reference.
```

Permissions can appear in method signatures to indicate how `this` is closed over. Callers would need sufficient permission applied to their referent object in order to invoke those methods.

```cs
class List<T>
{
    public void Add(T o) mutable { ... }
    public UInt32 IndexOf(T o) { ... }
    public T this[UInt32 index] { get; set; }
}

var points = new List<Point>();
var x = points[0]; // allowed, since the getter is readonly
points.Add(new Point(3, 4)) // error: cannot mutate readonly reference.
```

Permissions can also be applied to delegates to indicate the permission required to capture state.
```cs
// any lambda expression that can be converted to a `Pure<T>` must only capture immutable state.
public delegate T Pure<T>() immutable; 
```

Permission is a component of the type. Effectively, every type declared has a mutable, immutable, and readonly version. This means that, for example, a generic type argument can be annotated with a permission.
```cs
List<Point> rr;                     // a readonly list of readonly Points.
List<mutable Point> mm;             // a readonly list of mutable Points.
immutable List<mutable Point> im;   // an immutable list of mutable Points.
immutable List<immutable Point> ii; // an immutable list of immutable Points.
// etc...
```

When necessary, generic parameters can be used to parametrize over permissions (rather than types). For example:
```cs
public delegate void Permissions<T1, T2, permission P>(P T1 one, P T2 two);

// expands to: Permissions<mutable Point, mutable List<Point>>
Permissions<Point, List<Point>, mutable> func = ...
```

Types can be declared `immutable` to denote that all instances of the type are immutable. Record types can be declared `mutable` to add mutating operators, like property setters.
```cs
immutable class Exception { ... }
mutable class Point(Int32 X, Int32 Y);
```

## Ownership
Ownership annotations mark aliasing properties about the given object graph:
- `isolated`: Denotes that an object has one mutable 'in-reference' to the root object, and no other references to other objects outside of the object graph (except for immutable references).

An `isolated` object can be mutated in-place.
```cs
isolated List<Point> builder = new List<Point>();

foreach (var num in 1..100)
{
    builder.Add(new Point(num, num));
}
```

Once can `consume` an `isolated` object to change its one mutable 'in-reference' to another reference, optionally preserving the `isolated` nature of the object.
```cs
isolated List<Point> moreBuilder = consume(builder);
builder.Add(new Point(1, 1)) // error: builder is no longer accessible to this mutator

// can use consume to remove the isolation
List<Point> built = consume(moreBuilder);
```

Immutability of objects is guaranteed using `isolated`. Isolation provides a way to prove that an object can only be mutated from one alias. This way, it can be proved that that a particular object does not have any other (possibly `mutable`) references, which could break the immutability guarantee.
```cs
immutable List<Point> frozen = consume(moreBuilder);
```

Using ownership analysis, some objects can be proven implicitly isolated. An expression with only isolated/immutable inputs (in this case, `new[]`) that evaluates to a mutable object is implicitly is implicitly isolated. (Similarly, any such expression that evaluates to a readonly object is implicitly immutable.) 
```cs
immutable List<Point> frozen = new List<Point>(new[] { ... });
```

As with permission, `isolated` can be applied to delegates, allowing only `isolated` (or `immutable`) state to be captured. Using isolated marks any instance of the delegate as `isolated`. 
```cs
public async delegate T TaskBody<T>() isolated;
```

## Safety
Proving the safety of these techniques requires that all side effects are exposed into the type system. Thus, some patterns that could bypass these annotations must be disallowed.

### Ambient Authority
Eliminating ambient authority, while important for security reasons, is also important to maintain the immutability guarantees in the system. For example, using the `Pure` delegate from above:
```cs
static Int32 x = 42;

Pure<Int32> func = () => x++;
```

Assuming static variables could be mutated, even though the function is supposedly pure, it still causes side effects. No state was technically captured, so this function is valid. Creating a way to expose ambient side effects in the type system would be somewhat clumsy — and those annotations already exist conceptually (the permissions described above). Additionally, this doesn't fit with the overall message of the system — eliminating ambient authority — either. 

Thus, all statics must be constant. To replace static state, capability objects will be used instead. This way, side effects can be modeled using the conventional methods, plus the security that comes with capabilities. Creating objects at compile time can open up many novel optimizations as well, like sharing objects across all processes.

### Leaky Constructors
A leaky constructor shares `this` before construction is finished. This is dangerous because:
- Other consumers can receive objects, not knowing that they are not finished initializing. They can assume invariants of such objects that are, in reality, suspect.
- A constructor can pass a mutable reference of itself, even if a supposedly immutable object is being constructed, meaning that immutability guarantees can be broken.

The solution to this problem is the `init` permission. The `init` permission means that the target object is undergoing initialization. It cannot be converted to the 'top' permission, `readonly`. Constructors will have the `init` permission applied to them, so that `this` can never be passed to places where it is not expected. Since instance methods will not be able to be called, due to the permission mismatch, the `init` permission can be (carefully!) applied to methods, so that the methods do not leak `this` either.

### Data Races
In the scope of an `async` operation, all captured state must be `isolated` to prevent accidental data races.
```cs
async void M(mutable Int32[] p) 
{
    int n = p[0];
    await ...
    assert(n == p[0]);
}

mutable int[] nums = ...
Async<void> op = async M(nums); // error: cannot capture shared, mutable state
nums[0] = 2;
```

To solve this, one would either need to mark `nums` as `immutable` or `isolated`.
```cs
isolated int[] nums = ...
Async<void> op = async M(consume(nums));
nums[0] = 2; // error: nums is no longer accessible to this mutator
```

## Application
These patterns allow interesting patterns without applying any runtime overhead. For example, many concurrent activities can share `readonly` access to the same object (like a readers/writer lock).

These type annotations allow for safe, fine-grained, intra-process parallelism. These kinds of parallel operations come in two forms — *structured* and *unstructred*.

### Structured Parallelism
Operations like forks and data queries can have a finite and predictable lifetime. Thus, it is said that these operations are structured.

Structured parallel operations are allowed to capture `readonly` state, since the mutator is guaranteed to be paused. 
```cs
Int32[] arr = ...;
Int32[] results = await Async.Fork(
    () => await arr.Reduce((x, y) => x + y),
    () => await arr.Reduce((x, y) => x * y));
```

By merely reading from the array, the sum and product of this array are computed with data race freedom. This is guaranteed by the permission requirements of the `Fork` function:
```cs
public static async T[] Fork<T>(params ForkBody<T>[] funcs);
public async delegate T ForkBody<T>() readonly;
```

Because no side-effectful operations are allowed, optimizations like auto-parallelization can be introduced to operations like data queries.

### Unstructured Parallelism
Unlike forks, tasks do not have a structured lifetime, so not even `readonly` state can be captured. However, if a task needs to party on mutable state, it can capture `isolated` (or `immutable`) state, so that it can be linearly handed off to a task with guarantee that there are no auxiliary mutators.
```cs
public static Task<T> Run<T>(TaskBody<T> func);
public async delegate T TaskBody<T>() immutable isolated;
```

This example makes use of lambda capture lists to make consuming the isolated state more straightforward.
```cs
isolated Int32[] data = ...;
Task<Int32> t = Task.Run([consume(data)]() => {
    // Here here, we own `data`.
});
```

Because tasks are isolated,  `mutable` objects produced by them could become `isolated`, and likewise `readonly` objects becoming `immutable`, which is important from a compositional standpoint.


## Future Prospects
While these existing type system annotations will ensure the safety of intra-process parallelism, there are still opportunities for message passing races/TOCTOU bugs. 
```cs
async bool IsRed(AsyncColor c) {
    return (await c.R > 0 && await c.G == 0 && await c.B == 0);
}

AsyncColor c = ...;
await IsRed(c);
assert(await c.R > 0);
```

Assuming `c` is a mutable object, it is definitely possible for the assertion to fire, as the values of the object might change due to various concurrent activities interleaving with each other. A possible solution to this issue is to apply a form of the permission system to asynchronous activities themselves. Combined with a readers/writer-lock type idea, permissions affixed to async interfaces can ensure that state and operations are dispatched safely.

It may be interesting to invest in designing subsets of this system on GPUs. Side effects and memory ownership are very important concepts in GPU programming, so this model may help in making the programming model for that purpose more elegant and unified.



