---
layout: post
title:  "Some (Unscientific but Still) Interesting Ruby VM benchmarks"
---

Encouraged by the [impressive progresses of the TruffleRuby project](https://medium.com/graalvm/benchmarking-cruby-mjit-yjit-jruby-and-truffleruby-6a7178ca6906), a 
[Twitter
conversation](https://twitter.com/schneems/status/1479441562694205449) with Richard Schneeman, and the beginning of the new year, I decided to run
some benchmarks to evaluate the performance with a heavyduty number crunching
project such as [my own sisfc simulator](https://github.com/mtortonesi/sisfc).


## Experiment setup

I used the latest version of sisfc [(commit
f30dc92)](https://github.com/mtortonesi/sisfc/commit/f30dc92077b44732d164a69660c8c4f5ed6c3968)
and ran it using the example configuration provided in the examples directory.
I ran the experiments on my laptop, a 2019 MacBookPro with an 8-core Intel
i7-8569U 2.80GHz CPU, 16 GiB RAM and a 64-bit version of Monterey (MacOS 12.5).

Here is the benchmark script:

```sh
#!/bin/sh

TIME_BASED_ID=$(date +%Y%m%d%H%M%S)

FILENAME=profile_log_${TIME_BASED_ID}.txt

echo "Results for JRuby" > $FILENAME
rbenv local "jruby-9.3.2.0"
export JRUBY_OPTS="-Xcompile.invokedynamic=true"
/usr/bin/time -h -l -a -o $FILENAME bundle exec ./bin/sisfc examples/simulator.conf examples/vm_allocation.conf

echo "Results for MRI" >> $FILENAME
rbenv local "3.1.0"
/usr/bin/time -h -l -a -o $FILENAME bundle exec ./bin/sisfc examples/simulator.conf examples/vm_allocation.conf

echo "Results for TruffleRuby" >> $FILENAME
rbenv local "truffleruby+graalvm-21.3.0"
/usr/bin/time -h -l -a -o $FILENAME bundle exec ./bin/sisfc examples/simulator.conf examples/vm_allocation.conf
```

## Results

And here are the results:

```
Results for JRuby 9.3.2.0
	13,81s real		25,66s user		0,91s sys
           411623424  maximum resident set size
                   0  average shared memory size
                   0  average unshared data size
                   0  average unshared stack size
              300014  page reclaims
                  15  page faults
                   0  swaps
                   0  block input operations
                   0  block output operations
                   0  messages sent
                   0  messages received
                  24  signals received
                 214  voluntary context switches
               27078  involuntary context switches

Results for MRI 3.1.0
	10,59s real		10,34s user		0,17s sys
            24338432  maximum resident set size
                   0  average shared memory size
                   0  average unshared data size
                   0  average unshared stack size
               22039  page reclaims
                   4  page faults
                   0  swaps
                   0  block input operations
                   0  block output operations
                   0  messages sent
                   0  messages received
                  13  signals received
                  28  voluntary context switches
                6264  involuntary context switches

Results for TruffleRuby+GraalVM 21.3.0
	2m22,52s real		3m43,22s user		3,04s sys
          1248636928  maximum resident set size
                   0  average shared memory size
                   0  average unshared data size
                   0  average unshared stack size
             1844862  page reclaims
                   0  page faults
                   0  swaps
                   0  block input operations
                   0  block output operations
                   0  messages sent
                   0  messages received
                  11  signals received
                  34  voluntary context switches
               66243  involuntary context switches
```

As you can see, MRI is the fastest and less memory hungry of the bunch. JRuby
takes 30% more time to run the benchmark but shows a quite impressive memory
overhead: 16.9x max RSS! TruffleRuby takes this tendency to the extreme. It
performs very poorly in this particular benchmark, taking almost 13.5 times
longer than MRI to complete and using a very significant amount of memory (more
than 51x max RSS of MRI), and forces the laptop fan to work overtime.

Mind you, by publishing these results I don't want to bash on JRuby and
TruffleRuby. In fact, I don't believe these are actually bad results for JRuby
and TruffleRuby! They only mean that the default configuration of those VMs is
not terribly well suited for this particular task. Indeed, the fact that
TruffleRuby 21.3 is even capable of completing the job - something that
TruffleRuby 20.x couldn't manage do - is a testament to the maturity of the
project.

Instead, my purpose is purely educational. I am interested in learning how to
properly optimize JRuby and TruffleRuby to obtain the best performance with
this benchmark. Unfortunately, I couldn't find much information about how to
tune JRuby and TruffleRuby. (In the latter case, I recently bought and read the
book "Supercharge your applications with GraalVM", which is a pretty good
introduction to GraalVM but has a limited coverage of TruffleRuby.) If you have
some suggestion, please reach out to me. I am looking forward to receiving some
guidance. 
