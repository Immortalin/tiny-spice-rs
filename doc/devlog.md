# Development Log

## 2018-11-26
Between upgrading my desktop, getting a new laptop, losing and finding my
github stick, I needed to consolidate my github repos.


## 2018-02-02
Can't remember where I was and what's left to do. 

Ok, now I remember - I'm trying to write a spice deck reader. I'm not sure how far
I got, and what I'd consider a minimum viable product.

Suppose:
1. I, V, R, C, D
2. `trans` and `op` in `.control` blocks
3. Test all this
4. Waveforms? Do I need a `--output` switch, maybe?

Getting weird results from the command line testing:

    test spice_irrc ... ok
    test spice_irrrr ...   [ELEMENT] Current source: 0.015709353792572548A into node 0 and out of node 1
    test spice_reader ... 

It looks like `.success()` is not reliable. 

How do you test a SPICE engine?


## 2017-11-23
Rustup

https://users.rust-lang.org/t/how-do-you-test-binaries-not-libraries/9554/9


## 2017-11-17
Can read SPICE files now (crudely). The simulator engine now uses a configuration
object to store parameters like TSTEP, RELTOL and waveform filenames. All tests
have been updated to use this new scheme.

Todo:
* find out how to test binaries

Decisions:
* not going to support multiple analyses in .control block


## 2017-11-07
Started `tiny-spice.rs` which is the toplevel binary to tie everything
together. I'm trying to write this and the SPICE file reader at the 
same time so I can figure out what the interface should be.

## 2017-11-06
Implemented thing to read values like 1 or 1.0 or 1.0u.

## 2017-11-05
Initial musings on a SPICE deck reader.


## 2017-11-04
Adding a small cap across the diodes seems to make everything happy.

Fixed up a bunch of testcases and stamped version [TAG 0.6.0]

## 2017-11-03
Created a program to plot all the results from the loops I've got going.
It's called `r8n` and it plops a dot down at the last point of the waveform
that was calculated - showing me easily what broke.

Developed this to test out the SPICE engine as I have it now. I'm getting
failures on the diode-bridge loop that has an RC load. There's no obvious
correlation between the failing testcase and cap, or timestep or anything.


## 2017-11-02
Threw a Hail Mary and changed `f32` to `f64` throughout the program. This
seems to fix things!

What I'd like to do before stamping the next release:
 1. Look at waveforms generated by `fullbridge_loop`
 2. Make all sims pass
 3. Try reverting diode model back to a previous version to see
   how simple it can be and still converge.

What do I want to do with this in the future?
 * SPICE card deck reader
 * Circuit topology checks - no DC path to ground, unconnected nodes etc.
 * Try out some different integration methods
 * MOSFET model
 * Hook up to my tiny-verilog simulator to make a mixed-mode thing
 * Linear, NonLinear, NonLinearDiff partitions
 * Sub-circuits

What's this for, though? Why am I doing all this again?


## 2017-11-01
Downloaded SPICE2 FORTRAN code and coded up the diode voltage limiting
algorithm to match.

Still failing transient sims.


## 2017-10-31
More messing with the Colon/Nagel thing. Use `Cell` to remember the previous
values of things without making the entire `Diode` structure mutable and 
infecting the simulation engine.


## 2017-10-24
Some light internet searching. Maybe I have to try GMIN-stepping and
source-stepping algorithms?

Could I ask someone at work? One of the Daves? Marie?


## 2017-10-23
Colon didn't help. The V_crit is around the knee of the diode, so doesn't help
with the cycles too much.


## 2017-10-22
Going to try that Colon thing I found in Nagel. For this though, I need to know
the previous value of the voltage across the diode. Should I just add this to the
structure or what?


## 2017-10-21
Found Nagel's SPICE2 paper.


## 2017-10-14
I spent the last month or so trying to come up with a fix for the diode model
transient analysis problem. There was a bug in how I calculated G_Eq for the
diode companion model. I've made some progress, but the diode model doesn't
seem robust yet in transient analysis.

In the course of this investigation, the engine has been updated to print more
information about which RETOL etc it's using, and it spits out messages when it 
changes the timestep. I also make all the analyses return a result datastructure
so that the unit-tests can determine if the circuit time-stepped-to-small or not,
which is useful for robustness testing.


The circuit I used during the investigation was:
* Current source and resistor - 20V
* Diode bridge with 2 diodes commented out
* 1k load resistor

The diodes are in series with the resistor, one before the R, and one after. This
turns out to be a harder circuit for the simulator to solve than the full diode
bridge! The solver seems to be getting into a limit-cycle when solving for the
case where the input falls and crosses 0V and the diodes start to go into reverse
bias.

At least I think the simulator finds the 2-diode circuit more difficult to deal
with. What I really need to see is a full test suite - same circuit but varying
all the VNTOL, RETOL, saturation current of the diodes, etc.

I also need a SPICE file reader. And I need to decide when I'm finished.

I'm tempted to write the SPICE file reader next, as this will help with gathering
together circuits for the robustness testing. But this isn't trivial - I'll need
to write a parser, I'll need symbol tables, and I'll probably be tempted to expand
to control blocks and option blocks.

