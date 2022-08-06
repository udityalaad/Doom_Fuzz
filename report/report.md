# REPORT

## Personal Information
- Student Name: Uditya Uday Laad
- Student ID: 20986041
- WatID: ulaad


## What have been done to compile and run the code
After cloning the fuzz-battle and chocolate-doom repos:
- #### Compiled the code with Address Sanitizer 'ON'
    ```
    cmake -DCMAKE_C_COMPILER=clang-10 -DCMAKE_EXPORT_COMPILE_COMMANDS=ON \
      -DCMAKE_C_FLAGS='-fsanitize=fuzzer-no-link,address -fprofile-instr-generate -fcoverage-mapping -g -ggdb3 -O2' \
      ../ -GNinja
    
    ninja
    ```
- #### Executed the fuzzer with -runs=10 & without any initial seed corpus
    ```
    ./src/doom_fuzz -runs=10  >/dev/null
    ```
.

## What have been done to increase the coverage:
- #### Disabled data-leaks & executed with '100 runs' (without any intial seed corpus):
    ```
    export ASAN_OPTIONS="detect_leaks=0"
    ./src/doom_fuzz -runs=100 -detect_leaks=0  >/dev/null
    ```

- #### Executed with an intial seed corpus:
    (WAD files, mostly downloaded from https://www.doomworld.com/idgames and other internet sources)
    Was able to achive far better coverage.
    ```
    ./src/doom_fuzz -runs=100  ../Extras/CORPUS ../Extras/SEED -detect_leaks=0 >/dev/null
    ```

- #### Ran with different configurations of '-runs -jobs -workers'
    E.g.: ``` ./src/doom_fuzz -runs=100 -jobs=10 -workers=5 ../Extras/CORPUS ../Extras/SEED -detect_leaks=0 >/dev/null ```

- #### Ran with '-use_value_profile' option enabled
    E.g.: ``` ./src/doom_fuzz -runs=100 -jobs=10 -workers=5 ../Extras/CORPUS ../Extras/SEED -detect_leaks=0 -use_value_profile=1 >/dev/null```
    
    It is used to guide the corpus expansion.
    .

- #### Tried multiple runs by toggling the address sanitizer
.

In addition, some more changes were made to ```src/fuzz_target.c``` file, to cover certain parts of the code.
Executed the program with following changes, between multiple runs: (Different versions of ```fuzz_target.c``` can be found in ```fb/src/```)
- #### Tried the following values for 'filename' at ```fuzz_target.c:209```:
    - ##### Changed the value of 'filename' to '' (empty) ```(Refer: fuzz_target_v2.c)```
            Helped cover (w_wad.c:139) to (w_wad.c:141) in w_wad.c file

    - ##### Changed the value of 'filename' to '~fuzz.pk3' ```(Refer: fuzz_target_v3.c)```
            Helped cover the following lines in w_wad.c file
            a] (w_wad.c:118) to (w_wad.c:132)
            b] (w_wad.c:144) to (w_wad.c:161)
            .
        It also helped cover some other lines, taking the coverage higher.
        .

- #### Removed the following options at 'fuzz_target.c:145':
    a] Removed '-nogui', '-nosound', '-nomusicpacks'  ```(Refer: fuzz_target_v4.c)```

    b] Removed '-nosfx', '-nomusic'     ```(Refer: fuzz_target_v5.c)```
    .
The above 2 changes help improve the overall coverage.
.
- #### Made the following 2 changes at 'fuzz_target.c:143':
    - ##### Provided "Random" value (random string value) for "-iwad" argument ```(Refer: fuzz_target_v6.c)```
        This helped improve the overall coverage
        .

    - ##### Disabled Address Sanitizer and provided "" value (empty string value) for "-iwad" argument ```(Refer: fuzz_target_v7.c)```
        - This helped improve the overall coverage.
        - But with AddressSanitizer 'ON', this change would otherwise result in a bug as follows:
        ```
        =================================================================
        ==17235==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x60200000042f at pc 0x0000003e3db9 bp 0x7ffd66ace6f0 sp 0x7ffd66acdea0
        READ of size 1 at 0x60200000042f thread T0
            #0 0x3e3db8 in strcasecmp (/home/doom/stqam/fb/build/src/doom_fuzz+0x3e3db8)
            #1 0x4cdc1f in W_AddFile /home/doom/stqam/fb/build/../chocolate-doom/src/w_wad.c:143:9
            #2 0x477f47 in D_AddFile /home/doom/stqam/fb/build/../src/fuzz_target.c:104:12
            #3 0x477f47 in LLVMFuzzerTestOneInput /home/doom/stqam/fb/build/../src/fuzz_target.c:178:3
            #4 0x382a31 in fuzzer::Fuzzer::ExecuteCallback(unsigned char const*, unsigned long) (/home/doom/stqam/fb/build/src/doom_fuzz+0x382a31)
            #5 0x38476a in fuzzer::Fuzzer::ReadAndExecuteSeedCorpora(std::__Fuzzer::vector<fuzzer::SizedFile, fuzzer::fuzzer_allocator<fuzzer::SizedFile> >&) (/home/doom/stqam/fb/build/src/doom_fuzz+0x38476a)
            #6 0x384df9 in fuzzer::Fuzzer::Loop(std::__Fuzzer::vector<fuzzer::SizedFile, fuzzer::fuzzer_allocator<fuzzer::SizedFile> >&) (/home/doom/stqam/fb/build/src/doom_fuzz+0x384df9)
            #7 0x372e7e in fuzzer::FuzzerDriver(int*, char***, int (*)(unsigned char const*, unsigned long)) (/home/doom/stqam/fb/build/src/doom_fuzz+0x372e7e)
            #8 0x39c912 in main (/home/doom/stqam/fb/build/src/doom_fuzz+0x39c912)
            #9 0x7f1b9f210bf6 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x21bf6)
            #10 0x347bf9 in _start (/home/doom/stqam/fb/build/src/doom_fuzz+0x347bf9)
        ```
    .


