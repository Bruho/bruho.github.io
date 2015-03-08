---
layout: default
title: Build Status Addon overview
---

# Build Status plugin for Atlassian Stash [![Build Status](https://api.shippable.com/projects/54ba59b35ab6cc135288edff/badge?branchName=master)](https://app.shippable.com/projects/54ba59b35ab6cc135288edff/builds/latest)
<h5>aka "Build status for automerge"</h5>

[Grab it from Atlassian Marketplace](https://marketplace.atlassian.com/plugins/com.bolyuba.stash.plugin.stash-build-status-plugin) 

## Intro

Stash supports [build status reporting](https://developer.atlassian.com/stash/docs/latest/how-tos/updating-build-status-for-commits.html) for a commit. 
Idea is very simple: every time your code is built (by your CI server like [Jenkins](http://jenkins-ci.org), 
[TeamCity](https://www.jetbrains.com/teamcity/) or [Bamboo](https://www.atlassian.com/software/bamboo/) for example)
you can report status back to Stash along with some metadata. Report looks like this:

{% highlight json %}
{
    "state": "<INPROGRESS|SUCCESSFUL|FAILED>",
    "key": "<build-key>",
    "name": "<build-name>",
    "url": "<build-url>",
    "description": "<build-description>"
}
{% endhighlight %}

Build status is reported for a  **git commit**. For any given commit a number of builds statuses can be reported. If the **key** is the
same, then status will be replaced (think about this part, it is important).

## Out of the box approach

### Overview

As far as Pull Requests are concerned there is a use case for build status. Out of the box Stash allows you to add requirements 
to your repository saying: before a pull request can be merged source branch has to have a given number of successful builds reported. 
This is how configuration page for this feature looks like:

![Pull Request settings page](/images/6.setup-screen-stash.png)

## How does it work

That last line "Requires a minimum of X successful builds" is what Stash gives you. Check out 
[a quick overview of how pull requests work](https://confluence.atlassian.com/display/STASH/Using+pull+requests+in+Stash) in Stash. 
If you enable that option, before pull request is merged Stash will check following thing:

* For given Pull Request find source branch
* For source branch find last git commit id
* For that commit id find all build statuses
* Make sure that number of successful builds statuses found is equal or greater than requested (that drop down on config screen). 
Also there should be no failing builds for given commit.

Use case that comes to mind here: if you have multiple builds for a pull request/branch (running smock tests, 
if they pass run all of the test for example) you can ensure ALL of them pass before pull request is merged.

Bottom line: Stash out of the box functionality lets you build a lot of confidence in changes you made on your 
source branch (which is typically a feature branch or similar if you follow feature branch/git flow, etc.)

# This plugin's approach

## Challenge

While confidence in your feature branch is good, there is one thing I think (otherwise I would not create this plugin!) 
is even more important: confidence in your future. You know, the type of confidence you get by writing code/system only 
you can understand and support (not that we ever do it :) ).

I want to make sure that the changes I make will not break the target branch of my pull request (master branch for example). 
I can write the cleanest code with complete test coverage and still have conflicts because someone else's changes made it 
onto master first. Having a conflict is a good thing actually, the worst thing is: your pull request was merged w/o any 
issues and now tests are failing on your master branch. Both your changes and changes of your colleague built successfully 
separately and all tests passed, but that does not mean they will work when put together.

## Solution ##
Stash actually does the job for you (kinda). Every time pull request is created stash creates a shadow merge (automerge) 
in a branch named `/refs/pull-requests/<id of your pull request>/merge`. This branch will be missing if merge is not
possible due to conflict.

So, for every pull request there is a commit that effectively represents future state of the target branch IF your pull 
request is merge. Think of it as each pull requests can take a sneak pick into its own future! Note: it should be obvious 
that only one of these "futures" will become reality once pull request is merged, but not to worry: stash will automatically 
create new automerge for every open pull request if case target branch of that pull request changes.

Here are 3 simple-ish steps to make sure you always have a stable, predictable future (at least in your code repository):

1. Configure your CI to report builds for commits to Stash (check out link to build status API at the top of this page). 
Your CI needs to monitor `/refs/pull-requests/*` branches. There is some (rather poor) support for this in Bamboo.
TeamCity and Jenkins: use google to find detailed description of setup (or drop me a note)
1. Install plugin. You need to be admin on your Stash instance and if you are I bet you know how to do this
1. Enable plugin for your repository. Will look like this:

![Pull Request settings page with plugin enabled](/images/5.setup-screen-plugin.png)

After that on your pull request page you will see Automerge status. When there was no status reported, it will look like this:

![Can't merge: No build status](/images/1.cant-merge-no-build-status-small.png)

Note that Merge button is disabled. During the build it should look like this:

![Can't merge: Build in progress](/images/2.cant-merge-build-inprogress-small.png)

Then finally you get to this and can merge your pull request:

![Can finally merge](/images/3.can-merge-small.png)



<p>Check out <a href="https://bitbucket.org/bolyuba/stash-build-status-plugin/wiki/Automerge%20build%20status">this page</a> while we are preparing this one</p>
