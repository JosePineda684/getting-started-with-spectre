# Getting onto a cluster (Supercomputer)

Since binary black hole simulations are very computationally expensive, this
means that we have to use supercomputers for them to finish in a reasonable
amount of time.

The first step to getting onto a cluster is to generate an SSH key. This can be
done in your terminal by following the documentation [here](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent).

Once you've generated an SSH key, check in your .ssh folder (usually in your
home directory, if you can't find it try `ls -a` in your terminal) for 
`id_25519` and `id_25519.pub` (or if you generated an rsa key look for `id_rsa`
and `id_rsa.pub`). You'll need to send Dr. Lovelace the public key so that you
can connect to the supercomputer. You can do this a few different ways, but the
easiest is if you've cd' into the .ssh folder, do `cat id_25519.pub` then copy
everything it printed out and send that to Dr. Lovelace.

Now once an account has been created for you, you can connect to one of the
clusters, most likely Ocean2, or the older Ocean. To connect from your terminal
try `ssh your_username@ocean.fullerton.edu` for Ocean or
`ssh your_username@ocean2.fullerton.edu` for Ocean2 (please don't literally type
`your_username` unless your username on that cluster is in fact `your_username`
if you're confused about what your username is ask Dr. Lovelace). This process
is the same for pretty much every other cluster as well. Once you try to
connect, you might be prompted with a password and this should be the password
you used when generating the SSH key.

If you successfully logged in, then congratulations, you have now connected to a
supercomputer!

## Setting up Visual Studio Code

Now, using the base terminal is great and all, but there are better tools we can
use. I personally use vscode because it offers very useful extensions that help
you connect to clusters easier, find files faster, and keep track of/commit code
changes easier.

If you are using Ocean2, then you are free to use whichever version of VSCode
you like. Using VSCode with Ocean is a bit of a pain however, because Ocean (in
its current state) has an outdated version of linux installed that newer
versions of vscode don't support. The workaround for this is to download an
older version of vscode. The version most of us in the group use can be
downloaded [here](https://code.visualstudio.com/updates/v1_85).

Once installed, we need to download a few extensions, this can be done by
clicking on this icon on the left of the screen.

## Missing Figure

You can install any/all extensions your heart desires, but I'll leave my list of
must haves. Remote Explorer, Remote - SSH, GitLens - Git Supercharged, clangd,
H5Web, Jupyter, Python, and PDF Viewer. Once you've installed all of these
extensions, you should have a few new things appear on the left side of your
screen, the first we'll look at and set up is the Remote Explorer.

## Missing Figure

Click on the icon above and then hover over the SSH and click on the plus sign.

## Missing Figure

This should prompt you to try to log in to a cluster. Log in the same way you
did in section 1.1. It'll ask you for a platform, and you can choose Linux. Now
it should connect you to the cluster and in the bottom left of vscode you should
see (or the name of whichever cluster you connected to)

## Missing figure

Now, anytime you want to connect to the cluster, you can open the Remote
Explorer tab and hover over the name of the cluster and click the right arrow or
new window options. We'll do more with this extension later.

Now that vscode is setup, and you have some good extensions installed, it's time
to set up SpECTRE!

## Cloning SpECTRE

To start, you'll need to make a github account. If you've done this then you can
find the [SpECTRE repository](https://github.com/sxs-collaboration/spectre).
Now, find the tab that says fork and click on that and create your fork of
SpECTRE.

Before we go back to a cluster, we're going to add your public key to github.
This'll allow you to clone your fork of SpECTRE to a cluster and eventually push
changes you make to the code. To add your public key to github, go to
**settings** and then to **SSH and GPG keys**. There, you'll add a new key and
paste the same public key you generated and sent to Dr. Lovelace.

Now that you have that, you're ready to clone your repository to the cluster of
your choice. Go to your fork of SpECTRE and find the tab that says code and copy
the link in the SSH tab.

## Missing Figure

Now go back to your terminal that's connected to a cluster and type `git clone`
followed by the link you just copied. If this command says permission denied,
then there's something wrong with your SSH key connection. There are a few
things it could be, either haven't added your SSH key to github, or we have to
enable ForwardAgent in the config file, or alternatively, we can add the public
key on your cluster account to your github. I'm going to opt for adding
ForwardAgent to the config file. This'll allow the cluster you're on to use your
key you used to connect to the cluster for permissions like cloning a repository
\and what not. So we'll start by clicking on the Remote Explorer extension.
Then, hover over SSH and click on the cog wheel. This will ask if you want to
open one of your config files. Choose the one in your `.ssh` folder. In your
config file, you should see one or more Hosts, followed by Hostname and user.
Add the following lines into your config file:

```
Host *
  ForwardAgent yes
```

Now, you'll want to disconnect and reconnect back to the cluster you're working
on and try cloning the repository again. This time it should work!

If it cloned successfully, you should be able to type `ls` and see a spectre
directory.

## Setting up your Environment

Now that SpECTRE is cloned, we'll need to set up your environment so that it's
ready to build everything you'll need to simulate black holes or develop code.
Every time you open a new terminal or connect to a cluster, you'll need to do
what we call sourcing your environment and loading modules. Since this has to be
done every time you open a new terminal or connect to a cluster, we will want to
streamline this process.

The process is as follows. Depending on your cluster, you'll want to do
`export SPECTRE_HOME=/full/path/to/your/spectre` (once again don't literally
write `/full/path/to/your/spectre` but replace it with the full path to where
your spectre directory lives on your cluster). Then
`source $SPECTRE_HOME/support/Environments/ocean2.sh` depending on your
cluster, you'll want to change `ocean2.sh` to another clusters environment file.
Then, **IMPORTANT**, if you're on Ocean1, do 
`export SPEC_ROOT=/opt/ohpc/pub/apps/spec/spec-interp4/`.  
If you're on Ocean2 copy this line exactly as it is, don't change it to your
name: `export SPEC_ROOT=/home/alexcarpenter46/code_spec/spec`. Then, once you've
done that, do `spectre_load_modules`, then `export BIN=$SPECTRE_HOME/build/bin`,
then `export PATH=$BIN:$PATH`. 

So as you can see, this is a lot to do every time you open a new terminal or
connect to a cluster. So we can make a file that does all these things for us!

So first go to your home directory (you can do this by just typing `cd` then
enter). Now do `code activate.sh` this should make an empty file that shows up
in your vscode. Here, we'll paste all the commands we did above. Once you do
that, you should have a file that looks like this: 

```
export SPECTRE_HOME=/home/your_home_directory/spectre
source $SPECTRE_HOME/support/Environments/ocean2.sh
export SPEC_ROOT=/home/alexcarpenter46/code_spec/spec
spectre_load_modules
export BIN=$SPECTRE_HOME/build/bin
export PATH=$BIN:$PATH
```

It's important that you don't have exactly what I have above. The file above
points to `your_home_directory`, but yours should point to yours. Make sure to
save the file once you have something like this by doing ctrl+s or command+s
while you're clicked on the file. Now, to use this file, all you have to do is
`. ~/activate.sh` whenever you open a new terminal or connect to a cluster. A
few sanity checks you can do before moving on are opening a new terminal and
running that command. If it does everything correctly, you can try
`echo $SPECTRE_HOME` and it should print out the path you set it equal to, same
for `SPEC_ROOT` and `BIN`. To check if your modules are loaded, you can do
`module list` and a bunch of modules should appear in your terminal. If all this
checks out then you're ready to start building SpECTRE!