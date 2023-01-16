---
title: Code-makeover on how to code for better efficiency
teaching: 15
exercises: 0
questions:
- How to write the most efficient code?
objectives:
- Learn good tips and tools to improve your code.
keypoints:
- CPU, memory, and build time optimizations are possible when good code practices are followed.
---

## DUNE Computing Training

## Session Video

<!--The session will be captured on video a placed here after the workshop for asynchronous study.-->
The session was video captured for your asynchronous review. The video from the full two day training in May 2022 is provided here as a reference.

<center>
<iframe width="560" height="315" src="https://www.youtube.com/embed/z7myUSaGYnQ" title="DUNE Computing Tutorial May 2022 Code-makeover on how to code for better efficiency" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</center>

### Code Make-over

**How to improve your code for better efficiency**

DUNE simulation, reconstruction and analysis jobs take a lot of memory and CPU time.  This owes to the large size of the Far Detector modules as well as the many channels in the Near Detectors.  Reading out a large volume for a long time with high granularity creates a lot of data that needs to be stored and processed.

### CPU optimization:

**Run  with the “prof” build when launching big jobs.**  While both the "debug" and "prof" builds have debugging and profiling information included in the executables and shared libraries, the "prof" build has a high level of compiler optimization turned on while "debug" has optimizations disabled.  Debugging with the "prof" build can be done, but it is more difficult because operations can be reordered and some variables get put in CPU registers instead of inspectable memory. The “debug” builds are generally much slower, by a factor of four or more.  Often this difference is so stark that the time spent repeatedly waiting for a slow program to chug through the first trigger record in an interactive debugging session is more costly than the inconvenience of not being able to see some of the variables in the debugger.  If you are not debugging, then there really is (almost) no reason to use the “debug” builds.  If your program produces a different result when run with the debug build and the prof build (and it’s not just the random seed), then there is a bug to be investigated.

**Compile your interactive ROOT scripts instead of running them in the interpreter**   At the ROOT prompt, use .L myprogram.C++ (even though its filename is myprogram.C).  Also .x myprogram.C++ will compile and then execute it.  This will force a compile.  .L myprogram.C+ will compile it only if necessary.

**Run gprof or other profilers like valgrind's callgrind:**   You might be surprised at what is actually taking all the time in your program.  There is abundant documentation on the [web][gnu-manuals-gprof], and also the valgrind online documentation.
There is no reason to profile a "debug" build and there is no need to hand-optimize something the compiler will optimize anyway, and which may even hurt the optimality of the compiler-optimized version.

