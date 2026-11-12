# Starting Simulations with the cli

The cli stands for command line interface, it's also very commonly called the
pipeline, or bbh pipeline. Referring to the ``pipeline'' is the practice of
specifying some initial parameters, like mass, spin, orbital separation, and the
pipeline will take it through a full simulation. That is, generate initial data,
evolve inspiral, transition and evolve through ringdown and extract the
waveforms from your initial parameters. As of writing this, the pipeline is
still very much a work in progress.

SpECTRE has some very helpful tools to use that streamline the process of
simulating black holes. Before we use any of the commands we'll talk about
below, let's check to make sure everything is working. In your terminal, run the
command `spectre -h` this should bring up quite a bit of help text, showing you
all the great commands at your disposal. Using the cli works kind of like
traversing a directory. Since we want to simulate binaries, we'll want to look
at the commands related to running binary black holes, so now try to run the
command `spectre bbh -h`. This should bring up all of the commands you'll need
to run a full simulation. If these commands don't work once again it's probably
because you haven't sourced your modules (use `. ~/activate.sh`). If they still
don't work, you probably have not set your BIN path correctly in the
`activate.sh` script, make sure it's the full path going to the bin directory in
your build in spectre. Now, if all of these commands work, we can start
generating initial data!

## Generating Initial Data

To generate initial data, we'll use the cli/pipeline. When you ran the
`spectre bbh -h` command, you may have noticed the `generate-id` command. Run
`spectre bbh generate-id -h` and this should show a bunch of help text for how
to start generating initial data. I encourage you to read through each option
you can specify. In all of these commands, the help text on the right will
indicate whether or not an option is required, though some are a bit
complicated. Below is an example of me using the command line to start
generating initial data for an inspiral.

```
spectre bbh generate-id -q 1 --chi-A 0 0 0 --chi-B 0 0 0 -D 15 -e 0 -N 1 
--lev 1 -o EM_ID
```

Let's walk through all the options I specified.

The `-q` like the help text says refers to the mass ratio of the system. In the
numerical relativity business, our convention is $q \geq 1$. Mass ratio 1 means
that the black holes have the same mass, mass ratio 2 means that one black hole
is 2 times more massive than the other one. 

The next options `--chi-A 0 0 0 --chi-B 0 0 0` refer to the spins of the black
holes. Each number represents the spin on the x, y, and z axis respectively.
Here, I've chosen to not give either black hole any spin.

The next option `-D 15` refers to how far apart the black holes start in our
code units. For most of our runs, we start them around 15M or so apart (M is our
code units where $G=c=1$).

The next option `-e 0` is our target eccentricity for the initial data.
Eccentricity tells us about how circular our orbit is. An eccentricity of 0
means we have are orbiting in a perfect circle. As your eccentricity goes up
your orbit becomes more elliptical. It's important to know that if you're going
to specify `-e 0`, then this means you only have to specify one of these 3
options, `-D`, `-w`, or `-a`. The reason you only have to specify one of them is
because there are equations one can solve for a circular orbit where if you know
either their separation, orbital angular velocity, or radial expansion velocity,
you can determine the other 2 options. The way SpECTRE does this currently is by
using our old code SpEC to determine those other 2 missing options for you if
you request a circular orbit. This turns out to be why we needed to add
`SPEC_ROOT` to our activate.sh script!

Anywho, enough tangents, the next option `-N 1` refers to how many nodes we're
requesting for this job. Since this example was pulled from me using Ocean2, I
only requested one node, however, depending on what cluster you're on, you
might request more.

The next option is `--lev 1` this refers to the resolution we're setting the
simulation to. A higher lev means that your simulation will be more accurate
(in most cases :P ). Specifically, what the lev does at the time of me writing
this is it changes the `-L` and `-P` refinement. Below is a more detailed
discussion of these options. In the future though, `--lev` will still choose a
`-L` and `-P`, but more what this is doing is setting tolerances for what we
call **amr** which stands for adapative mesh refinement, this changes the
resolution dynamically during the simulation based on if the simulation needs
more resolution or less resolution to reach a desired accuracy.

