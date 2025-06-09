---
layout: default
title: gsoc-selection
description: a detailed account of my gsoc journey and an example-driven explanation of data imputation.
---

## how it started

so this journey began back in 2023 when i approached a senior in my college, devesh, to ask how he got into gsoc. back then, it seemed so cool—especially since he had gotten in during his freshman year.

he told me the first step is to shortlist organizations based on their past projects, ideally finding something that matches your tech stack or one you’re interested in learning. the proposal submission deadline was in april, and i had only started looking in march, so my chances were slim. still, i thought, why not give it a shot?

i had some experience writing r code, so i checked out r-related projects and found some focused on package development. now, package development (as in other languages) isn’t rocket science; of course, the complexity depends on what the package does. since r has been around for a while, there are plenty of dependencies to inherit from and build upon. i chose the [imputetestbench package](https://cran.r-project.org/web/packages/imputeTestbench/index.html) for my application and completed the tasks listed on the project page.

### what is imputation?

the imputetestbench package deals with data imputation — essentially,<span style="color:blue;">filling in missing data</span>. imagine you’re baking a cake and realize you’re out of sugar. you impute (replace) it with honey. similarly, in data science, imputation means filling in missing values with educated guesses. but unlike baking, bad imputation can ruin your analysis.

for starters, think about a simple series:  
`1, 3, 5, 7, _, 11`  
it’s easy to guess the missing number is 9, since it’s an arithmetic progression of odd numbers (a = 1, d = 2).

but real-world data is rarely this neat. datasets often have missing values that don’t follow a clear pattern, so you need smarter ways to fill in the gaps.

example: daily temperature dataset

suppose you have this dataset of daily temperatures (in °c) for a week:

```
day 1: 22
day 2: 24
day 3: ? (sensor failed)
day 4: 28
day 5: ? (sensor failed)
day 6: 30
day 7: 31
```
goal: fill in days 3 and 5.  
here’s how different imputation methods work:

#### method 1: mean imputation

"just use the average!"  
calculate the mean of existing temps:  
`(22 + 24 + 28 + 30 + 31) / 5 = 27°c`

result:

```
day 3: 27
day 5: 27
```

pros: simple.  
cons: ignores trends. day 3 was likely between 24 and 28 (rising trend), not exactly 27.

#### method 2: time-series imputation

"follow the pattern!"  
notice temps are rising each day. use neighboring values:

- day 3: midpoint between day 2 (24) and day 4 (28) → 26°c
- day 5: midpoint between day 4 (28) and day 6 (30) → 29°c

result:

```
day 3: 26
day 5: 29
```

pros: captures trends.  
cons: struggles with outliers (e.g., sudden storms due to low pressure).

#### method 3: machine learning imputation

"let the algorithm predict!"  
train a model on past data (e.g., mondays are cooler, fridays hotter). suppose the model predicts:

```
day 3: 25°c (weekday trend)
day 5: 29.5°c (weekend trend)
```

pros: handles complex patterns.  
cons: requires historical data and compute power.

### why imputation matters

bad imputation leads to garbage in, garbage out. for example:

- if day 3 was actually 19°c (due to a storm), mean imputation (27°c) would distort climate models.
- the imputetestbench package lets you test methods side-by-side to pick the best fit for your data.

<span style="color:blue;">tl;dr: imputation is like detective work - use clues (existing data) to fill in gaps smartly.there is no one-size-fits-all solution!</span>

although i didn’t get selected that year(well my project itself did not get selected so i consoled myself by thinking that if it had been, i definitely would have gotten in, lol), the process taught me a lot about open source, project selection, and the importance of starting early. it also gave me the motivation to try again, this time with better preparation and a clearer focus.

## back to present

fast forward to 2025: i had left my job to prep for an entrance exam for iisc. even with a sub-300 rank, i didn’t get into my preferred program. right around then, gsoc was announced—so i figured, why not try again?

i was still on the r contributors slack, so i pinged heather (admin + r project board member) about getting back into contributing. she assigned me some issues—one was fixing example errors in package docs, another was a screen flicker bug when scrolling. since r is mostly c under the hood, my c experience helped a lot. and, not gonna lie, having llms around made debugging faster (but seriously, don’t let llms do all your thinking).

after a few patches, heather mentioned she was mentoring a gsoc project around the r dev container. this container is basically a ready-to-go environment: you can run r, build packages, and skip the hassle of local installs or dependency hell. there are some size limits, but for most users (even power users), it’s more than enough.

the project had been built over the last two gsoc cycles, but some things still needed work—like <span style="color:blue;">lldb integration for c debugging</span>, <span style="color:blue;">better mac/windows support</span>, and <span style="color:blue;">positron ide compatibility</span>.


#### lldb: c debugging inside the container

most of r is written in c, so debugging at that level is huge. the goal was to get lldb working inside the container. this meant building r with debug symbols and setting up the right launch config.

**docker run config for debugging:**

```json
// in devcontainer.json
"runArgs": [
  "--cap-add=SYS_PTRACE",
  "--security-opt", "seccomp=unconfined"
]
```


<span style="color:blue;">these flags are essential for ptrace-based debugging like lldb</span>.

**build r with debug symbols:** (building with debug symbols is like making your program “debugger-friendly” so you can actually see and fix bugs in your source code, not just guess from some errors.)

run `./configure --enable-debug && make -j4`, then use lldb or vscode to set breakpoints and debug your c code while running r.

#### making it work on macos & windows

while the container already works well on linux, for <span style="color:blue;">real adoption, it needs to run smoothly on mac and windows too</span>.

**macos:**  fix file sync issues using docker delegated mounts with `consistency=delegated` and install any missing r dependencies.  
**windows (wsl2):** resolve path mapping quirks with the correct workspace mounts and validate file access using `docker exec`.

the plan is also to build docker containers that support both arm and x86 architectures, so everyone can use the same setup regardless of their machine.

#### positron ide integration

not everyone prefers vscode, so adding <span style="color:blue;">positron ide support</span> is a priority. the idea is to use devpod to create an ssh-enabled workspace, then connect positron remotely.

**devpod workspace setup:**


```bash
devpod workspace create --provider docker-desktop --ide positron --ports 2000:2000
```

**positron config (host-side):**

```json
{
  "remote.SSH.workspaces": {
    "r-dev": {
      "host": "r-dev.devpod",
      "lldbPort": 2000
    }
  }
}
```


with this, you will be able to set breakpoints in positron, start a debug session, and inspect c code—just like you would in vscode.



### why this matters

<span style="color:blue;">making the r dev container robust and easy to debug means more contributors can jump in without fighting their setup</span>. it’s about lowering the barrier to entry for open source and making sure your time goes into code, not config.

---

i’m excited to see this land in time for r dev day @ user! 2025. if you’re reading this and want to get started with r development, this container is probably the easiest way in. and if you hit a wall, you know where to [reach out](/contact).



