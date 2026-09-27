# Go Day 0 — Baseline Assessment

- **Date:** 2026-09-27
- **Type:** Diagnostic assessment, not a teaching session
- **Purpose:** Establish a baseline in Go semantics, concurrency, runtime understanding, and production-code reasoning. The track assumes professional Go experience; it is intended to deepen conceptual and engineering judgment, not teach syntax from scratch.

## Summary diagnosis

Approximate assessment: **C+ / B−**. The user has working professional familiarity with Go, but uneven conceptual depth. The central finding is:

> I am currently stronger at recognizing and using Go constructs than at reasoning about their underlying semantics.

This is a baseline, not a verdict on overall professional ability. The main conceptual gaps were context, runtime/memory, synchronization, interfaces, and advanced concurrency. Answers, corrections, and uncertainties below are preserved as they occurred; the assessment was not made more favorable in retrospect.

## Assessment scope

The diagnostic covered arrays and slices, backing arrays and `append`, maps, interfaces, pointers, method receivers, errors and `%w`, `defer`, `panic`, package/API design, goroutines, data races, channels, deadlocks, `select`, scheduling, escape analysis, context, production HTTP code review, and worker pools.

## Topic-by-topic results

### Arrays, slices, and `append`

**Answer:** Initially described a slice as “the pointer to this list” and contrasted arrays as passed by value with slices “by reference.” Initially predicted that changing an element through a copied slice would leave the original unchanged, then corrected the prediction during the answer: both slices show the changed element because they share backing storage.

For `append`, initially expected a separate array. That was incorrect as a general rule.

**Correct mental model:** A slice is a descriptor with a pointer to a backing array, a length, and a capacity. Copying a slice copies that descriptor. Element writes through either descriptor affect shared backing storage. `append` reuses that storage if capacity allows and allocates a new backing array when capacity is insufficient.

```go
a := make([]int, 3, 4)
copy(a, []int{1, 2, 3})
b := a
b[0] = 100 // a[0] is now 100 too

c := append(b, 4) // capacity permits reuse here
```

With insufficient capacity, append has to use different storage:

```go
short := make([]int, 1, 1)
long := append(short, 2) // needs capacity 2; uses a new backing array
```

When `append` reallocates, the returned slice refers to new storage. Whether later element writes are shared depends on whether the slices still share a backing array; capacity matters.

**Finding:** Basic slice familiarity exists, but backing-array sharing and append/capacity effects need deliberate study.

### Maps

**Answer:** Correctly said that a missing-key lookup returns the map value type's zero value and identified comma-ok:

```go
x, ok := m["bar"]
```

Also asked why examples use `foo` and `bar`; these are conventional placeholder names, not Go features.

**Finding:** Map lookup semantics are a strength.

### Interfaces

**Answer:** Described an interface as roughly equivalent to `any`. This was a significant misconception.

**Correct mental model:** `any` is an alias for `interface{}`. A non-empty interface specifies methods; types satisfy it implicitly by implementing those methods. An interface value conceptually carries a dynamic type and dynamic value. An interface containing a typed nil pointer is not itself nil:

```go
type Animal interface{ Speak() }
type Dog struct{}
func (Dog) Speak() {}

var d *Dog = nil
var a Animal = d
fmt.Println(a == nil) // false: dynamic type is *Dog
```

**Finding:** Interface meaning, method satisfaction, and typed nil behavior are major learning priorities.

### Pointers and method receivers

**Answer:** Understood that pointers can let a function mutate a caller-visible value. This was directionally correct but incomplete. The user understood that pointer receivers can mutate the receiver and value receivers operate on a copy, but was unsure and could not give a concrete bug example.

**Mental model to develop:** Go passes values; passing a pointer copies the pointer value, which still refers to the same object. Reason about addressability, copying, mutation, and whether a pointer is appropriate. Receiver choices also affect copying, method sets, and interface satisfaction—not only mutation.

**Finding:** Partial understanding; revisit value semantics, pointer semantics, and receiver selection.

### Errors and `%w`

**Answer:** Correctly noticed that returning a new generic error discards useful underlying error information. Did not know `%w` and guessed it was a formatting form.

**Correct mental model:** `%w` wraps an error so callers can inspect the chain with `errors.Is` and `errors.As`:

```go
return fmt.Errorf("database lookup failed: %w", err)
```