The last option `-o EM_ID` refers to the directory that you would like the data
from the run to output to. Also important to know, unless you specify a full
path for `-o`, it'll make a new directory inside the directory you're currently
in. Please please please please please don't put simulations inside your spectre
directory. Also a good convention is to give it a descriptive name, so not
something like `-o run` cause you'll look back at that and be like wow I have no
idea what this is or who gave it this horrible name! So my naming convention
usually is centered around its mass ratio followed by if it's initial data then
ID, or if it's evolving initial data, then Inspiral or something along those
lines.

Now what if, for instance, I didn't want to use `-e 0`? A common example of this
is what if I wanted to do a headon simulation? A headon collision occurs when
you don't give the black holes any orbital angular velocity. An example of me
generating initial data for a headon collision is below.

```
spectre bbh generate-id -q 1 --chi-A 0 0 0 --chi-B 0 0 0 -D 15 -w 0 -a 0 -N 1 
--lev 1 -o Headon_ID
```

Since the other options were discussed above, I'll only be discussing `-w` and
`-a` here.

The option `-w 0` is the orbital angular velocity I'm saying my binary has.
Since I gave it 0, this means that the black holes won't orbit each other at
all, they'll just slowly crash into each other in a straight line (unless
they're spinning of course).

The next option `-a 0` tells us how quickly the black holes moving falling
towards each other. Since we put 0 here and 0 above, essentially we're saying
our black holes start at rest.

Now in practice, variations of these commands are pretty much all you'll need to
start simulating black holes. However, there are a few important options we left
unspecified that you should know about.

The `-L` option refers to the refinement level you're requesting. This is a way
to increase the resolution or accuracy of your data. The best way to think of
refinement in spectre is by visualizing a cube. If you have 0 refinement, then
you're left with a single unaltered cube. If you refine it once along the x,
this cuts the cube in half along the y-z plane and now you have 2 cubes. If you
then refine along the y and z, it cuts it in half along the x-z and x-y planes
respectively and now you have 8 total cubes. This increases the resolution of
the initial data by simulating a smaller region, so it captures smaller effects.

