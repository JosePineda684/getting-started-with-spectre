# Visualizing Volume Data

Now, when we talk about volume data, what does that mean? Volume data holds
information about our entire domain and quantities we choose to keep track of at
a specific time in the simulation. This data takes up a lot of space, so we try
to only output when we need it. Think of it like we're taking a picture of our
domain every once and a while, and after a lot of simulation time, we have a
scrapbook full of pictures of our domain as it has evolved in time.

So, how do we visualize volume data? Well, we use an application called
**ParaView**. Some people really have a problem with ParaView, and it definitely
does have a bit of a learning curve, but overall I think it's pretty not
terrible to work with. First, you'll need to
[download ParaView](https://www.paraview.org/download/). The version you pick
does matter if you're going to be connecting your local ParaView to a cluster
(see [connecting_to_clusters](#connecting-paraview-to-a-cluster)). However, if
you don't care about that, then just download one of the latest versions.

## Making ParaView Readable Files

Now that you have ParaView, we need to convert the data into a format that
ParaView can read. Luckily we have a tool for this in the cli. To start, go into
the Segment directory that contains the data you're trying to visualize. Then,
run this command `spectre generate-xdmf -h` (if it doesn't work, make sure
you've sourced your modules or run the activate script and if you have no idea
what I'm talking about go up to the
[Building SpECTRE section](../001_BuildSpECTRE/)). This should give you some
help text as to how to get started. The main thing is this command will generate
an xmf file that ParaView can read. Here's an example of me using the script and
we'll walk through each option.

## MISSING FIGURE

To start, you'll notice that there's no flag before I put `BbhVolume*.h5`, you
just need to point it to whatever file you want to visualize in ParaView. For
volume data, you want to make sure you get all of the Volume files, that's
why there's the * before .h5. \\

The `-o` we've seen before, this just names the output file and adds the .xmf
extension to it.

Last is the `-d` which is the subfile name. This depends on the file you're
using. A nice trick is if you do something like `-d garbage` then it'll output
all the available subfiles (unless of course it for some reason has a subfile
named garbage). If you don't do this and instead you pass something like
`BbhVolume0.h5` then only a part of the domain will show up in ParaView. This is
because we split up and assign parts of the domain to different nodes or parts
of nodes.

Now this command might take a while because it's trying to get all the data into
a readable format for ParaView. Once it's finished, you should see a file with
the name you specified for the `-o` option with .xmf extension on the end.

We can do the same thing for the `BbhSurfaces.h5`, because those files contain
the data for visualizing the apparent horizons of the black holes. One way you
can think about volume vs. surfaces is that volume data holds information about
the domain/grid of the space we simulate, where as surfaces are the black holes
themselves (at least what we can know about it). Start by running this command:

```
spectre generate-xdmf BbhSurfaces.h5 -d garbo
```

This should error and tell you which subfiles are available. Now, use this
command to make an xmf file for object A's apparent horizon.

```
spectre generate-xdmf BbhSurfaces.h5 -d ObservationAhA -o AhA
```

You should now see an `AhA.xmf` file in your current directory if that command
completed successfully. Now, alter the command above to give you an xmf file for
object B.

**NOTE:**

If you would like one xmf file that has data from each segment, you can do that
by altering the commands we tried above. If you're in a directory containing all
of the segments from the Inspiral, you can run this command:

```
spectre generate-xdmf Segment_*/BbhSurfaces.h5 -o AhA
```

Once you have 3 xmf files, one for the volume data, and 2 for each apparent
horizon, we're ready to start visualizing. (You also don't need all 3, you could
just decide to visualize the apparent horizons or the volume data).

There are a few ways to go about visualizing the volume data the simulations
generate. Like I mentioned earlier, volume data takes up a lot of disk space,
and by a lot I mean it can have hundreds of gbs of data for a single simulation.
If you're going to try to visualize that on your local computer, you'll have to
download all of that data, which is not ideal. ParaView also uses a lot of ram
to render volume data, so most laptops or pcs are not equip for this. So
instead, we can connect our local ParaView to a cluster which lets ParaView use
the supercomputer's resources to render all of the data, which it can do easily.
Only a few clusters have the ability to do this right now, Ocean2, Mbot which is
a supercomputer at Cornell hosted by Nils Deppe, and CaltechHpc which is a
supercomputer at Caltech hosted also by Caltech. If you would like to setup an
account on these clusters check out the next section of the guide
[connecting_to_clusters](#connecting-paraview-to-a-cluster).

## Connecting ParaView to a Cluster

So, here's where I mentioned the ParaView version matters. If you're connected
to a cluster and have sourced the modules, you can run the command
`module list`. This should show you all of the modules that are currently
loaded. If you're on Ocean2, you'll notice we don't have a version of Paraview
loaded at the moment. To load a version of Paraview, let's first see what
versions are available to load. Run the command `module spider`, scroll through
the long list it outputs until you find Paraview. You should see something like:

## MISSING FIGURE

To load this module, run the command `module load paraview/5.13.3`, once that's
done, you need to make sure you've downloaded that exact version of Paraview on
your local machine or else you won't be able to connect to the clusters
paraview. If you're on Mbot, you should see Paraview/5.11.1 loaded for you.
Clusters sometimes have multiple versions of ParaView on them. To see all of the
versions, run the command `module spider`. If you find a ParaView version that
you have, you'll have to unload the currently loaded ParaView module by doing
`module unload name_of_module_you_dont_want` and then load the version you want
by doing`module load name_of_module_you_want`.

Now, once you have the same version of ParaView as the one on the cluster, we're
ready to connect. Before we do there are some good practices we can employ to
make sure that we don't accidentally try to connect to the same ParaView session
as another person. For CSUF students, we'll be using the last 4 digits of our
campus wide ID when running the command below. The last 4 digits of my campus
wide ID are 1572 so the command I would run is `pvserver -p 11572` which should
output something like this:

## MISSING FIGURE

This ensures we won't run into the problem of trying to use the same port
number. Each cluster has some rules that you may want to follow when using this
command so as to not run into people requesting the same port number. Mbot has
it's own convention that can be found
[here](https://github.com/sxs-collaboration/WelcomeToSXS/wiki/Mbot).

Once you've run `pvserver -p YOUR_PORT_NUMBER` now you can go to your local
terminal and run a command like this: 

```
ssh -L11111:ocean2:YOUR_PORT_NUMBER YOUR_USERNAME@ocean2.fullerton.edu
```

The 11111 at the start should not be edited. Now, once you run the command
above, you'll notice that you're now connected to the cluster! From here, open
ParaView on your local machine and click on this icon

## MISSING FIGURE

Here, you'll need to click on the add server and you can edit the name, but you
don't have to, just click configure. Then once you have that, click on the
configuration and hit connect. This might take a second for it to connect, but
once it finishes, you should see this: 

## MISSING FIGURE

I named my configuration mbot, which is why that name shows up there. From here,
we can actually start to visualize our data!

## Basics of Using ParaView

To start let's load the data you made with `spectre generate-xdmf`, to do that,
click on the file icon with an up arrow in the top left of paraview. Now
navigate to where you made the xmf file and double click on it or click okay in
the bottom after clicking on. You should notice that the file you loaded is now
in your Pipeline Browser! Now click the apply button below the Pipeline Browser
and after it loads you should see this!

## MISSING FIGURE

This is THE gumball where all the computation takes place. Maybe better put,
this is the domain where we're telling the computer to simulate the black
holes/space around the black holes.

Now at first this doesn't look like much, so let's slice this gumball open.
Click on this icon while you have your xmf file selected (your xmf file should
be highlighted if you've selected it).

## MISSING FIGURE

This should add a new object to your pipeline browser called Slice1, if you
look below the pipeline browser at the properties, we'll want to do a few things
before clicking apply. First, let's slice it along the z-axis, then uncheck the
show plane option, and finally uncheck the triangulate the slice option. Once
you click apply, your gumball should be a tiny slice along the z-axis like so: 

## MISSING FIGURE

Now zoom in towards the center of the slice and you should see two circular
cutouts. This is where we've cut out the parts of our domain that contain the
singularities from the black holes! Often referred to as the excised regions.
