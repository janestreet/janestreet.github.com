---
layout: default
---

# patdiff example

`patdiff` is a tool which displays differences between two files. It
uses the patience algorithm and display word-by-word diferrences when
appropriate instead of just line differences. The full documentation
is included in the distribution
([repository](https://github.com/janestreet/patdiff)). Following is an
example of output, produced by the command:

<pre class="sh_sh">
$ patdiff writer.ml~109.07.00 writer.ml~109.08.00
</pre>

The result:

<pre class="sh_sourceCode">
---writer.ml~109.07.00
+++writer.ml~109.08.00
<span style="color:#86cdea;font-weight:bold;">@@@@@@@@@@ </span><span style="color:#86cdea;font-weight:bold;">-915,29 +915,24</span><span style="color:#86cdea;font-weight:bold;"> @@@@@@@@@@</span>
  let stderr = lazy (snd (Lazy.force stdout_and_stderr))
  
  let apply_umask perm =
    let umask = Core_unix.umask 0 in
    ignore (Core_unix.umask umask);
    perm land (lnot umask)
  ;;
  
<span style="color:#fe8686;font-weight:bold;">-|</span>let with_file_atomic <span style="color:#fe8686;text-decoration:underline;font-weight:bold;">?temp_prefix</span> ?perm ?fsync:(do_fsync = false) file ~f =
<span style="color:#aece92;font-weight:bold;">+|</span>let with_file_atomic <span style="color:#aece92;">?temp_file</span> ?perm ?fsync:(do_fsync = false) file ~f =
    Async_sys.file_exists file
    &gt;&gt;= fun file_exists -&gt;
    (match file_exists with
    | `Yes -&gt; (Unix.stat file &gt;&gt;| fun stats -&gt; Some stats.Unix.Stats.perm)
    | `No | `Unknown -&gt; return None)
    &gt;&gt;= fun current_file_permissions -&gt;
<span style="color:#fe8686;font-weight:bold;">-|</span>  <span style="color:#fe8686;text-decoration:underline;font-weight:bold;">let prefixed_temp_file =</span>
<span style="color:#fe8686;font-weight:bold;">-|</span><span style="color:#fe8686;text-decoration:underline;font-weight:bold;">    match temp_prefix with</span>
<span style="color:#fe8686;font-weight:bold;">-|</span><span style="color:#fe8686;text-decoration:underline;font-weight:bold;">    | None -&gt; file</span>
<span style="color:#fe8686;font-weight:bold;">-|</span><span style="color:#fe8686;text-decoration:underline;font-weight:bold;">    | Some temp_prefix -&gt; temp_prefix ^ file</span>
<span style="color:#fe8686;font-weight:bold;">-|</span><span style="color:#fe8686;text-decoration:underline;font-weight:bold;">  in</span>
<span style="color:#fe8686;font-weight:bold;">-|</span><span style="color:#fe8686;text-decoration:underline;font-weight:bold;">  </span>Unix.mkstemp <span style="color:#fe8686;text-decoration:underline;font-weight:bold;">prefixed_temp_file</span>
<span style="color:#aece92;font-weight:bold;">+|</span>  Unix.mkstemp <span style="color:#aece92;">(Option.value temp_file ~default:file)</span>
    &gt;&gt;= fun (temp_file, fd) -&gt;
    let t = create fd in
    with_close t (fun () -&gt;
      f t
      &gt;&gt;= fun result -&gt;
      let new_permissions =
        match current_file_permissions with
        | None -&gt;
<span style="color:#86cdea;font-weight:bold;">@@@@@@@@@@ </span><span style="color:#86cdea;font-weight:bold;">-958,34 +953,34</span><span style="color:#86cdea;font-weight:bold;"> @@@@@@@@@@</span>
    &gt;&gt;| function
      | Ok () -&gt; result
      | Error exn -&gt;
        don't_wait_for (Unix.unlink temp_file);
        failwiths &quot;Writer.with_file_atomic could not create file&quot;
          (file, exn) &lt;:sexp_of&lt; string * exn &gt;&gt;
  ;;
  
<span style="color:#fe8686;font-weight:bold;">-|</span>let save <span style="color:#fe8686;text-decoration:underline;font-weight:bold;">?temp_prefix</span> ?perm ?fsync file ~contents =
<span style="color:#fe8686;font-weight:bold;">-|</span>  with_file_atomic <span style="color:#fe8686;text-decoration:underline;font-weight:bold;">?temp_prefix</span> ?perm ?fsync file ~f:(fun t -&gt;
<span style="color:#aece92;font-weight:bold;">+|</span>let save <span style="color:#aece92;">?temp_file</span> ?perm ?fsync file ~contents =
<span style="color:#aece92;font-weight:bold;">+|</span>  with_file_atomic <span style="color:#aece92;">?temp_file</span> ?perm ?fsync file ~f:(fun t -&gt;
      write t contents;
      Deferred.unit)
  ;;
  
  let sexp_to_buffer ?(hum = true) ~buf sexp =
    if hum then
      Sexp.to_buffer_hum ~buf sexp
    else
      Sexp.to_buffer_mach ~buf sexp
  ;;
  
<span style="color:#fe8686;font-weight:bold;">-|</span>let save_sexp <span style="color:#fe8686;text-decoration:underline;font-weight:bold;">?temp_prefix</span> ?perm ?fsync ?hum file sexp =
<span style="color:#aece92;font-weight:bold;">+|</span>let save_sexp <span style="color:#aece92;">?temp_file</span> ?perm ?fsync ?hum file sexp =
    let buf = Buffer.create 1 in
    sexp_to_buffer ?hum ~buf sexp;
    Buffer.add_char buf '\n';
<span style="color:#fe8686;font-weight:bold;">-|</span>  save <span style="color:#fe8686;text-decoration:underline;font-weight:bold;">?temp_prefix</span> ?perm ?fsync file ~contents:(Buffer.contents buf);
<span style="color:#aece92;font-weight:bold;">+|</span>  save <span style="color:#aece92;">?temp_file</span> ?perm ?fsync file ~contents:(Buffer.contents buf);
  ;;
  
  let transfer t pipe_r write_f =
    let producers_to_flush_at_close_elt =
      Bag.add t.producers_to_flush_at_close (fun () -&gt;
        Deferred.ignore (Pipe.upstream_flushed pipe_r))
    in
    let consumer =
</pre>
