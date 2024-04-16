+++
title = 'Google Summer of Code 2023 Final report'
date = 2023-09-05
draft = false
+++

**Student**: [Pranav Goswami](https://github.com/pranavchiku) \
**Project**: [Compiling SciPy using LFortran](https://summerofcode.withgoogle.com/programs/2023/projects/6IzqNOrE) \
**Organisation**: [Fortran-lang](https://fortran-lang.org/) \
**Mentor**: [Ondřej Čertík](https://github.com/certik) \
**Weekly Blogs**: [pranavchiku.github.io](https://pranavchiku.github.io/gsoc.html)

## Overview of Work Done

I focused on enhancing compatibility of LFortran with the SciPy Fortran codebase. 
Successfully implemented functionalities like `COMMON`, `BLOCK DATA`, `SAVE`, `DATA IMPLIED DO LOOPS`, `EXTERNAL`, etc.
My tasks encompassed intricate undertakings, including AST to ASR conversion, which culminated in the successful 
compilation of each file to ASR format. This effort achieved a notable 77.01% of files compiled to LLVM.
Moreover, I played a pivotal role in setting up continuous integration (CI) to ensure the functionality of the ASR.
Recognizing the necessity of comprehensive alignment, I along with [Ondřej Čertík](https://github.com/certik/) initiated the alignment of [specfun](https://github.com/scipy/scipy/tree/main/scipy/special/specfun) with GFortran. 
This included the endeavor to incorporate LFortran into SciPy's build system and the execution of tests crafted by the SciPy team, ensuring seamless integration and test execution.

## Detailed Description

As identified while curating the proposal, the main obstacles to getting [SciPy](https://github.com/scipy/scipy) compiled with [LFortran](https://github.com/lfortran/lfortran/) were `COMMON BLOCKS`, `BLOCK DATA`, and `SAVE`. 
Consequently, I began my GSoC journey by implementing `COMMON` Blocks. 

- **Week 1, 2 and 3**: I concentrated on the successful implementation of COMMON blocks and bug fixes for the same.
- **Week 4**: Worked towards getting LFortran successfully compile BLOCK DATA. We realised that BLOCK DATA is used at very very few
places in SciPy, so we tried to workaround SciPy code and got it working correctly.
- **Week 5**: Up until the fifth week, the majority of our code was compiling to ASR, and the most common bug was the SAVE attribute, so I began working to get the SAVE attribute functioning with LFortran and was ultimately successful.

After removing the main roadblocks, we could see a clear path to the finish line, but then we encountered new features such as 
`DATA IMPLIED DO LOOPS`, `ENTRY STATEMENTS`, `INTRINSIC FUNCTIONS`, `ARGUMENT CASTING`, `IMPLIED DO LOOPS`, etc. So, I began working on it in the fifth week, beginning with Data Implied Do loops.
- **Week 5**: I got `Data Implied do loops` succesfully implemented.
- **Week 6**: Got `Entry` functions working with LFortran.
- **Week 7**: Implemented `min`, `max` intrinsic for multiple arguments.
- **Week 8**: Later we encountered that SciPy uses, `-fallow-argument-mismatch` option of GFortran which allows mismatch in function call arguments, so
I implemented `--implicit-argument-casting` compiler option for the same. With this we got **entire SciPy compiled to ASR**
- **Week 9 and 10**: Then, I worked towards getting `Implied Do Loops` completely implemented, but I faced a few difficulties over there
and then we decided to go with Depth First way to compile SciPy towards LLVM, so I started working on porting LFotran in SciPy build system.
- **Week 11**: We were able to sucessfully port LFortran into SciPy build system, and decided to get `specfun` fully working with LFotran, I 
mainly focused on understanding how SciPy build system, test suite is working.
- **Week 12**: I replicated the steps to build SciPy on Linux System, and achieved partial success for the same.

List of Issues created / closed: [lfortran/issues](https://github.com/lfortran/lfortran/issues?q=+is%3Aissue+author%3APranavchiku+)  \
List of PRs opened / merged / closed: [lfortran/pulls](https://github.com/lfortran/lfortran/pulls?q=+is%3Apr+author%3APranavchiku)

### My commits during GSoC:

```console
% git shortlog --summary --numbered --since="2023-05-15"
   310  Ondřej Čertík
   200  Pranav Goswami    <---------------
   164  Ubaid Shaikh
    84  Gagandeep Singh
    69  Smit Lunagariya
    69  Thirumalai Shaktivel
    37  HarshitaKalani
    19  gptsarthak
    18  Lubis
     7  Harshita Kalani
     4  Luthfan Anshar Lubis
     2  Benson Muite
```

## Current State
LFortran now compiled entire SciPy codebase to ASR, 77.01% of files get compiled to LLVM. We have a robust CI to assure the efficient compilation to ASR on every commit, we have a dedicated CI to build, and run SciPy tests for `scipy.special`.


## Future Work

After completion of 12 weeks of GSoC, I continued my work to get `specfun` aligned completely with LFotran and guess what, LFortran is able to compile, and run majority of test without any divergence for `specfun`. There are a few tests related to `nan` which needs to be fixed, we need to develop a robust build system for SciPy, also will remove mangling `_` problems that we are facing right now. I will keep contributing to LFortran and get entire SciPy compiled as soon as possible :)

## Learnings

I learnt a lot of new things throughout the journey:
- I now have great confidence on various Fortran functionalities.
- I pursued an internship simultaneously, hence time management was the key quality I learnt here.
- Got to know about various different `gfortran` functions, flags.
- Ddebugging tools like `lldb`, `gdb`, etc.
- Importance of minute details like utiziling wrong python path in conda environment during SciPy build, `branch_cut`, etc.
- `git` version control, `conda`, `mamba`, etc.

## Acknowledgments

I am grateful to [Ondřej Čertík](https://github.com/certik) for believing in me, guiding me throughout, providing valuable feedback, reviewing PRs. I wish to extend my thanks to [Gagandeep Singh](https://github.com/czgdp1807), [Thirumalai Shaktivel](https://github.com/Thirumalai-Shaktivel) ! Thanks to the great community of LFortran who made the work enjoyable and amazing! Looking forward to many more commits ;)


