---
title: A workflow for contributing to d3.parcoords.js
layout: post
tags: git workflow development
---

In this post I try to set out my workflow for contributing to the d3 parallel
coordinates component by Kai Chang. The goal is to have my own clone, with
various feature and fix branches, to have maximum freedom in development without
interfering the progress of the main repository. However, this should not lead
to a 'funny' history. That is, the final contributions that go upstream should
result in a linear history.

## Initial setup

I have my own clone of [syntagmatic/parallel-coordinates](https://github.com/syntagmatic/parallel-coordinates)
at [bbroeksema/parallel-coordinates](https://github.com/bbroeksema/parallel-coordinates).
First we create a clone on our local machine for development:

    # git clone git@github.com:bbroeksema/parallel-coordinates.git
    # cd parallel-coordinates

Next we add the main repository of Kai as a second remote:

    master # git remote add syntagmatic git@github.com:syntagmatic/parallel-coordinates.git
    master # git fetch syntagmatic

## Developing a feature or fix

Now we can start our development. We basically want to separate development of
fixes, to keep all our commits as focussed as possible. One approach to reach
this is by using feature branches. **Note:** this is an approach among many
others I guess, but it fits my development style quite well.

### Creating the branch

As an example, I will add a new feature: a function that returns the extents of
the current brushes in the parallel-coordinates. To get started on this feature
we first make sure that we are up to date with the latest changes from the main
repository.

    master # git pull syntagmatic
    master # git push origin

At this point we have made sure that the master branch of our private clone is
upto date with the main repository (both the local and the github hosted repos
of our private clone).

Now we can start development of our feature. To this extend we create a new
branch with a name which is descriptive for the feature.

    master # git branch brushextents
    master # git checkout brushextents
    brushextents #

### Development

The developement of this new feature requires changes to axis.js. After I have
done the required coding and testing, I add the new feature (in this case with
a single commit) to my branch.

    brushextents # make clean
    brushextents # make
    brushextents # git add src/axis.js
    brushextents # git add d3.parcoords.js
    brushextents # git commit

The 'make clean' and 'make' commands are required to make sure that the file 
d3.parcoords.js is containing our changes as well.

### Creating the merge request

Before we push our feature branch to github, we want to make sure that it still
applies to the current master of the main repo (syntagmatic).

    brushextents # git checkout master
    master # git pull syntagmatic
    master # git checkout brushextents
    brushextents # git rebase master

Your commits should cleanly apply on top of master. In case of conflicts you
will have to modify your commits. Make sure that your feature branch in the end
still has a linear history. Conflict handling is out of scope for this post.
Now, we can push our local branch to github.

    # git push
