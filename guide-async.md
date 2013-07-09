---
layout: default
---

# Dummy's guide to Async

Async is a library for asynchronous programming, i.e. programming
where some part of the program must wait for things that happen at
times determined by some external entity (e.g. a human or another
program).  This includes pretty much any program that uses blocking
calls (e.g. networking code, disk access), timeouts, or event loops
(e.g. GUIs).

In a nutshell, the idea is to use non-preemptive user-level threads
and first-class blocking operations with blocking expressed in the
type system.  The benefits of the approach are:

- Non-preemptive threading is simpler to reason about than pre-emptive
  threading.  For the bulk of one's program, one doesn't have to think
  about race conditions, mutexes, etc.  One can also use existing
  non-thread-safe code without worries.
- User level threads are much cheaper than threads provided by the
  operating system, both in creation and context switching.
- We control the scheduler for our threads, which makes it possible
  for us to do a better job for the application at hand.
- Writing low latency applications is easier -- it's harder to have an
  application block indefinitely by mistake.

In this programming model, well-behaved functions never block -- or
rather blocking is explicitly expressed in the type of a function.
Instead of a blocking function

<pre class="sh_caml">
val f : unit -> 'a
</pre>

we would have

<pre class="sh_caml">
val f : unit -> 'a Deferred.t
</pre>

to express that "f" can block.  One can not use a deferred directly
(since to do so would block).  One must schedule a handler to run upon
completion of the blocking call.

<pre class="sh_caml">
upon (f ()) (fun v -> ... (* do something with the result of f *) ...)
</pre>

You can get a pretty good introduction to Async by reading the mli's.
Start by looking at `async/lib/std.ml`, which has comments that should
guide you through reading the rest of it.

It's worth noting that it is possible to call a blocking function
inside of an Async program, but you should avoid it.  What will happen
if you do that is that that single blocking operation will stop
everything else in the Async world from happening.  There are some
things that are done in Async to make it harder to call an ordinary
blocking function accidentally, but on some level, one just has to be
careful.

## `Deferred.t`

As discussed above, a deferred represents a value that may not be
ready yet to use.  We described the operation `upon`, which allows you
to schedule a closure to be run as soon as the deferred is determined.
Deferreds also form a monad, so you have the usual bind (`>>=`), map
(`>>|`) and return operators.  (Don't worry if you don't know what a
monad is.)  The types of those operators are:

<pre class="sh_caml">
(* return *)
val return : 'a -> 'a Deferred.t

(* map *)
val (>>|) : 'a Deferred.t -> ('a -> 'b) -> 'b Deferred.t

(* bind *)
val (>>=) : 'a Deferred.t -> ('a -> 'b Deferred.t)-> 'b Deferred.t

(* upon *)
val (>>>) : 'a Deferred.t -> ('a -> unit) -> unit
</pre>

`return` simply creates a deferred from a value, where the newly
created deferred is immediately available.  `map` is analogous to the
usual map function, and it can be used to do a non-blocking
transformation to a blocking value.  Thus, if you want a function that
reads a line and then uppercases it, you could write:

<pre class="sh_caml">
let rlu (r : Reader.t) : string option Deferred.t =
  Reader.read_line r
  >>| function
    | `Ok x -> Some (String.uppercase x)
    | `Eof -> None
;;
</pre>

this is using the Reader module which is Async's library for reading
from files and sockets.

`bind` is used for the case where you want to take a deferred value
and then, when it is determined, do another blocking operation.  Thus,
if you want to open a file and then read the first line, you might do
something like:

<pre class="sh_caml">
let read_first_line (f : string) : string option Deferred.t =
  Reader.open_file f 
  >>= (fun r -> Reader.read_line r)
  >>| function
    | `Ok x -> Some x
    | `Eof -> None
</pre>

And you can then do longer sequences of this if you want to do more
transformations, like this function that reads in the first two lines
of the given file and concatenate them:

<pre class="sh_caml">
let read_and_concat f : string -> string Deferred.t =
  Reader.open_file f >>= fun r ->
  Reader.read_line r >>= fun line1 ->
  Reader.read_line r >>= fun line2 ->
  return (line1 ^ line2)
</pre>

Note the use of return so that the last function actually returns a
deferred, even though it's just computing that value.  We could as
well have written:

<pre class="sh_caml">
let read_and_concat f : string -> string Deferred.t =
  Reader.open_file f >>= fun r ->
  Reader.read_line r >>= fun line1 ->
  Reader.read_line r >>| fun line2 ->
  line1 ^ line2
</pre>

## Error Handling

A monitor is a context that determines what to do when there is an
unhandled exception.  Every Async computation runs within the context
of some monitor, which, when the computation is running, is referred
to as the "current" monitor.  Monitors are arranged in a tree -- when
a new monitor is created, it is a child of the current monitor.

