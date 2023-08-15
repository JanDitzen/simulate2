# simulate2 and psimulate2

![version](https://img.shields.io/github/v/release/janditzen/simulate2)  ![release](https://img.shields.io/github/release-date/janditzen/simulate2) 

### simulate2 - enhanced functions for [Stata's simulate](https://www.stata.com/manuals/rsimulate.pdf)
### psimulate2 - running simulate2 in parallel

__Table of Contents__
1. [Syntax](#1-syntax)
2. [Description](#2-description)
3. [Options](#3-options)
4. [Examples](#4-examples)
5. [Problems](#5-problems)
6. [How to install](#6-how-to-install)
7. [About](#7-about)

# 1. Syntax
```
simulate2 [exp_list] , reps(#) options1 options2 :command
```
and
```
psimulate2 [exp_list], reps(#) parallel(#2, options3) options2 command
```

#### General Options

options1 | Description
--- | ---
nodots | suppress replication dots
dots(#) | display dots every {it:#} replications
noisily |display any output from command
trace | trace command
nolegend | suppress table legend
verbose | display the full table legend


options2 | Description
--- | ---
saving(filename, ...) | save results to filename
seed(options) | control of seed, see  seed options
seedsave(options) | saves the used seeds, see  saving seeds
seedstream(integer) | starting seedstream, only psimulate2
nocls | do not refresh window (only {cmd:psimulate2})
onlydots |display dots rather than output window. Recommended for server use.
docmd(string) | Alterantive command to call do files.
globalid(string_ | Sets id for simulation run. Necessary if multiple instances of psimulate2 are run on the same machine.
{synoptline}

                    
options3 | Description
--- | ---
exe(string) | sets the path to the Stata.exe
temppath(string) | alternative path for temporary files
processors(string) | max number of processors, only for Stata MP
simulate | use simulate rather than simulate2.  If psimulate2 is run on Stata 15 then simulate is automatically used.

All weight types supported by command are allowed; see weight.  simulate2 uses frames and requires Stata 16 or higher.  psimulate2 requires Stata 15 or higher.  psimulate2 works on MacOS, Microsoft Windows and Unix systems. # is the number of repetitions and #2 the number of parallel Stata instances.


# 2. Description

**simulate2** eases the programming task of performing Monte Carlo-type simulations.  Typing

```
simulate2 exp_list, reps(#) : command
```

runs command for # replications and collects the results in `exp_list`.

command defines the command that performs one simulation.  Most Stata commands and user-written programs can be used with **simulate2**, as long as they follow standard Stata syntax.  The by prefix may not be part of command.

`exp_list` specifies the expression to be calculated from the execution of command.  If no expressions are given, exp_list assumes a default, depending upon whether command changes results in e() or r().  If command changes results in e(), the defaultis `b`. If command changes results in r() (but not e()), the default is all the scalars posted to r().  It is an error not to specify an expression in exp_list otherwise.

**simulate2** is an extension (or "hack") of the Stata build-in simulate command.  It extends the command by allowing programs to return macros (strings) to e() or r().  To do so it uses frame post rather than postfile.  The computational costs to return strings are small and simulate2 is only marginally slower than simulate.

**simulate2** and **psimulate2** can save results to frames instead of dtas. Frames and .dta can be both appended as well.

**simulate2** has advanced options to assign seeds, random number generator states to specific draws of simulate2 and save those.  The rngstate (or seed), the random number generator and the seed stream can saved in a separate frame or datatset.

For an introduction into drawing pseudo random numbers see ``set rng``, ``seed`` and ``rngstream``.

**psimulate2** is a parallel version of simulate2 speeding up simulations. Typing

```
psimulate2 exp_list, reps(#) parallel(#2) : command
```

runs command for # replication on #2 parallel Stata instances and collects the results in exp_list.

**psimulate2** splits the number of replications into equal blocks and each block is run on a separate Stata instance.  To do so **psimulate2** creates a do file and a batch file.  The batch file is then used to start a new Stata instance running the corresponding do file.  The running instance is acting as a parent instance, the other are child instances.  The output of psimulate2 differs to the one from simulate or simulate2.  It shows the percentage which is done, elapsed time and expected time left and finishing time.

Before a new instance is started psimulate2 saves the current data set in memory.  This allows all that all Stata instances start using the same dataset.  It is also able move all programs in memory which are not saved in an ado directory, such as programs defined in the do file before calling psimulate2.  Any mata defind functions (see mata mata memory, help file) will be moved from the parent to the child instance.  psimulate2 will create a new mlib file and store it in the temp folder and then set the temp folder as a new ado path.  Mata matrices are moved from the parent to the child instance and are saved in the temp folder.  Globals and not permanently set ado paths are moved as well.  It does not move frames, locals, matrices (only Stata), scalars (only Stata) or values saved in e(), r() or s() from the parent to the child instances.

**psimulate2** uses seedstreams if no seedstream is defined in option seed().  In this case each child instance is assigned its own seedstream.  This ensures that random number draws do not overlap.  Parallel use of psimulate2 is possible with different seedstreams for each machine.  The option seedstream() sets the seedstream for the first instance.

Care is required if **psimulate2** is used in loops.  If no seed options are set, psimulate2 will always use the current Stata seed.  However after psimulate2 is completed it does not (and cannot) set the seed to the last seed from the simulation. Therefore random draws will be the same across iterations of the loop.  To avoid this behaviour seed(`current`) saves the last used seed in a global. In all consecutive iterations of a loop, the global will be picked up, the seed updated for the 
simulate2 runs used and after finishing the global will be updated again.  This allows that draws across iterations of a loop differ.

# 3. Options

Option | Description
 --- | --- 
reps(#) | is required -- it specifies the number of replications to be performed.
nodots and dots(#) | specify whether to display replication dots.  By default, one dot character is displayed for each successful replication.  A red `x' is displayed if command returns an error or if any value in exp_list is missing.  You can also control whether dots are displayed using set dots.
nodots | suppresses display of the replication dots.
dots(#) | displays dots every # replications.  dots(0) is a synonym for nodots.
noisily | requests that any output from command be displayed.  This option implies the nodots option.
trace | causes a trace of the execution of command to be displayed.  This option implies the noisily option.
saving(filename [, suboptions frame append]) | creates a Stata data file (.dta file) consisting of (for each statistic in exp_list) a variable containing the replicates. See prefix_saving_option for details about suboptions.  simulate2 and psimulate2 can save to frames if option frame is used.  It appends an existing dta or frame if option append is used.
nolegend | suppresses display of the table legend.  The table legend identifies the rows of the table with the expressions they represent.
verbose |requests that the full table legend be displayed.  By default, coefficients and standard errors are not displayed. 
simulate | requests psimulate2 to use simulate rather than simulate2.  If psimulate2 is run on Stata 15 then simulate is automatically used.  If option simulate is used each instance is assigned its own seed stream.
seed(options) | controls the random-number seed.  It is possible to set a seed, the random number generator and seed stream or to load all three from either a frame or saved Stata dataset.  Options are:
seed(#) | sets the random-number seed.  If simulate is used in combination with psimulate2, then only seed(#) can be set. Seed streams are automatically assigned. seed(integer1 [string integer3]) sets the seed (integer1), the random-number generator (string) and the seedstream (integer3).  The default for string is the default random-number generator and for integer3 seedstream number 1.
seedstream(integer) |  is a convience option for psimulate2.  It sets the inital seedstream number for the first instance.  For example if 3 instances are set (parallel(3)) and seedstream(4) is used, then instance 1 will use seed stream number 4, instance 2 stream 5 and instance 3 stream 6.  This function allows the parallel use of psimulate2 on multiple computers with the same starting seed, but different seedstreams.
seedsave(filename\frame), [frame append seednumber(#)] | Saves the seeds from the beginning of each draw in a dataset defined by filename.  If option frame is used, it saves the seeds in a frame.  append appends the frame or dataset.  seednumber(#) specifies the first value of variable run.  If not specified it is set to 1 and in the case of option append it is set to _N + 1.  In all cases, the number of the draw, state of the random number generator, the type and the stream are saved.
parallel(#2) | sets the number of parallel Stata instances.  It is advisable not to use more instances than CPU cores are available. 
parallel(#2, exe(string)) | sets the path to the Stata.exe when using psimulate2.  psimulate2 will try to find the path, but might fail if Stata.exe is in a non-conventional folder or has a non-conventional file name. 
parallel(#2, temppath(string)) | sets an alternative path to save temporary files.  psimulate2 saves several do file and .bat files in the temporary folder.  In rare cases Stata might not have read/write rights or it is not possible to start a .bat file from this folder.  In this case temppath() is required.  psimulate2 cleans up the temp folder before using it.  All files starting with psim2_ are removed. 
parallel(#2, processors(integer)) | sets the maximum number of processors each Stata instance is allowed, see set processors.  This is only relevant for Stata MP.  For example if Stata MP with 4 cores is used and two parallel instance of psimulate2, then the remaining two cores can be used for each instance.  The default is 1, meaning that psimulate only one processor is available to each Stata instance.
docmd(string) | specifies an alternative command to run do files. For example on a Ubuntu system, {cmd:docmd(stata)} is necessary to start a do file.
onlydots | instead of the progress window dots are displayed. The option is intended to minimize the size of log files.
globalid(integer) | Multiple instances of psimulate2 can be run on the same machine. If the same path to save temporary files is used, files may be overwritten. globalid(integer) specifies the number of the parallel instance to avoid files being overwritten.




# 4. Examples

Make a dataset containing the OLS coefficient, standard error, the current time and save the seeds in a frame called seed_frame. Perform the experiment 1000 times:

```
        program define testsimul, rclass
                version 17
                syntax anything
                clear
                set obs `anything'
                gen x = rnormal(1,4)
                gen y = 2 + 3*x + rnormal()
                reg y x
                matrix b = e(b)
                matrix se = e(V)
                ereturn clear
                return scalar b = b[1,1]
                return scalar V = se[1,1]
                return local time "`r(current_time)'"
        end
        
    . simulate2 time = r(time) b = r(b) V = r(V), reps(1000) saveseed(seed_frame,frame): testsimul 100
```

Now we can pick up the seeds and re-do the experiment for the first 500 repetitions:

```
simulate2 time = r(time) b = r(b) V = r(V), reps(500) seed(seed_frame seeds, frame): testsimul 100
```

and for the second 500 repetitions the starting seed is set to seed number 501:

```
simulate2 time = r(time) b = r(b) V = r(V), reps(500) seed(seed_frame seeds, frame start(501)): testsimul 100
```
Likewise, we can first do 500 draws, save the seeds, do another 500 draws, append the saved seeds and do the experiment for all 1000 draws. For comparison results are saved in frames:

```
simulate2 time = r(time) b = r(b) V = r(V), reps(500) seed(123) seedsave(seed_frame, frame) saving(first500, frame): testsimul 100
simulate2 time = r(time) b = r(b) V = r(V), reps(500) seedsave(seed_frame, frame append) saving(second500, frame): testsimul 100
simulate2 time = r(time) b = r(b) V = r(V), reps(1000) seed(seed_frame seeds, frame): testsimul 100
```

Note that for the second run, no new seed is set and the option append is used. First we compare the results for the first 500 draws, then for the second 500 draws:

```
sum if _n <= 500
frame first500: sum
sum if _n > 500
frame second500: sum
```

The results of the first two command lines and the second two command lines are expected to be identical.

Likewise, we can parallelise the simulation from above. For example we want to run two instances at the same time, i.e. instance 1 runs the first 500, instance 2 the second 500 repetitions:

```
psimulate2 , reps(500) seed(123) parallel(2, temppath("C:\psim2_temp")): testsimul 200
```

parallel(2, temppath("C:\LocalStore\jd71\temp")) sets two parallel instances.  temppath() specifies the path temporary files are saved to.  This option is only necessary if Windows does not allow to run batch files from the temp folder. The runs 1 and 500 will have the same random-number state, but a different seed stream and therefore random draws will differ.

In the case one or more psimulate2 are nested in a loop or called sequentially, they would use the same initial seed.  To avoid this, psimulate2 returns the last seed state of the last instance in r().  To run psimulate sequentially we code:

```
psimulate2 , reps(100) seed(_current) p(2) seedsave(seed, frame): testsimul 200
set rng `r(rng_current)'
set rngstate `r(rngstate)'
psimulate2 , reps(100) seed(_current) p(2) seedsave(seed, frame): testsimul 200
```

Using `current` within seed() psimulate2 will use the `current` seed of the parent instance as an initial seed for all child instances.  Each child instance will still have a different seed stream to ensure the random number draws are different.

## 4.1 Example Unix Server

Let's assume we want to run the example above, but we want to run the simulation with 20, 50 100 and 1000 observations. 
We also want to compare results if errors are standard normal and uniform distributed. 
Option `uniform` is added to the testsimul program:

```
program define testsimul, rclass
    version 18
    syntax anything, [uniform]
    clear
    set obs `anything'
    gen x = rnormal(1,4)
    if "`uniform'"== "" gen e = normal()
    else  gen e = runiform(-1,1)
    gen y = 2 + 3*x + e
    reg y x
    matrix b = e(b)
    matrix se = e(V)
    ereturn clear
    return scalar b = b[1,1]
    return scalar V = se[1,1]
    return local time "`c(current_time)'"
end
```

We can use a - say Ubuntu - Unix server with a total of 20 cores and we want to use all of them. Stata is installed on the machine and can be started from the command line with the command **stata**. A folder to store temporary files is created in the home directory and called `tmp`.

To run the simulations, we write two do files, one for the DGP with standard normal errors, one for the DGP with uniform errors. The do files are called `spec1.do` and `spec2.do`.
Both do files include a loop over the set of number of observations and save the simulated results.
The simulation is repeated 1000 times with 9 parallel instances:

**spec1.do**:
```
    clear
    set seed 12345
    foreach N in 20 50 100 1000 {
        psimulate2 , reps(1000) parallel(9, temppath("/home/tmp")) seed(_current) onlydots globalid(1) docmd(stata): testsimul `N' 
        save res_`N'_spec1
    }
```

To run the second specification, we add the option `uniform` and alter the filename of the dataset with the Monte Carlo results.

**spec2.do**:
```    
    clear
    set seed 678910}
    foreach N in 20 50 100 1000 }
        psimulate2 , reps(1000) parallel(9, temppath("/home/tmp")) seed(_current) onlydots globalid(2) docmd(stata): testsimul `N' , uniform}
        save res_`N'_spec2  
```

The ``spec1.do`` and ``spec2.do`` do files can run on the server at the same time and we make optimal use of the 20 cores.
Depending on the server, it might be necessary to write a batch file which allocates memory, number of cores and run time for each do file. After the simulations are run, there should be a total of 8 files, 4 for each specification.


Next we discuss the options in detail:


Option | Description
 --- | --- 
parallel(9, temppath("/home/tmp")) | sets the number of parallel instances to 9. Each do file then uses in total 10 cores.
The option temppath()} specifies the path to save temporary files which needs to be set to be read and writeable.
seed(_current) | ensures that the seed is altered for the different values. Otherwise the first 20 observations if N=50 would be the same as the observations of the case N=20.
onlydots | helps to reduce the size of the log files. Instead of an overview of the progress, only dots are displayed.
globalid() | sets the id for each of the instances of **psimualte2**.  All temporary files are named `psim2#Instance_` and thus it is ensured that no temporary file is overwritten. Alternatively we could specify a temporary path (eg: "/home/tmp/tmp_1") for each do file/specification.
docmd(stata) | specifies the command a do file is started with from the command line/within Stata. On our Ubuntu server, new do files are started with `"home/dofiles/dofile.do"` and we need to specify the command to do so.


# 5. Problems
 - On some Windows installations or servers the default temporary folder is locked or not accessible. In this case Stata and psimulate2 cannot write any files in the temporary folder.  Option temppath() can be used to set an alternative temporary folder.
 - psimulate2 can crash if the temporary path or any other path it writes on is in a cloud storage folder from services such as Dropbox, OneDrive or Backup and Sync from Google. A fix is to pause those services.
- psimulate2 has problems with long names, such as variable names or command names. In such cases it tends to shorten the names which might cause interruptions in the code.  The best solution is to shorten the names.
- Unix servers require the nocls function to run psimulate2.
- The onlydots options should be used with servers tpo avoid large log files.

# 6. How to install

**simulate2** and **psimulate2** can be directly installed from GitHub:

```
net install simulate2, from(https://janditzen.github.io/simulate2/)
``` 

# 7. About

## Author

Jan Ditzen (Free University of Bozen-Bolzano)

Email: jan.ditzen@unibz.it

Web: www.jan.ditzen.net

simulate2 was inspired by comments from Alan Riley and Tim Morris at the Stata User group meeting 2019 in London.  Parts of the program code and help file were taken from simulate.  Kit Baum initiated the integration of MacOS and Unix and assisted in the implementation.  Michael Porst and Gabriel Chodorow-Reich provided much valued feedback. I am grateful for all of their help.  All remaining errors are my own. I do not take over responsibility for any computer crashes, lost work or financial losses following the use of *(p)simulate2*.

## Change Log

Version 1.07 to 1.08
- added options onlydots, docmd() and globalid to improve support for servers

Version 1.06 to 1.07
- fix in program lines with more than 250 characters (thanks to Gabriel Chodorow-Reich)

Version 1.05 to 1.06
- fix in program lines with more than 250 characters (thanks to Gabriel Chodorow-Reich)

Version 1.05 to 1.06
- various small bug fixes (thanks to Gabriel Chodorow-Reich)

Version 1.04 to 1.05
- bug fix in exepath

Version 1.03 to 1.04
- bug fixed if data appended to frame but frame does not exists
- fix in temppath local

Version 1.02 to 1.03
- improved behaviour for long lines in do files or programs
- warning message if no seed set

Version 1.01 to Version 1.02
- Mata matrices and scalars are moved from parent to child instance as well.

Version 1.0 to Version 1.01
- bug fixes in program to get exe name
- no batch file written anymore; support for MacOS
- added options nocls and processors to set max processors for Stata MP
- added that mata defined function are moved.

