# Building SpECTRE and Simulating Black Holes

Now that we've cloned SpECTRE, and set up our environment, now it's time to
build all the executables you'll need to start simulating things!

## Where, How, and What to Build
To start, we'll talk a tiny bit about executables. Executables are the product
that comes from compiling code. It's the program that's going to simulate the
black holes. There are a few executables that we'll need to build to get you
started simulating things.

Before that though, we need to make a build directory and get that set up. `cd`
into your spectre directory and do `mkdir build` and cd into the build
directory. Inside the build directory, run the command `spectre_run_cmake`
(this command sometimes has `_gcc` or `_clang` at the end of it on certain
clusters). *IMPORTANT* Only run `spectre_run_cmake` in a build directory or else
you'll have fun cleaning up the mess it makes in whatever directory you ran it
in. If that command is not found, make sure you set up your environment
correctly (most of the time you just need to `. ~/activate.sh). Make sure that
this command completes without any errors! If there are CMake Warnings that's
okay. Now, if you `ls` you'll notice this directory has a lot of stuff in it
now! Running `spectre_run_cmake` sets up your build directory so that you can
make executables.

Speaking of making executables, let's try that out! There are a few executables
you need to start simulating black holes. The first is SolveXcts, which
generates the initial data for whatever you are trying to simulate. To build it,
make sure you're in your build directory and run the command
`make -j4 SolveXcts`. The -j4 splits the building among 4 processors which
significantly speeds up this process. Now this will take a while, this can take
somewhere around 30 minutes on Ocean1. This also requires your computer to be
awake and connected to the cluster the entire  time. However, if it gets
interrupted that's okay, it saves the progress. Once the executable is finished
it'll say something like `built target SolveXcts`. After this, we'll want to
build the command line interaface (cli). This executable unlocks a bunch of cool
features that you can run right in the terminal. Most importantly, the bbh
``pipeline'', referring to starting a simulation by generating initial data,
simulating inspiral, transitioning and simulating ringdown, and extracting the
waveforms. More details on doing so below. Now, run the command `make -j4 cli`.
Then once that finishes, we'll run the command
`make -j4 EvolveGhBinaryBlackHole`. This is the executable in charge of
simulating the two (binary) black holes. This one will also take quite a while.
Now that executable is for binary black holes, and so we need to build the last
executable for a single, remanent black hole. I'll give you a moment to guess
the command for the single black hole executable. If you thought to run the
command `make -j4 EvolveGhSingleBlackHole` great job. Once all of these finish,
a good sanity check is to cd into the bin directory that's in your build
directory and `ls`. You should see all of the executables you built and perhaps
a bunch of extra stuff you don't recognize. If you see all the executables that
you built, then you are ready to start simulating some black holes!

## What is a full simulation of a binary?

A full simulation simulates the Inspiral, Merger and Ringdown portions of a
binary system. The inspiral portion is when the 2 black holes are circling
around each other, at first, they slowly move closer together, but as their
separation decreases, they orbit around each other faster and faster emitting
more gravitaional waves which causes them crash into other at very high speeds.
Once they get very close to each other, the merger phase starts, this is the
most violent and difficult part of the simulation where the two black holes will
crash into each other and combine into a single black hole. Once they combine
into a single black hole, that's when the ringdown phase starts. The reason it's
called the ringdown phase is because a single black hole that's perturbed is
analagous to a bell being struck and ringing, slowly decaying energy away until
it relaxes and no longer rings.
 
So what do we need for for a full inpsiral, merger and ringdown simulation? The
first thing you need is InitialData. Initial data serves as the initial
conditions for a simulation. The SolveXcts executable is used to generate the
initial data. We then evolve that initial data using the EvolveGhBinaryBlackHole
executable. This simulates the inspiral and merger portions of the full
simulation we're after. Once they've combined into a single black hole and we
have enough data, the simulation of the binary is stopped. From here, the data
from the combined black hole is evolved using the EvolveGhSingleBlackHole
executable. This will simulate the ringdown portion thus completing our full
inspiral, merger, and ringdown, sometimes referred to as IMR. Now that we have a
bit of background, we're ready to start simulating some black holes!