**Finding:** Error preservation was recognized; wrapping and error inspection need study.

### `defer`

**Answer:** Correctly predicted that `defer fmt.Println(x)` prints the value captured when the defer statement runs, while a deferred closure reads the later value of the captured variable. The explanation was imperfect.

**Correct mental model:** Arguments to a deferred function call are evaluated when the `defer` executes. The deferred function runs later; a closure can observe a captured variable's later value.

```go
func directCall() {
	x := 10
	defer fmt.Println(x) // prints 10
	x = 20
}

func closureCall() {
	x := 10
	defer func() { fmt.Println(x) }() // prints 20
	x = 20
}
```

**Finding:** Working knowledge with a semantic gap.

### `panic`

**Answer:** Said ideally never, except in unusual situations.

**Nuance:** `panic` is not ordinary error handling, but can be appropriate for unrecoverable programmer errors or violated invariants.

**Finding:** Partial understanding.

### Package and API design

**Answer:** Focused on the insecurity of storing plaintext passwords and suggested a map such as `map[int]UserInfo`. Did not identify encapsulation and exposure of internal state as the main architectural concern. Also suggested making `user_id` a global constant in the later HTTP review.

**What to learn:** Package ownership of invariants; behavior-oriented APIs; encapsulation; domain models versus persistence models and DTOs; keeping password hashes and internal fields out of inappropriate structures. A `map[int]User` only gives unique keys inside that map; it does not solve domain-level duplicate-user rules.

**Finding:** API boundaries and data modeling need deliberate practice.

### Goroutines, races, and synchronization

**Goroutine lifecycle — answer:** Correctly understood that `go` starts concurrent work and that the caller does not automatically wait. Suggested `sync.WaitGroup`. Incorrectly attributed goroutine termination to garbage collection.

**Correction:** When `main` returns, the process exits. The garbage collector does not terminate goroutines because a function returned. A `WaitGroup` can coordinate when goroutines have finished.

**Data race — answer:** Recognized that concurrent `counter++` is unsafe, but first proposed a `WaitGroup` alone.

**Correction:** A `WaitGroup` answers whether goroutines have completed; it does not protect concurrent memory access. Lifecycle synchronization and memory synchronization are different concerns. Protect shared state with a mutex, `sync/atomic` where suitable, or channel-based ownership/serialization.

```go
var counter int
var wg sync.WaitGroup
var mu sync.Mutex

wg.Add(1)
go func() {
	defer wg.Done()
	mu.Lock()
	counter++
	mu.Unlock()
}()
wg.Wait() // waits; the mutex protects counter
```

**Finding:** Concurrency lifecycle and memory-safety models need reinforcement; using a `WaitGroup` does not make code race-free.

### Channels, deadlock, and `select`

**Channels — answer:** Demonstrated reasonable understanding of unbuffered sends requiring a receiver and buffered channels queuing values up to capacity.

**Deadlock — answer:** Correctly identified that sending to an unbuffered channel in the same goroutine before receiving blocks before execution reaches the receive. Correctly suggested a buffer of capacity one or another goroutine for the send.

**`select` — answer:** Compared it to a switch over channel operations and understood cancellation/timeout uses. This was directionally good, but imprecise about synchronization.

**Correct core:** `select` waits until one or more listed channel operations can proceed, then executes one ready case. Channel synchronization comes from the communication operation itself.

**Finding:** Channels and deadlock basics are relative strengths; refine `select` semantics.

### Scheduler and goroutines versus OS threads

**Answer:** One of the stronger responses. Described goroutines as a runtime abstraction over OS threads and recognized Go's M:N scheduling model: many goroutines can be multiplexed over fewer OS threads. The Python comparison was broad and should not be treated as a strong claim.

**Finding:** High-level scheduler knowledge is relatively strong; runtime details remain to be learned.

### Escape analysis

**Answer:** Explicitly did not know the terminology. For a function returning `&u`, guessed that `u` would somehow be stored where the caller needs it.

**Correct mental model:** The compiler performs escape analysis to ensure a value remains valid for its uses. It can arrange for an escaping value to live on the heap; programmers do not manually choose stack versus heap. Allocation is an implementation decision, and source shape alone should not be used to claim a guaranteed allocation.

```go
func newUser() *User {
	u := User{}
	return &u
}
```

The compiler analyzes whether `u` must outlive the call; the programmer reasons about lifetime and ownership, not by manually assigning a variable to a particular memory region.