The `-P` option refers to what order polynomial you're requesting. This is
another way to increases the resolution or accuracy of your data. A way to think
about this is by plotting a function. Since we use computers, we have finite
resources to represent a function, so the way we represent a function is to pick
a few points that capture some of the shape and features of our function. The 
P-refinement refers to how many points you're using to specify functions. The
more you use, the better you capture the underlying solution (if you're within
the convergent regime of course). SpECTRE uses the legendre polynomials to place
grid points at specific places in our domain (these specific places happen to be
called collocation points in case you're curious).

There are other options that change this command a bit, but what I've outlined
are some of the more important ones.

The `-J` option allows to put a silly name for the job. This will appear for
everyone if you type `squeue` so yeah.

Now, let's actually run one of those commands above! When you do, it should
start creating a directory, copying the executable and finally ask you if you'd
like to submit the job. Type `y` or `Y` or `yes` or okay you get the point and
your run will submit and start generating initial data! If this is your first
time, congratulations, it took a lot of set up to get here, so pat yourself on
the back! Now, there are a few ways to check on your run. The first thing to do
to see if your run is in fact running is to run the command `squeue`. This
brings up all the jobs running on the cluster you're on! You should see
something like

# MISSING FIGURE

If under ST it says PD this means you're in line, so be patient! Also be careful
to not request more nodes than are available on your cluster. The way to check
how many nodes there are is with the command `sinfo`. This will show you the
queues, how many nodes are in use and how many are idle. If you request more
nodes than the cluster has, your job will in fact sit there. Forever.

Anywho, now that you have a job running, that's great, initial data usually
takes around 30-60 minutes to complete. The way to know it's complete is if you
`squeue` and you don't see your job anymore, then `cd` into the directory you
specified in the `-o` option and then `ls` you should see a few ControlParams
directories. The least I've seen is 3 and the most I've seen is 7. If you see
this then congrats, you have initial data! Now it's time to evolve that initial
data.

## Evolving Initial Data

Initial data serves as the starting point for our evolutions! The way we solve
for the next time is we look at our current conditions, we then solve for their
time and spatial derivatives to see how it's changing over time. We then use
those time and spatial derivatives to update our current conditions forward in
time. This then serves as our current conditions and we solve for time and
spatial derivatives and the cycle repeats! So how do we get it to start doing
this with initial data? Once again, we'll use the cli/pipeline.

First, let's try running the command `spectre bbh start-inspiral -h` this should
print out a bunch of help text again (if it doesn't, remember to source
modules). Once again, I'm going to give an example of me using the command and
then we'll walk through all the options.

```
spectre bbh start-inspiral EM_ID/ControlParams_00Last_One/InitialData.yaml 
-O EM_Inspiral --lev 1 -N 1
```

So the first thing to notice is that after `start-inspiral` we don't put an
option before giving a path to the InitialData.yaml file. There are lots of
commands that expect a file or path with no option. What you input here varies
depending on how many `ControlParams_` there are in your initial data directory.
Point it to the last ControlParams, usually it's 007, but I've seen 003 a few
times as well. The reason we point it to the last one is because that has the
most updated parameters.

The `-O` option is like the `-o` option we used in the InitialData generation.
This makes a directory where your data will be stored. The only difference is
with `-O` it makes a directory inside the directory you specify called
`Segment_0000` and then it makes more if your run checkpoints.

The `-N` option is the same as the initial data, this is how many nodes you're
requesting. 

There are a few other options that you can change, it has the same `-L` and `-P`
for simulation resolution. See the discussion of those in the Initial Data
section.

Now, just like before, let's run the command! After you submit the job, it'll
most like take 2-3 days if you're simulating an inspiral. If you're simulating a
headon collision, then it'll likely take less than a day. To check on your run,
you'll want to `cd` into the Segment directory inside the directory you
specified with `-O`. Inside each Segment directory there is a `spectre.out`
file. Open that file and there it'll tell you things like what time the
simulation has gotten to and horizon finds. We'll talk more about what this
means later.

The way to know if your simulation has completed and found a common horizon is
to open the `spectre.out` file in the last Segment directory it created and
search for ObservationAhC (you can do a search with control+f or command+f). We
call it a common horizon because it's a horizon that fits encapsulates the 2
original black holes. This means they've gotten close enough for us to consider
thinking about evolving it as one black hole instead of 2. A successful
ObservationAhC will look something like.

# MISSING FIGURE

If the run error'd, but has found ObservationAhC's then that's okay. If it
error'd before finding a common horizon, then the run has failed and we can't
continue evolving the single black hole. There will be a section later on
talking about what the common errors are, why they happen, and potential fixes!
However, if you have common horizon finds, then congratulations, your black
holes have merged! The next simulation we can do once we have data containing
common horizon finds is to evolve the common horizon letting the final single
black hole ring down.

## Ringing down a common horizon

The script we use to transition from simulating 2 black holes to a single black
hole is quite involved. This is the most dynamic part of the simulation where
we're trying to let the black holes get as close as possible without the
constraints (simulation error) getting too high. This is because it's much
easier to ringdown a black hole that is more spherical. We'll look at what the
common horizon looks like in a later section, but the most important things to
note is that we'd ideally want at least 8 successful common horizon finds before
the simulation using our binary domain finishes. To ensure this, you can go back
to the previous section and check the `spectre.out` in the last Segment
directory and search for `ObservationAhC` and make sure there are at least 8.
Now once you're sure you've got all the common horizons you'll need we can run
the command below to start ringing it down! 

# MISSING FIGURE

The script starts similarly to the evolution script, but here instead of the a
file, we'll point it to the last Segment directory it made in your evolution
directory (please do not just put `Path/To/Last/Segment`). Then, finally, the
usual `--lev`, `-O`, and `-N` options. For the ringdown, if you get a memory
error, then you should increase the number of nodes you're using and try again.
This should be all you need as the script has been updated to determine all of
the extra options below on its own. However, it's good to know what these
options do if you ringdown doesn't work and you need to troubleshoot or want to
try a few things out.

The `--path-to-output-h5` option requires a bit of context. During the binary
evolution, we have 2 excisions, I know I haven't really talked about excisions
yet, but I'll give you the rundown on them now! Black holes have singularities
inside them. Singularities are not very good to simulate because they contain
things like infinities which computers aren't very good at handling. The way
that SpECTRE deals with this singularity problem is by cutting out a portion of
our domain that contains the singularity. This is what we refer to as the
excision, the part that's cutout that contains the singularity. Now, during the
binary evolution, these excisions need to follow the black holes and match their
shape, size, and position so that the singularity never enters our domain. We
have these things called functions of time that track all of these things,
however during the binary evolution, we're only tracking these quantities for
the 2 excisions. When we switch from the binary evolution to ringdown, we now
need to have functions of time for a single excision that'll track the shape,
size, and position of the common horizon. The start-ringdown script solves for
the functions of time needed to track the shape, size, and position and the
shape information gets stored in the file you specify in this option. Phew,
that was a lot for this one option.

The  `--number-of-ahc-finds-for-fit` option refers to how many common horizon
finds you'd like to use to make the functions of time for the shape and
translation functions of time in the ringdown. Usually around 10-20 is
sufficient, this option depends on the match time you choose, so be careful. If
you choose a match time that's one of the earliest common horizon finds, you'll
want to give this option the max amount of successful common horizon finds. This
is definitely an oversight and should be changed in the future to be the amount
of horizon finds before the match time.

The `--match-time` option is what time you'd like to start the ringdown
evolution at. This should be one of the times it found a common horizon at. This
option varies greatly between different evolutions. I suggest using one of the
last common horizon find times. To find this time, go into the `spectre.out`
file in the last segment directory of the binary evolution. There, you can
search for \textbf{ObservationAhC} and look at one of the last times it had a
successful find. From there, make sure that it has found at the very least the
number of ahc finds for fit you specified in the option above. However, that
option looks from the very last common horizon find time, not the match time you
specify. So for example, if my last common horizon find is at t=4915.20 (and it
should find a horizon every 0.01) and I choose a match time of 4915.15 and I
want at least 10 points of data to be used to make the functions of time, I
should specify 15 for the number of ahc finds for fit (and I should ensure that
I have horizon finds going back to 4915.05). This should've gone into the
section for that option, but I think it works here too.

The `--settling-timescale` this is the settling time for certain functions of
time like the expansion map. You don't need to worry about this, just set it to
10.0.

Now, once you submit the job, it'll take somewhere between 3-10 minutes to
interpolate all of the grid points from the binary domain to the single black
hole domain. Once it does this it'll start the simulation at the match time you
specified. It usually starts up just fine, but after that there's no guarantees.
It starts off very very very slow while it settles into this new domain trying
to let the black hole ring down. It usually fails of incoming char speeds within
the first 2M it evolves after the match time. I haven't found a good way to
combat this. However, if it makes it further than 2M past the match time, then
it might just work, but take that with a grain of salt. It's important to know
that the file this script generates to evolve might output a lot of volume data
which takes up a lot of space. I need to make a section outlining some tips on
how to change the input files these scripts generate.

## Monitoring your simulations

On some clusters (sadly neither Ocean) you can use the command
`spectre status -w 3` which will tell you a bit about your runs, how long
they've been going for, how far apart your black holes are, the constraints and
other useful things. The `-w 3` tells it to refresh this information every 3
seconds, if you don't want that and just a snapshot of how the runs are doing,
just run `spectre status`.

The other thing to look at for your runs are the `spectre.out` files. This is
where information on your runs is printed, it has diagnostics printed to it
throughout the simulations. For initial data, this file is in the directory you
specified with the `-o` option. For the inspiral/ringdown, there's a
`spectre.out` file in each Segment directory that gets made in the directory you
specified with `-o`. So once you cd into a directory with a `spectre.out` file
and run `code spectre.out`, then you can see how your run is doing. 