## What bugs have been found? Can you replay the bug with chocolate-doom, not with the fuzz target?
The following bug was detected while fuzzing [without having to change any parameters in fuzz_target.c (for example - as in previous (above) case)]:
```
AddressSanitizer:DEADLYSIGNAL
=================================================================
==26921==ERROR: AddressSanitizer: SEGV on unknown address 0x000000000070 (pc 0x000000574ac5 bp 0x7fff337636f0 sp 0x7fff337635c0 T0)
==26921==The signal is caused by a WRITE memory access.
==26921==Hint: address points to the zero page.
    #0 0x574ac5 in P_GroupLines /home/doom/Workspace/ulaad/fb/build/../chocolate-doom/src/doom/p_setup.c:593:28
    #1 0x575ed7 in P_SetupLevel /home/doom/Workspace/ulaad/fb/build/../chocolate-doom/src/doom/p_setup.c:838:5
    #2 0x4ff201 in G_DoLoadLevel /home/doom/Workspace/ulaad/fb/build/../chocolate-doom/src/doom/g_game.c:657:5
    #3 0x478419 in LLVMFuzzerTestOneInput /home/doom/Workspace/ulaad/fb/build/../src/fuzz_target.c:271:3
    #4 0x382a51 in fuzzer::Fuzzer::ExecuteCallback(unsigned char const*, unsigned long) (/home/doom/Workspace/ulaad/fb/build/src/doom_fuzz+0x382a51)
    #5 0x382195 in fuzzer::Fuzzer::RunOne(unsigned char const*, unsigned long, bool, fuzzer::InputInfo*, bool*) (/home/doom/Workspace/ulaad/fb/build/src/doom_fuzz+0x382195)
    #6 0x384ab7 in fuzzer::Fuzzer::ReadAndExecuteSeedCorpora(std::__Fuzzer::vector<fuzzer::SizedFile, fuzzer::fuzzer_allocator<fuzzer::SizedFile> >&) (/home/doom/Workspace/ulaad/fb/build/src/doom_fuzz+0x384ab7)
    #7 0x384e19 in fuzzer::Fuzzer::Loop(std::__Fuzzer::vector<fuzzer::SizedFile, fuzzer::fuzzer_allocator<fuzzer::SizedFile> >&) (/home/doom/Workspace/ulaad/fb/build/src/doom_fuzz+0x384e19)
    #8 0x372e9e in fuzzer::FuzzerDriver(int*, char***, int (*)(unsigned char const*, unsigned long)) (/home/doom/Workspace/ulaad/fb/build/src/doom_fuzz+0x372e9e)
    #9 0x39c932 in main (/home/doom/Workspace/ulaad/fb/build/src/doom_fuzz+0x39c932)
    #10 0x7fd1f4352bf6 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x21bf6)
    #11 0x347c19 in _start (/home/doom/Workspace/ulaad/fb/build/src/doom_fuzz+0x347c19)
```

.

## Did you manage to compile the game and play it on your local machine (Not inside Docker)?
Yes I was able to play the game on my machine.
I have also attached a screenshot, for your reference (in ```fb/Report/```)

- I ran it on my Ubuntu Virtual Machine.
- I played chocolate-doom with DOOM.wad.

.

## Coverage
I was able to achieve an overall coverage of 10.3%.
- ```p_setup.c```: 82.3% lines were covered
- ```w_wad.c```: 89.4% lines were covered