**The Debugger can be used as a simple profiler:** If your program is horrendously slow (and/or it used to be fast), pausing it at any time is likely to pause it while it is doing its slow thing.  Run your program in the debugger, pause it when you think it is doing its slow thing (i.e. after initialization), and look at the call stack.  This technique can be handy because you can then inspect the values of variables that might give a clue if there’s a bug making your program slow.  (e.g. looping over 10<sup>15</sup> wires in the Far Detector, which would indicate a bug, such as an uninitialized loop counter or an unsigned loop counter that is initialized with a negative value.

**Don't perform calculations or do file i/o that will only later  be ignored.**  It's just a waste of time.  If you need to pre-write some code because in future versions of your program the calculation is not ignored, comment it out, or put a test around it so it doesn't get executed when it is not needed.


**Extract constant calculations out of loops.**


<div style="display: grid;grid-template-columns: repeat(2,460px);grip-gap: 5px;width:1120px;border: 2px solid #ffffff;font-family:Courier, monospace;color: #000000;font-size: 10pt;">

<div style="background-color: #EFEAF4; border: 1px solid #000000;text-align: center; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;color: #280071;border-left-width: thick; border-left-color: #280071;border-radius: 5px;">Code Example (BAD)</div>

<div style="background-color: #EFEAF4; border: 1px solid #000000;text-align: center; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;color: #280071;">Code Example (GOOD)</div>

<div style="background-color: #EBEBEB; border: 1px solid #000000;text-align: left; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px; border-left-width: thick; border-left-color: #280071;border-radius: 5px;">
double sum = 0;<br>
for (size_t i=0; i<n_channels; ++i)<br>
{<br>
  sum += result.at(i)/TMath::Sqrt(2.0);<br>
}<br>
</div>
<div style="background-color: #EBEBEB; border: 1px solid #000000;text-align: left; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;">
double sum = 0;<br>
double f = TMath::Sqrt(0.5);<br>
for (size_t i=0; i<n_channels; ++i)<br>
{<br>
  sum += result.at(i)*f;<br>
}<br>
</div>

</div><!--side by side table by DeMuth-->


The example above also takes advantage of the fact that floating-point multiplies generally have significantly less latency than floating-point divides (this is still true, even with modern CPUs).

**Use sqrt():** Don’t use `pow()` or `TMath::Power` when a multiplication or `sqrt()` function can be used.

<div style="display: grid;grid-template-columns: repeat(2,460px);grip-gap: 5px;width:1120px;border: 2px solid #ffffff;font-family:Courier, monospace;color: #000000;font-size: 10pt;">

<div style="background-color: #EFEAF4; border: 1px solid #000000;text-align: center; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;color: #280071;border-left-width: thick; border-left-color: #280071;border-radius: 5px;">Code Example (BAD)</div>

<div style="background-color: #EFEAF4; border: 1px solid #000000;text-align: center; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;color: #280071;">Code Example (GOOD)</div>
<div style="background-color: #EBEBEB; border: 1px solid #000000;text-align: left; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px; border-left-width: thick; border-left-color: #280071;border-radius: 5px;">
double r = TMath::Power(  TMath::Power(x,2) + TMath::Power(y,2), 0.5);
</div>

<div style="background-color: #EBEBEB; border: 1px solid #000000;text-align: left; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;">
double r = TMath::Sqrt( x*x + y*y );
</div>
</div>

The reason is that `TMath::Power` (or the C math library’s `pow()`) function must take the logarithm of one of its arguments, multiply it by the other argument, and exponentiate the result.  Modern CPUs have a built-in `SQRT` instruction.  Modern versions of `pow()` or `Power` may check the power argument for 2 and 0.5 and instead perform multiplies and `SQRT`, but don’t count on it.

If the things you are squaring above are complicated expressions, use `TMath::Sq()` to eliminate the need for typing them out twice or creating temporary variables.  Or worse, evaluating slow functions twice.  The optimizer cannot optimize the second call to that function because it may have side effects like printing something out to the screen or updating some internal variable and you may have intended for it to be called twice.


<div style="display: grid;grid-template-columns: repeat(2,460px);grip-gap: 5px;width:1120px;border: 2px solid #ffffff;font-family:Courier, monospace;color: #000000;font-size: 10pt;">
<div style="background-color: #EFEAF4; border: 1px solid #000000;text-align: center; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;color: #280071;border-left-width: thick; border-left-color: #280071;border-radius: 5px;">Code Example (BAD)</div>

<div style="background-color: #EFEAF4; border: 1px solid #000000;text-align: center; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;color: #280071;">Code Example (GOOD)</div>

<div style="background-color: #EBEBEB; border: 1px solid #000000;text-align: left; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px; border-left-width: thick; border-left-color: #280071;border-radius: 5px;">
double r = TMath::Sqrt( slow_function_calculating_x()*<br>slow_function_calculating_x() + <br> slow_function_calculating_y()* <br> slow_function_calculating_y()  );
</div>

<div style="background-color: #EBEBEB; border: 1px solid #000000;text-align: left; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;">
double r = TMath::Sqrt( TMath::Sq(slow_function_calculating_x()) + <br> TMath::Sq(slow_function_calculating_y()));
</div>
</div>

**Don't call `sqrt()` if you don’t have to.**

<div style="display: grid;grid-template-columns: repeat(2,460px);grip-gap: 5px;width:1120px;border: 2px solid #ffffff;font-family:Courier, monospace;color: #000000;font-size: 10pt;">
<div style="background-color: #EFEAF4; border: 1px solid #000000;text-align: center; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;color: #280071;border-left-width: thick; border-left-color: #280071;border-radius: 5px;">Code Example (BAD)</div>

<div style="background-color: #EFEAF4; border: 1px solid #000000;text-align: center; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;color: #280071;">Code Example (GOOD)</div>

<div style="background-color: #EBEBEB; border: 1px solid #000000;text-align: left; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px; border-left-width: thick; border-left-color: #280071;border-radius: 5px;">
if (TMath::Sqrt( x*x + y*y ) < rcut )<br>
{<br>
  do_something();<br>
}
</div>

<div style="background-color: #EBEBEB; border: 1px solid #000000;text-align: left; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;">
double rcutsq = rcut*rcut;<br>
if (x*x + y*y < rcutsq)<br>
{<br>
  do_something();<br>
}
</div>
</div>

<!--
hopefully factor out of loops this line:

~~~
double rcutsq = rcut*rcut;
~~~
{: .source}

and then execute:

~~~
if ( x*x + y*y < rcutsq)  
{  
  do_something();  
}  
~~~
{: .source}
-->

**Use binary search features in the STL rather than a step-by-step lookup.**

~~~
std::vector<int> my_vector;
(fill my_vector with stuff)

size_t indexfound = 0;
bool found = false;
for (size_t i=0; i<my_vector.size(); ++i)
{
  if (my_vector.at(i) == desired_value)
    {
	indexfound = i;
	found = true;
    }
}
~~~
{: .source}

If you have to search through a list of items many times, it is best to sort it and use std::lower_bound; see the example [here][cpp-lower-bound].
`std::map` is sorted, and `std::unordered_map` uses a quicker hash table.  Generally looking things up in maps is `O(log(n))` and in a `std::unordered_map` is `O(1)` in CPU time, while searching for it from the beginning is `O(n)`.  The bad example above can be sped up by an average factor of 2 by putting a break statement after `found=true;` if you want to find the first instance of an object.  If you want to find the last instance, just count backwards and stop at the first one you find; or use `std::upper_bound`.

**Don’t needlessly mix floats and doubles.**


<div style="display: grid;grid-template-columns: repeat(2,460px);grip-gap: 5px;width:1120px;border: 2px solid #ffffff;font-family:Courier, monospace;color: #000000;font-size: 10pt;">
<div style="background-color: #EFEAF4; border: 1px solid #000000;text-align: center; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;color: #280071;border-left-width: thick; border-left-color: #280071;border-radius: 5px;">Code Example (BAD)</div>

<div style="background-color: #EFEAF4; border: 1px solid #000000;text-align: center; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;color: #280071;">Code Example (GOOD)</div>

<div style="background-color: #EBEBEB; border: 1px solid #000000;text-align: left; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px; border-left-width: thick; border-left-color: #280071;border-radius: 5px;">
double sum = 0;<br>
std::vector &lt;double&gt; results;<br>
(fill lots of results)<br>
for (size_t i=0; i<results.size(); ++i)<br>
{<br>
  float rsq = results.at(i)*result.at(i);<br>
  sum += rsq;<br>
}
</div>

<div style="background-color: #EBEBEB; border: 1px solid #000000;text-align: left; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;">
double sum = 0;<br>
std::vector &lt;double&gt; results;<br>
(fill lots of results)<br>
for (size_t i=0; i<results.size(); ++i)<br>
{<br>
  sum += TMath::Sq(results.at(i));<br>
}
</div>
</div>

**Minimize conversions between int and float or double**

The up-conversion from int to float takes time, and the down-conversion from float to int loses precision and also takes time.  Sometimes you want the precision loss, but sometimes it's a mistake.

**Check for NaN and Inf.**  While your program will still function if an intermediate result is `NaN` or `Inf` (and it may even produce valid output, especially if the `NaN` or `Inf` is irrelevant), processing `NaN`s and `Inf`s is slower than processing valid numbers.  Letting a `NaN` or an `Inf` propagate through your calculations is almost never the right thing to do - check functions for domain validity (square roots of negative numbers, logarithms of zero or negative numbers, divide by zero, etc.) when you execute them and decide at that point what to do.  If you have a lengthy computation and the end result is `NaN`, it is often ambiguous at what stage the computation failed.

**Pass objects by reference.**  Especially big ones.  C and C++ call semantics specify that objects are passed by value by default, meaning that the called method gets a copy of the input.  This is okay for scalar quantities like int and float, but not okay for a big vector, for example.  The thing to note then is that the called method may modify the contents of the passed object, while an object passed by value can be expected not to be modified by the called method.

**Use references to receive returned objects created by methods**  That way they don't get copied.  The example below is from the VD coldbox channel map.  Bad, inefficient code courtesy of Tom Junk, and good code suggestion courtesy of Alessandro Thea.  The infotohcanmap object is a map of maps of maps:  std::unordered_map<int,std::unordered_map<int,std::unordered_map<int,int> > > infotochanmap;

<div style="display: grid;grid-template-columns: repeat(2,460px);grip-gap: 5px;width:1120px;border: 2px solid #ffffff;font-family:Courier, monospace;color: #000000;font-size: 10pt;">
<div style="background-color: #EFEAF4; border: 1px solid #000000;text-align: center; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;color: #280071;border-left-width: thick; border-left-color: #280071;border-radius: 5px;">Code Example (BAD)</div>

<div style="background-color: #EFEAF4; border: 1px solid #000000;text-align: center; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;color: #280071;">Code Example (GOOD)</div>

<div style="background-color: #EBEBEB; border: 1px solid #000000;text-align: left; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px; border-left-width: thick; border-left-color: #280071;border-radius: 5px;">
int dune::VDColdboxChannelMapService::getOfflChanFromWIBConnectorInfo(int wib, int wibconnector, int cechan)<br>
{<br>
  int r = -1;<br>
  auto fm1 = infotochanmap.find(wib);<br>
  if (fm1 == infotochanmap.end()) return r;<br>
  auto m1 = fm1-&gt;second;<br>
  auto fm2 = m1.find(wibconnector);<br>
  if (fm2 == m1.end()) return r;<br>
  auto m2 = fm2-&gt;second;<br>
  auto fm3 = m2.find(cechan);<br>
  if (fm3 == m2.end()) return r;<br>
  r = fm3->second;  <br>
  return r;<br>
</div>
<div style="background-color: #EBEBEB; border: 1px solid #000000;text-align: left; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;">
int dune::VDColdboxChannelMapService::getOfflChanFromWIBConnectorInfo(int wib, int wibconnector, int cechan)<br>
{<br>
  int r = -1;<br>
  auto fm1 = infotochanmap.find(wib);<br>
  if (fm1 == infotochanmap.end()) return r;<br>
  auto& m1 = fm1-&gt;second;<br>
  auto fm2 = m1.find(wibconnector);<br>
  if (fm2 == m1.end()) return r;<br>
  auto& m2 = fm2-&gt;second;<br>
  auto fm3 = m2.find(cechan);<br>
  if (fm3 == m2.end()) return r;<br>
  r = fm3->second;  <br>
  return r;<br>
}<br>
</div>
</div>

**Minimize cloning TH1’s.**  It is really slow.

**Minimize formatted I/O.**   Formatting strings for output is CPU-consuming, even if they are never printed to the screen or output to your logfile.  `MF_LOG_INFO` calls for example must prepare the string for printing even if it is configured not to output it.

**Avoid using caught exceptions as part of normal program operation**  While this isn't an efficiency issue or even a code readability issue, it is a problem when debugging programs.  Most debuggers have a feature to set a breakpoint on thrown exceptions.  This is sometimes necessary to use in order to track down a stubborn bug.  Bugs that stop program execution like segmentation faults are sometimes easer to track down than caught exceptions (which often aren't even bugs but sometimes they are).  If many caught exceptions take place before the buggy one, then the breakpoint on thrown exceptions has limited value.

**Use sparse matrix tools where appropriate.**  This also saves memory.

**Minimize database access operations.**  Bundle the queries together in blocks if possible.  Do not pull more information than is needed out of the database.  Cache results so you don’t have to repeat the same data retrieval operation.

Use `std::vector::reserve()` in order to size your vector right if you know in advance how big it will be.  `std::vector()` will, if you `push_back()` to expand it beyond its current size in memory, allocate twice the memory of the existing vector and copy the contents of the old vector to the new memory.  This operation will be repeated each time you start with a zero-size vector and push_back a lot of data.
Factorize your program into parts that do i/o and compute.   That way, if you don’t need to do one of them, you can switch it off without having to rewrite everything.  Example:  Say you read data in from a file and make a histogram that you are sometimes interested in looking at but usually not.  The data reader should not always make the histogram by default but it should be put in a separate module which can be steered with fcl so the computations needed to calculate the items to fill the histogram can be saved.

## Memory optimization:

Use `valgrind`.  Its default operation checks for memory leaks and invalid accesses.  Search the output for the words “invalid” and “lost”.  Valgrind is a `UPS` product you can set up along with everything else.  It is set up as part of the dunesw stack.

~~~
setup valgrind
valgrind --leak-check=yes --suppressions=$ROOTSYS/etc/valgrind-root.supp myprog arg1 arg2
~~~
{: .source}

More information is available [here][valgrind-quickstart].  ROOT-specific suppressions are described [here][valgrind-root].  You can omit them, but your output file will be cluttered up with messages about things that ROOT does routinely that are not bugs.

Use `massif`.  `massif` is a heap checker, a tool provided with `valgrind`; see documentation [here][valgrind-ms-manual].

**Free up memory after use.**  Don’t hoard it after your module’s exited.

**Don’t constantly re-allocate memory if you know you’re going to use it again right away.**

**Use STL containers instead of fixed-size arrays, to allow for  growth in size.**   Back in the bad old days (Fortran 77 and earlier), fixed-size arrays had to be declared at compile time that were as big as they possibly could be, both wasting memory on average and creating artificial cutoffs on the sizes of problems that could be handled.  This behavior is very easy to replicate in C++.  Don’t do it.

**Be familiar with the structure and access idioms.** These include `std::vector`, `std::map`, `std::unordered_map`, `std::set`, `std::list`.

**Minimize the use of new and delete to reduce the chances of memory leaks.**  If your program doesn’t leak memory now, that’s great, but years from now after maintenance has been transferred, someone might introduce a memory leak.

**Use move semantics to transfer data ownership without copying it.**

**Do not store an entire event’s worth of raw digits in memory all at once.**  Find some way to process the data in pieces.

**Consider using more compact representations in memory.**  A `float` takes half the space of a double.  A `size_t` is 64 bits long (usually).  Often that’s needed, but sometimes it’s overkill.

**Optimize the big uses and don’t spend a lot of time on things that don’t matter.**  If you have one instance of a loop counter that’s a `size_t` and it loops over a million vector entries, each of which is an `int`, look at the entries of the vector, not the loop counter (which ought to be on the stack anyway).

**Rebin histograms.**  Some histograms, say binned in channels <i>x</i> ticks or channels <i>x</i> frequency bins for a 2D FFT plot, can get very memory hungry.

## I/O optimization:

**Do as much calculation as you can per data element read.**  You can spin over a TTree once per plot, or you can spin through the TTree once and make all the plots.  ROOT compresses data by default on write and uncompresses it on readin, so this is both an I/O and a CPU issue, to minimize the data that are read.

**Read only the data you need**  ROOT's TTree access methods are set up to give you only the requested TBranches.  If you use TTree::MakeClass to write a template analysis ROOT macro script, it will generate code that reads in _all_ TBranches and leaves.  It is easy to trim out the extras to speed up your workflow.

**Saving compressed data reduces I/O time and storage needs.**  Even though compressing data takes CPU, a slow disk or network can mean your workflow is in fact faster to trade CPU time instead of the disk read time.

**Stream data with xrootd**  You will wait less for your first event than if you copy the file, put less stress on the data storage elements, and have more reliable i/o with dCache.

## Build time optimization:

**Minimize the number of #included files.**  If you don’t need an #include, don’t use it.  It takes time to find these files in the search path and include them.

**Break up very large source files into pieces.**   `g++’s` analysis and optimization steps take an amount of time that grows faster than linearly with the number of source lines.

**Use ninja instead of make**   Instructions are [here][ninjadocpageredmine]

## Workflow optimization:

**Pre-stage your datasets**  It takes a lot of time to wait for a tape (sometimes hours!).  CPUs are accounted by wall-clock time, whether you're using them or not.  So if your jobs are waiting for data, they will run slowly even if you optimized the CPU usage.  Pre-stage your data!

**Run a test job**  If you have a bug, you will save time by not submitting large numbers of jobs that might not work.

**Write out your variables in your own analysis ntuples (TTrees)**  You will likely have to run over the same MC and data events repeatedly, and the faster this is the better. You will have to adjust your cuts, tune your algorithms, estimate systematic uncertainties, train your deep-learning functions, debug your program, and tweak the appearance of your plots.  Ideally, if the data you need to do these operatios is available interctively, you will be able to perform these tasks faster.  Choose a minimal set of variables to put in your ntuples to save on storage space.

**Write out histograms to ROOTfiles and decorate them in a separate script**  You may need to experiment many times with borders, spacing, ticks, fonts, colors, line widths, shading, labels, titles, legends, axis ranges, etc.  Best not to have to re-compute the contents when you're doing this, so save the histograms to a file first and read it in to touch it up for presentation.

## Software readability and maintainability:

**Keep the test suite up to date**   dunesw and larsoft have many examples of unit tests and integration tests.  A colleague's commit to your code or even to a different piece of code or even a data file might break your code in unexpected, difficult-to-diagnose ways.  The continuous integration (CI) system is there to catch such breakage, and even small changes in run time, memory consumption, and data product output.

**Keep your methods short**  If you have loaded up a lot of functionality in a method, it may become hard to reuse the components to do similar things.  A long method is probably doing a lot of different things that can be given meaningful names.

**Update the comments when code changes**   Not many things are more confusing than an out-of-date-comment that refers to how code used to work long ago.

**Update names when meaning changes** As software evolves, the meaning of the variables may shift.  It may be a quick fix to change the contents of a variable without changing its name, but some variables may then contain contents that is the opposite of what the variable name implies.  While the code will run, future maintainers will get confused.

**Use const frequently** The const keyword prevents overwriting variables unintentionally.  Constness is how *art* protects the data in its event memory.  This mechanism is exposed to the user in that pointers to const memory must be declared as pointers to consts, or you will get obscure error messages from the compiler.  Const can also protect you from yourself and your colleagues when you know that the contents of a variable ought not to change.

**Use simple constructs even if they are more verbose**  Sometimes very clever, terse expressions get the job done, but they can be difficult for a human to understand if and when that person must make a change.   There is an [obfuscated C contest][obfuscated-C] if you want to see examples of difficult-to-read code (that may in fact be very efficient!  But people time is important, too).

**Always initialize variables when you declare them**  Compilers will warn about the use of uninitialized variables, so you will get used to doing this anyway.  The initialization step takes a little time and it is not needed if the first use of the memory is to set the variable, which is why compilers do not automatically initialize variables.

**Minimize the scope of variables**  Often a variable will only have a meaningful value iniside of a loop.  You can declare variables as you use them.  Old langauges like Fortran 77 insisted that you declare all variables at the start of a program block.  This is not true in C and C++.  Declaring variables inside of blocks delimiated by braces means they will go out of scope when the program exits the block, both freeing the memory and preventing you from referring to the variable after the loop is done and only considering the last value it took.  Sometimes this is the desired behaviour, though, and so this is not a blanket rule.

## Coding for Thread Safety

Modern CPUs often have many cores available.  It is not unusual for a grid worker node to have as many as 64 cores on it, and 128 GB of RAM.  Making use of the available hardware to maximize throughput is an important way to optimize our time and resources.  DUNE jobs tend to be "embarrassingly parallel", in that they can be divided up into many small jobs that do not need to communicated with one another.  Therefore, making use of all the cores on a grid node is usually as easy as breaking a task up into many small jobs and letting the grid schedulers work out what jobs run where.  The issue however is effective memory usage.  If several small jobs share a lot of memory whose contents do not change (code libraries loaded into RAM, geometry description, calibration constants), then one can group the work together into a single job that uses multiple threads to get the work done faster.  If the memory usage of a job is dominated by per-event data, then loading multiple events' worth of data in RAM in order to keep all the cores fed with data may not provide a noticeable improvement in the utilization of CPU time relative to memory time.

Sometimes multithreading has advantages within a trigger record.  Data from different wires or APAs may be processed simultaneously.  One thing software managers would like to make sure is controllable is the number of threads a program is allowed to spawn.  Some grid sites do not have an automatic protection against a program that creates more threads than CPUs it has requested.  Instead, a human operator may notice that the load on a system is far greater than the number of cores, and track down and ban the offending job sumitter (this has already happened on DUNE).  If a program contains components, some of which manage their own threawds, then it becomes hard to manage the total thread count in a program.  Multithreaded *art* keeps track of the total thread count using TBB, or Thread Building Blocks.

See this very thorough [presentation][knoepfel-thread-safety] by Kyle Knoepfel at the 2019 LArSoft [workshop][LArSoftWorkshop2019].  Several other talks at the workshop also focus on multi-threaded software.  In short, if data are shared between threads and they are mutable, this is a recipe for race conditions and non-reproducible behavior of programs.  Giving each thread a separate instance of each object is one way to contain possible race conditions.  Alternately, private and public class members which do not change or which have synchronous access methods can also help provide thread safety.


[cpp-lower-bound]: https://en.cppreference.com/w/cpp/algorithm/lower_bound
[gnu-manuals-gprof]: https://ftp.gnu.org/old-gnu/Manuals/gprof-2.9.1/html_mono/gprof.html
[valgrind-quickstart]: https://www.valgrind.org/docs/manual/quick-start.html
[valgrind-ms-manual]: https://www.valgrind.org/docs/manual/ms-manual.html
[ninjadocpageredmine]: https://cdcvs.fnal.gov/redmine/projects/dunetpc/wiki/_Tutorial_#Using-the-ninja-build-system-instead-of-make
[valgrind-root]: https://root-forum.cern.ch/t/valgrind-and-root/28506
[obfuscated-C]: https://www.ioccc.org/
[knoepfel-thread-safety]: https://indico.fnal.gov/event/20453/contributions/57777/attachments/36182/44065/2019-LArSoftWorkshop-ThreadSafety.pdf
[LArSoftWorkshop2019]: https://indico.fnal.gov/event/20453/timetable/?view=standard

{%include links.md%}