For robustness testing, it's this or writing a bunch of testcases in Rust which'll
all write to the same log file and stuff. I need to recompile every time there's
a circuit change.

## 2017-09-16
Had a go at simulating capacitors in transient.

With an RC low-pass filter test circuit, I'm seeing a few problems:
1. The output wave seems to depend on timestep, not on input wave
  frequency.
2. The output wave is leading the input wave.

[FIX] - wire up the current source in the capacitor companion model in the
correct direction.

And I've noticed on the diode bridge sim:
1. It's not very robust with timestep.


## 2017-09-15
Got transient analysis working! I even tried it on the diode bridge
circuit and it was a success!

Interestingly, even with nonlinear devices in there, the timestep maxes out on
this. Maybe that's to be expected - I've nothing that holds energy in the circuit
so I don't need integration and all that.

The logfile produced for the diode-bridge transient simulation is 4.5M. This is
huge.

Where to next?
* L & C models
* Write out engine metrics
* Propper logging to quieten output
* MOSFET model?
* The LTE timestep thing?


## 2017-09-14
Think I've an implementation of the iteration-count time-marching loop. Albeit
that I've just ran it on a static circuit, but at least I can see the timesteps
increasing in time because there's nothing happening.

## 2017-09-12
Found an algorithm for time-stepping that uses only iteration counts and
no fancy error calculations (used in SPICE2). See openoffice doc for more 
details.

## 2017-09-10
The unloaded diode bridge does find a DC solution, but if I load the output
with a resistor, there are NaNs all around the place.

It was having problems solving the current through the 0V source to ground.
I put in a hack to make the result 0.0 if the results isn't a finite number,
and things seem to work! [TAG 0.4.0]

### Next?
Where to I go next?

* DC Sweep
 - sweeping parameters
 - recording results
* Transient Analysis
 - [DONE] sinewave source
 - [DONE] sweeping time
 - [DONE] recording waveforms
 - L & C models mean numerical integration routines


## 2017-09-09
Got a simple diode-Isource-resistor circuit to converge by limiting
the voltage I calculate currents for in the diode model. I was quite happy
about this.

But then I tried to go for broke and find the DC solution for a diode bridge.
This did not work.

Liverpool got blown out today. Feck.


## 2017-09-08
I'm gonna stamp some diodes tonight.
Gonna stamp some diodes.
My moves are nonlinear, but that's ok.
Gonna stamp some diodes.

The other option is to stamp everything once. And update the parameters at each
iteration. This means keeping the twiddle values and their locations, but means
we don't have to copy/reallocate the huge matrix all the time. After all, the
companion models don't change.


## 2017-09-07
I hope to implement DC operating points with diodes in the circuit.

Things I have to solve:
* Generating parameters for a diode companion model
* Stamping some kind of base linear matrix with diode companion model
* [DONE] Updating the matrix after every iteration. Either
 * [DONE] keep a linear matrix base around and reuse
 * [NO] stamp and unstamp
* [DONE] Convergence testing: VNTOL, ABSTOL & RELTOL

Next: figure out how nonlinear devices live in an enumeration for linear
devices.


## 2017-09-06
Turns out that ODEs are not stored in the matrix:
1. The nonlinear circuit elements are linearised around an operating point 
   using Newton-Raphson.
2. Values are used to "stamp" the matrix with a linear companion model of the
   nonlinear element
3. Run Gaussian Elimination on this matrix to compute the unknowns
4. Look for convergence, where v(n+1) ~= v(n) and stop.

Companion model for a diode is a current source in parallel with a resistor.
And possibly a GMIN resistor too.


## 2017-09-05
I'm having trouble figuring out how I might handle ODEs in matrices, so the new
plan is:
1. Update circuit builder to MNA
2. Update Guassian Elimination algorithm

This actually was done. 

The next step is to read source code of open-source circuit simulators to see
how ODEs are handled.


## 2017-09-02
I've been using PDF on the internet that got the F'(V) for a diode wrong. After
fixing this, the DC operating point algorithm I have converges if the initial 
guess for the diode voltage is larger than what it should be. For lower initial
guesses, things fail.



## 2017-08-31

I can go a few ways now:
1. Update the Gaussian Elimination algorithm to match the better one on wikipedia
2. Update the circuit matrix builder to MNA
  - This will help handle V sources without circuit transformations
3. Start on nonlinear solver
4. Remove the [0] column and row from the matrix to save space. This means either
   changing the node index of GND to something other than 0, or littering the code
   with lots of `[i-1]`s.
5. Do LC transient analysis.

The nonlinear solver is the most interesting bit, I think. Although LC transient
is kinda interesting too...

Fixed back-substitution.


## 2017-08-30
Have a basic KCL solver for Is and Rs.

## 2017-08-29
What's the core of a SPICE engine?

Well, I need:

1. DC Operating Point
 a. Netlist representation
 b. Node equation builder
 c. Gaussian reduction algorithm
 d. Newton-Raphson for nonlinear equations

2. Transient Simulation
 a. 