**Finding:** Clear runtime/memory gap.

### Context

**Answer:** The largest conceptual weakness. Described `context.Context` in terms of static information, connections, memory/heap storage, and reusing connections.

**Correct mental model:** Contexts propagate cancellation, deadlines, and request-scoped metadata through a call tree. They are not connection pools, caches, general state containers, or substitutes for normal function parameters.

```go
func query(ctx context.Context, db *sql.DB) error {
	_, err := db.ExecContext(ctx, "UPDATE jobs SET started = TRUE")
	return err // cancellation/deadline can stop the operation
}
```

For work that does not accept a context-aware API, code can observe `ctx.Done()` and stop cooperatively. HTTP request context should flow to downstream work such as database, Redis, and HTTP calls.

**Finding:** Major gap; explicitly rated D in the assessment.

### Production HTTP code review

**Answer:** Identified potentially useful concerns around authentication/authorization, generic error handling, and HTTP status distinctions. The review was incomplete and included the incorrect suggestion to make `user_id` a global constant.

**Review skills to build:** Request validation; authentication and authorization; status semantics; context propagation and timeouts; safe database interaction; logging; avoiding sensitive implementation details in responses; JSON encoding errors and content type; observability; expensive work; centralized error mapping; and graceful cancellation.

**Finding:** Production code-review reasoning needs systematic practice.

### Worker pool

**Answer:** Explicitly did not know how to implement one and was unsure about context cancellation, worker pools, goroutine leaks, error propagation, and channel coordination.

**Finding:** Major conceptual concurrency gap, not a syntax failure. This should be taught incrementally after core synchronization and cancellation.

## Approximate topic ratings

| Area | Rating |
| --- | --- |
| Basic Go syntax | B |
| Arrays / slices | B− |
| Maps | B |
| Structs / pointers | B− |
| Methods / receivers | B− |
| Interfaces | C |
| Error handling | C+ |
| `defer` | B− |
| Package / API design | C |
| Goroutines | C+ |
| Channels | B |
| `select` | B− |
| Synchronization | C |
| Context | D |
| Runtime / memory | D |
| HTTP / backend Go | C |
| Production code review | C |
| Advanced concurrency | D |

Overall: **C+ / B−**.

## Strengths

- Professional Go familiarity and basic syntax are already present.
- Correct map zero-value and comma-ok understanding.
- Correctly identified the basic unbuffered-channel deadlock.
- Reasonable channel buffering intuition.
- High-level M:N scheduler description was strong.
- Correct output predictions for both `defer` examples.
- Recognized some HTTP concerns and the danger of throwing away errors.

## Weaknesses and misconceptions to revisit

- Slices are not simply references; descriptor copies can share backing arrays, and append behavior depends on capacity.
- An interface is not equivalent to `any`; typed nil interface values can be non-nil.
- Pointer/value and receiver reasoning is incomplete.
- `%w` and error-chain inspection were unknown.
- Goroutine lifetime is not controlled by garbage collection; a `WaitGroup` is not a race-prevention mechanism.
- Context was substantially misunderstood (rated D).
- Escape analysis and runtime memory decisions were unfamiliar.
- Package/API encapsulation, production HTTP review, and worker-pool coordination need development.

## Recommended learning progression

These phases are recommendations, not completed work.

1. **Go semantics:** slices/arrays/capacity; pointers and value semantics; structs/receivers; interfaces and typed nil; errors and wrapping; `defer`, `panic`, and `recover`.
2. **Concurrency:** goroutine lifecycle; races; mutexes and `WaitGroup`; channels and `select`; cancellation; worker pools; graceful shutdown.
3. **Context and backend Go:** context propagation; HTTP servers and middleware; request lifecycle; database interactions; timeouts/retries; graceful shutdown.
4. **Runtime:** stack/heap concepts; escape analysis; GC; scheduler; allocations; profiling with `pprof`.
5. **Production engineering:** tests, benchmarks, race detector, fuzzing, package/API design, observability, performance, and debugging.

## Completion and next task

**Day 0 status:** Diagnostic complete and recorded. No later phase is complete by this record.

**Next task — Go Day 1:** Investigate slice descriptors, shared backing arrays, length versus capacity, and `append` reusing or replacing backing storage. Begin with predictions and small experiments before explanation; preserve an attempt-first learning format.
