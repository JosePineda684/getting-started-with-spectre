# Visualizing Reduction Data

Visualizing data is an incredibly important step when doing any sort of
research. This is the main way you'll convey your results and give not only
yourself, but others a chance to gather insights about your research. In
SpECTRE, there are a few different types of data to visualize. If you've gone
through the other sections leading up to this and have at least some of an
inspiral completed then we can start thinking about visualizing the data it
generated. In your inspiral run, you should see Segments folders, if you `cd`
into one of them, you should see files like `BbhSurfaces.h5`, `BbhVolume0.h5`,
`BbhReductions.h5`, and `BondiSachsCceR0150.h5`. All of these files contain
different data from the simulation and we have a bunch of different tools in the
cli that can help you visualize them. 

## Visualizing Reduction Data

So you may be asking yourself what is reduction data? In every Segment
directory, there's a `BbhReductions.h5` file that contains information about
specific quantities we're interested in plotting. It contains information on the
trajectory of the black holes, the error throughout the whole simulation,
control system diagnostics, power monitors and many other useful things! For all
of these, we'll be using the cli, so start by making sure this command runs
without error `spectre plot -h` this should show you the many plotting tools we
have available. If it ran fine, we're almost ready to start plotting things from
the `BbhReductions.h5` file.

Before we get started plotting, it'd be good to combine all of the
`BbhReductions.h5` files so that we can plot all of the data. Steps on how to do
that can be found in [Section 9](../009_CombineFiles/).

## Plotting the Trajectory

We'll start by plotting the trajectory of the black holes cause this is probably
the coolest thing to plot. Above you ran `spectre plot -h`, now try
`spectre plot trajectories -h` this plotting tool is quite easy to use, all you
have to do is run this command 
`spectre plot trajectories BbhReductions.h5 -o Trajectories.pdf` notice how
there's no flag before `BbhReductions.h5`, also if you don't like pdf, you can
change it to png, or jpeg (but nobody likes jpeg). If that ran no problem, then
you should see a new file called `Trajecories.pdf` in your current directory and
it should look something like

# MISSING FIGURE
Now this is a great tool to get a good look at what's going on, but if you
wanted to make some plots on your own for something like a conference or
individual plots, we'll look at how to do that in **ADD SECTION**

## Plotting the Constraints

Plotting the constraints is really important, it gives us a view of how accurate
our simulation is overall. The lower the constraints are, the closer we are to
perfectly simulating the physics of the system. There are many constraints that
you can plot, but the main ones that we like to look at are the **Gauge**,
**Three Index**, and **Constraint Energy**. If you're confused about what
constraints are, a good analogy are Maxwell's equations, if you have already
taken an introductory course in electricity and magnetism. When in vacuum,
**$\vec{\nabla} \cdot \vec{E}=0$** and **$\vec{\nabla} \cdot \vec{B}=0$**. This
is true analytically, but when you solve for electric and magnetic fields
numerically, these equations won't be perfectly true. These constraint equations
tell us that there are no charges and we don't have magnetic monopoles. They
give us a way to check to make sure that what we're simulating is not deviating
from in this case Maxwell's equations, but for SpECTRE's case Einstein's
equations. So, when we're plotting our constraints, we're looking for them to be
as close to 0 as possible.

So to start plotting, let's try running this command:

```
spectre plot dat -h
```

If this ran without error, you should see many options to choose from to
**'Plot columns in '.dat' datasets in H5 files'**. Now there's a lot that we can
plot with this script and it might seem a bit overwhelming at first, so let's
walk through only the most important options you need to be able to get a plot.

The first option is `-d` which is the dat subfile name that contains the data
you'd like to plot. Now you may be asking yourself at this point "What is the
right subfile?" And that is a great question. If you weren't overwhelmed before,
try looking at all your options by running this command

```
spectre plot dat BbhReductions.h5 -d Garbo
```

We'll focus on the `Norms` subfile to plot constraints. Now, if you ever want to
know what you can plot try adding `-l` to the end of your command. So try
something like:

```
spectre plot dat BbhReductions.h5 -d Norms -l
```

And this should tell you all of the quantities you have available to plot. If
you don't want the list to output, just delete `-l` from your command.

I usually run the command above before I write the next option because it can be
a hassle to write out the full name of the constraints. The next option is `-y`
this is how you tell it what to plot on the y-axis.

The next important option is `--y-logscale` this as the name implies tells it to
make the y-axis log scaled. When plotting any constraint, use a log scale for
the y-axis, it makes it much easier to read/understand.

And of course, the last option is `-o` to name the plot that this script will
output. You can add extension like .pdf or .png to your name and it'll make it
that file type.

Now, there are other options like what to put on the x-axis but it defaults to
time which is what we want 99% of the time. There's also axis titles if you're
into that sort of thing. But anywho, putting all of this together, let's plot
the **Three Index Constraint**. Try running a command like the one below:

```
spectre plot dat BbhReductions.h5 -d Norms -y 'L2Norm(PointwiseL2Norm(ThreeIndexConstraint))' --y-logscale -o Three_Index.pdf
```

Notice how I put single quotes around
`L2Norm(PointwiseL2Norm(ThreeIndexConstraint))`, you'll get an error if you
don't do this. Now, if that worked you should see something along the lines of
this graph!

# MISSING FIGURE

Errors somewhere in the range of 1e-5 to 1e-7 is pretty typical when the
constraints go flat (in my graph around time 900M or so until time 4800M or so),
which means this simulation is doing well and the constraints error is not high.
If your graph looks pretty similar to mine, I enourage you to plot other
quantities to see what they typically look like and to get a feel for what to
expect to see in these simulations!