The simplest way to handle errors is to use
<code>Monitor.try_with</code>.  Suppose that we would like to write
the client of a server over a Tcp connection. The server expects us to
send the <code>"hello"</code> string, upon which it will feed us with
a stream of data that we will return to the caller. If the connection
fails, we will use cached information from a local file:

<pre class="sh_caml">
let connect_or_use_cached ~cache host_and_port =
  let connect () =
    let host = Host_and_port.host host_and_port in
    let port = Host_and_port.port host_and_port in
    Tcp.connect (Tcp.to_host_and_port host port)
  in
  Monitor.try_with connect >>= function
  | Ok (reader, writer) ->
    Writer.write writer "hello";
    Writer.close writer >>= fun () ->
    Reader.contents reader
  | Error _exn ->
    read_and_concat cache
</pre>

Another possible behavior is for the function to try to reconnect to
the server. The following recursive functions has the desired
behavior, but is not tail recursive:

<pre class="sh_caml">
let connect ~maximum_delay host_and_port =
  let connect () =
    let host = Host_and_port.host host_and_port in
    let port = Host_and_port.port host_and_port in
    Tcp.connect (Tcp.to_host_and_port host port)
  in
  let rec retry ~delay =
    Monitor.try_with connect >>= function
    | Error _exn ->
      let delay = Time.Span.min maximum_delay delay in
      Clock.after delay >>= fun () ->
      retry ~delay:(Time.Span.add delay delay)
    | Ok (reader, writer) ->
      Writer.write writer "hello";
      Writer.close writer >>= fun () ->
      Reader.contents reader
  in
  retry ~delay:(Time.Span.of_sec 1.)
</pre>

We will learn below how to create a tail recursive function. We leave
you as an exercise to modify the previous function to its tail
recursive form.

## Avoiding Space Leaks

There are a few easy ways to create space leaks in Async that you
should know about.

### Streams

**Note**: Streams are now largely deprecated in Async.  Use the `Pipe`
module instead!

A `Stream.t` is a functional representation of a stream of future
events.  A stream is functional in the sense that its meaning doesn't
change over time: the stream always gives you access to the whole
stream of events, starting at the same point no matter how many
further events occur.

That of course means that if you hold on to a stream that is getting
filled up, then you are implicitly holding on to every event placed on
the stream.  This is usually a mistake, since it means that nothing
put on the stream can ever be collected.  A very common idiom is to
construct a stream and iterate over it immediately, never storing the
stream directly anywhere, as in:

<pre class="sh_caml">
open Async.Std
open Async_print

let () =
  Stream.iter (Clock.at_intervals (Time.Span.of_sec 5.))
      ~f:(fun () -> printf "tick!\n%!");
  Scheduler.go ()
</pre>

Here, the stream generated by the call to `Clock.at_intervals` will be
collected almost immediately, and so is safe.  Here's an example that
would involve a space leak:

<pre class="sh_caml">
open Async.Std
open Async_print

type state = { clock_stream: unit Stream.t;
               outc: out_channel }

let () =
  let state = { clock_stream = Clock.at_intervals (Time.Span.of_sec 5.);
                outc = stdout; }
  in
  Stream.iter state.clock_stream
     ~f:(fun () -> fprintf state.outc "tick!\n%!");
  Scheduler.go ()
</pre>

### `Stream.filter_map`

At the moment, a `Stream.filter_map` is never garbage collected as
long as the underlying stream is still being updated.  So if you have
code which repeatedly calls Stream.filter_map on a stream, uses it for
a while and then throws away its reference to that stream, then that
code has a leak.

This is more or less a bug in `Stream.filter_map`.  Steven knows how
to fix it, and it will eventually get resolved.

### Stream consumers do not push back to stream producers

Code that extends a stream does not automatically block when the
consumer of the stream is not able to keep up.  So, you need to make
sure that your consumer can keep up, or that there is some out-of-band
pushback mechanism by which the consumer causes the producer to slow
down if it cannot keep up.

### Writer buffers can grow unboundedly large

Writers have a buffer that holds the bytes that have been written but
not yet sent to the OS.  When you call [Writer.write] you add bytes to
that buffer, and possibly grow it.  Periodically, a background
micro-thread in the writer makes a system call to send bytes to the
OS.  There is no guarantee that the data sync on the other side of the
writer can keep up with the rate at which you are writing.  If it
cannot, the OS buffer will fill up and the writer micro-thread will be
unable to send any bytes.  In that case, calls to [Writer.write] will
grow the buffer without bound, as long as your program produces data.

One solution to this problem is to check `Writer.bytes_to_write` and
not produce any more data if that is beyond some bound.
