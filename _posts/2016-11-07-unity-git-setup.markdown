---
layout: post
title:  "Unity Project Setup"
date:   2016-11-07 3:35:00s -0700
categories: unity git git-lfs nuget setup configuration
author: ryan
---
Unity, lfs, yaml-merge, NuGet...There's a lot to configure in a new Unity project. Enough that going through the
individual steps became tedious, mundane and easy to screw up. In order to make our lives easier, we maintain an empty
Unity project in our Gitlab and now on Github.

* TOC
{:toc}

TLDR: [Use this to start a project.](https://github.com/gruegames/unity_starter_project)

# Machine Setup

Before I go over all the pieces and reasons for everything in the project, here's the most important part: how to use it.
There are a few per-machine steps that need to be done to make use of the repo properly. Here are the instructions for
Windows, I'll try to get to OSX soon, but Windows is definitely the hard one.

## Windows Per Machine Setup

### Install Git
{:.no_toc}

1. https://git-scm.com/download/win
2. Run installer
3. Optionally install SourceTree (handy GUI)
    1. https://www.sourcetreeapp.com/
4. Run this from a command prompt:
    1. `git config --global core.autocrlf false`

### Add Large File Storage to Git
{:.no_toc}

1. Install lfs
    1. https://git-lfs.github.com/
2. Run the following from a command prompt:
    1. `git lfs install`

### Install Unity
{:.no_toc}

1. Currently we're using 5.4.1p3
2. Create an empty project on your machine (I recommend C:\Dev\Empty_Unity)
3. Configure cache server (out of scope here)

### Configure Unity Merging
{:.no_toc}

1. Locate the Unity merge tool
    1. Should reside at C:\Program Files\Unity\Editor\Data\Tools\UnityYAMLMerge.exe
    2. Validate that path or find your latest Unity install
2. Find your .gitconfig file
    1. On Windows, should reside at C:\Users\\\<username\>\\.gitconfig
3. Add the following lines to the end of the file:

        [merge "unityyamlmerge"]
            name = Unity's YAML (scene/prefab) merger
            driver = 'C:\\Program Files\\Unity\\Editor\\Data\\Tools\\UnityYAMLMerge.exe' merge -p --force "%O" "%B" "%A" "%A"

---

#### A Note About Araxis
{:.no_toc}

Unity's default configuration for using Araxis as a fallback tool for merging is broken on windows (unfortunately). At some point Araxis switched from slash (/) options to dash (-) options and never updated their documentation. In order to get Unity's mergetool to work with it properly, we have to update it. The file is called mergespecfile.txt and is located at Editor/Data/Tools in the Unity installation (e.g. C:\Program Files\Unity\Editor\Data\Tools\mergespecfile.txt).

Find this line:

```
* use "%programs%\Araxis\Araxis Merge\compare.exe" /3 /a2 /wait /title1:"Other" /title2:"Base" /title3:"Local" "%l" "%b" "%r" "%d"
```

And replace it with this one:

```
* use "%programs%\Araxis\Araxis Merge\compare.exe" -3 -a2 -wait -title1:"Other" -title2:"Base" -title3:"Local" "%l" "%b" "%r" "%d"
```

# Project Setup

This is all completed in [the Github project](https://github.com/gruegames/unity_starter_project) and ready-to-go. Just
fork and be on your way. For the curious, it's worth ruminating on the what and why.

## Unity

Unity out-of-the-box doesn't come configured the way we like it so there are a few changes that get made. First, we add a
scripting define: `DEVELOPER_TOOLS` to the project. We find it's useful to have a define other than `DEBUG` or `RELEASE`
that controls whether tools we create for development are included in the build or not. Keeping it independent of other
defines allows us to toggle it for any build configuration. Our 'shipping' builds don't include the flag.

Oh, and don't ever do `#if DEVELOPER_TOOLS || UNITY_EDITOR` since it makes fixing those final shipping issues a PITA.

The rest are done under `Edit > Project Settings > Editor`

1. Version Control Mode -> Visible Meta Files
2. Asset Serialization Mode -> Force Text
3. C# Project Generation Additional extensions... -> `txt;xml;fnt;cd;json`

What you don't see in this project is our preference for putting everything we create into a project-specific folder under
assets. Since asset store packages can put anything wherever they want, it helps us keep everything separate.

## Git

Git is great version control. Large game projects traditionally had issue with it, but we're having success utilizing
[Large File Storage](https://git-lfs.github.com/). The only thing interesting in our .gitignore is the addition of
/Assets/Prefs/User/ which we intend to use for a settings system we're working on (and hopefully the topic of a future post).

Our .gitattributes file lists a bunch of extensions we use lfs with. Anything large and binary should get added to it. It's
shown itself to be a little cumbersome to get out ahead of new binary formats showing up in the project and we're trying
to figure out how to deal with it. We might switch to marking `*` for lfs and whitelisting the files we want treated normally
but that seems counter to the git philosophy.

In addition, we're making use of Unity's [smart merge](https://docs.unity3d.com/Manual/SmartMerge.html) tool to help git
merge scene files and the like. It requires setup on each machine, but the triggers are contained in the .gitattributes
file. This makes working in scenes and on prefabs in teams much more tolerable even though it still isn't perfect.

There's also a .gitkeep file under Assets so that git will create that folder for us. Unity won't see the repo as an
actual project without the Assets folder.

## NuGet

Finally, we've placed a nuget.config file into the root of the project. We have started hosting our shared tech for various
projects with NuGet on Artifactory. Hopefully we can release one of them globally soon. We're not fans of Unity's
.unitypackage format and the lack of version and upgrade support. NuGet is filling that gap for us pretty well and gives
us access to a large number of general C# code & extensions that would otherwise be cumbersome to track.

You'll need to create a .cs file somewhere under Assets for Unity to create a project file (and not just a solution).
Once that's done, Visual Studio can handle adding, updating, and removing various NuGet packages for you. We're considering
adding a tool to Unity to view and update NuGet packages. For now, VS provides a good enough tool and we're
(almost) all on Windows anyway.

{% include author.html %}