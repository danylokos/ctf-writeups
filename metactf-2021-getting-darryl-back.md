# Dread Pirate Darryl - MetaCTF 2021

## Initial static analysis with Ghidra

1. Fire up Ghidra and find refereces to the string used for the password prompt:

    ```
        s_Hey,_Darryl,_I'm_going_to_need_y_001060a0     XREF[3]:
                                                                    FUN_0010245e:00102471(*),
                                                                    FUN_001025de:001025f1(*),
                                                                    FUN_00102b22:00102b35(*)  
    001060a0 48 65 79        ds         "Hey, Darryl, I'm going to need your password 
                2c 20 44 
                61 72 72 
    001060fd 00              ??         00h
    001060fe 00              ??         00h
    001060ff 00              ??         00h
    ```

    References:

    ```
    FUN_0010245e:00102471(*)
    FUN_001025de:001025f1(*) 
    FUN_00102b22:00102b35(*) 
    ```

1. Follow the execution flow for each of the addresses till you find a branching point for password success/failure check:

    `FUN_0010245e:`

    ```
        LAB_00102509                                    XREF[1]:     001091c9(*)  
    00102509 ff d1           CALL       RCX
    0010250b 84 c0           TEST       AL,AL
    0010250d 74 61           JZ         LAB_00102570
    0010250f 48 8d 35        LEA        RSI,[s_Okay_Darryl,_here_you_go!_00106120]       = "Okay Darryl, here you go! "
            0a 3c 00 00
    ```

    Here we can see a function call `CALL` which uses adress from `RCX` register followed by `TEST` that compares least significant byte of EAX `AL` and jumps `JZ` to `LAB_00102570` if that comparison set a zero flag `ZF` to `1`. If the jump didn't occur we can see a loading of a success message into the `RSI` register.

## Dynamic analysis with LLDB

1. Lets fire LLDB with our binary:

    ```sh
    $ lldb ./darryl_vault 
    (lldb) target create ./darryl_vault
    Current executable set to '/home/kali/Desktop/darryl_vault' (x86_64)
    ```

1. Start execution by typing `r`

    ```sh
    (lldb) r
    Process 1629 launched: '/home/kali/Desktop/darryl_vault' (x86_64)
    Welcome, Dread Pirate Darryl, to your secret vault!
    What secret would you like to retrieve?
    1: Your mothers maiden name
    2: The name of your first pet
    3: The city in which you were born
    4: The location where you keep your treasure
    Enter a number (1-4) to continue... 
    ```

1. `Ctrl+B` to stop the execution so we can set our breakpoint:

    ```sh
    Process 1629 stopped
    * thread #1, name = 'darryl_vault', stop reason = signal SIGSTOP
        frame #0: 0x00007ffff7cf58be libc.so.6`__read + 14
    libc.so.6`__read:
    ->  0x7ffff7cf58be <+14>: cmp    rax, -0x1000
        0x7ffff7cf58c4 <+20>: ja     0x7ffff7cf5920            ; <+112>
        0x7ffff7cf58c6 <+22>: ret    
        0x7ffff7cf58c7 <+23>: nop    word ptr [rax + rax]
    (lldb) 
    ```

1. But first we need to find out our binary's base address (by default Ghidra load our binary at base address 0x100000, so all addresses are relative to 0x100000):

    ```sh
    (lldb) [  0] 569F12C5-B02A-E777-66B5-0BAC4E64803A-B0DD48D0 0x0000555555554000 /home/kali/Desktop/darryl_vault
    ```

1. Now let's set a breakpoint on `0x102509` (`CALL RCX`):

    ```sh
    (lldb) b -a 0x0000555555554000+0x2509
    Breakpoint 1: where = darryl_vault`___lldb_unnamed_symbol7$$darryl_vault + 171, address = 0x0000555555556509
    ```

1. We can double check that is the correct address by dissassembling it:

    ```sh
    (lldb) dis -s 0x0000555555554000+0x2509
    darryl_vault`___lldb_unnamed_symbol7$$darryl_vault:
    0x555555556509 <+171>: call   rcx
    0x55555555650b <+173>: test   al, al
    0x55555555650d <+175>: je     0x555555556570            ; <+274>
    0x55555555650f <+177>: lea    rsi, [rip + 0x3c0a]
    ```

    As we can see those are the same instruction we've seen in Ghidra.

1. Continue execution with `c`:

    ```c
    (lldb) [  0] 569F12C5-B02A-E777-66B5-0BAC4E64803A-B0DD48D0 0x0000555555554000 /home/kali/Desktop/darryl_vault
    ```

1. Our breakpoint got hit:

    ```sh
    Hey, Darryl, Im going to need your password real quick for this one. Enter it here for me: 
    AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
    Process 1738 stopped
    * thread #1, name = 'darryl_vault', stop reason = breakpoint 1.1
        frame #0: 0x0000555555556509 darryl_vault`___lldb_unnamed_symbol7$$darryl_vault + 171
    darryl_vault`___lldb_unnamed_symbol7$$darryl_vault:
    ->  0x555555556509 <+171>: call   rcx
        0x55555555650b <+173>: test   al, al
        0x55555555650d <+175>: je     0x555555556570            ; <+274>
        0x55555555650f <+177>: lea    rsi, [rip + 0x3c0a]
    ```

1. Let's inspect `RCX` register:

    ```sh
    (lldb) reg r $rcx
     rcx = 0x00005555555562f6  darryl_vault`___lldb_unnamed_symbol4$$darryl_vault
    ```

1. To get a relative address:

    ```sh
    (lldb) im loo -a 0x00005555555562f6
      Address: darryl_vault[0x00000000000022f6] (darryl_vault.PT_LOAD[1]..text + 230)
      Summary: darryl_vault`___lldb_unnamed_symbol4$$darryl_vault
    ```

1. Go to this address (`0x1022f6`) in Ghidra:

    We can see a function that loads a constant string - `DARRYL_IS_THE_GREATEST` before making another call, let's try it as a password.

    ```asm
        001022f6 55              PUSH       RBP
        001022f7 48 89 e5        MOV        RBP,RSP
        001022fa 48 83 ec 10     SUB        RSP,0x10
        001022fe 48 89 7d f8     MOV        qword ptr [RBP + local_10],RDI
        00102302 48 89 75 f0     MOV        qword ptr [RBP + local_18],RSI
        00102306 48 8b 45 f0     MOV        RAX,qword ptr [RBP + local_18]
        0010230a 48 8d 35        LEA        RSI,[s_DARRYL_IS_THE_GREATEST_00106012]          = "DARRYL_IS_THE_GREATEST"
                 01 3d 00 00
        00102311 48 89 c7        MOV        RDI,RAX
        00102314 e8 96 10        CALL       FUN_001033af                                     undefined FUN_001033af()
                 00 00
        00102319 c9              LEAVE
        0010231a c3              RET
    ```

1. The password worked for 2nd and 3rd options but not for the 4th option (the one we actually need):

    ```sh
    Enter a number (1-4) to continue... 2
    Hey, Darryl, Im going to need your password real quick for this one. Enter it here for me: 
    DARRYL_IS_THE_GREATEST
    Okay Darryl, here you go! Your first pet was SMALL DARRYL
    ```

    ```sh
    Enter a number (1-4) to continue... 3
    Hey, Darryl, Im going to need your password real quick for this one. Enter it here for me: 
    DARRYL_IS_THE_GREATEST
    Okay Darryl, here you go! You were born on the high seas, like any real dread pirate should be
    ```

1. Following the flow from `FUN_00102b22` (for the 4th option) we can find address `0x102bcd` where the similar check is done:

    ```sh
                             LAB_00102bcd                                    XREF[1]:     00109237(*)  
        00102bcd ff d1           CALL       RCX
        00102bcf 84 c0           TEST       AL,AL
        00102bd1 74 61           JZ         LAB_00102c34
        00102bd3 48 8d 35        LEA        RSI,[s_Okay_Darryl,_here_you_go!_00106120]       = "Okay Darryl, here you go! "
                 46 35 00 00
    ```

1. Breaking on `0x102bcd` revelas the address for a new comparator function `0x1027f2` to look up in Ghidra:

    ```sh
    (lldb) b -a 0x0000555555554000+0x2bcd
    Breakpoint 1: where = darryl_vault`___lldb_unnamed_symbol11$$darryl_vault + 171, address = 0x0000555555556bcd
    (lldb) c
    Process 2087 resuming
    4
    Hey, Darryl, Im going to need your password real quick for this one. Enter it here for me: 
    AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
    Process 2087 stopped
    * thread #1, name = 'darryl_vault', stop reason = breakpoint 1.1
        frame #0: 0x0000555555556bcd darryl_vault`___lldb_unnamed_symbol11$$darryl_vault + 171
    darryl_vault`___lldb_unnamed_symbol11$$darryl_vault:
    ->  0x555555556bcd <+171>: call   rcx
        0x555555556bcf <+173>: test   al, al
        0x555555556bd1 <+175>: je     0x555555556c34            ; <+274>
        0x555555556bd3 <+177>: lea    rsi, [rip + 0x3546]
    (lldb) reg r $rcx
        rcx = 0x00005555555567f2  darryl_vault`___lldb_unnamed_symbol10$$darryl_vault
    (lldb) im loo -a 0x00005555555567f2
        Address: darryl_vault[0x00000000000027f2] (darryl_vault.PT_LOAD[1]..text + 1506)
        Summary: darryl_vault`___lldb_unnamed_symbol10$$darryl_vault
    ```

1. `0x1027f2` is a huge function, which start with a buffer filled with constant 39 bytes, which looks like our flag, but it turns out to be encrypted in some way, most likely just XORed. Also the are a bunch of nested function calls, most of them do nothing, but decryption should happend somewhere.

1. At some point at adress `0x102a8f` there is a call to library `memcmp` function which takes two buffes and a size of `39` bytes. Let's put a breakpoint on it and inspect memory that two registers (params of the functions) points to:

    ```sh
    Process 2087 stopped
    * thread #1, name = 'darryl_vault', stop reason = breakpoint 2.1
        frame #0: 0x0000555555556a8f darryl_vault`___lldb_unnamed_symbol10$$darryl_vault + 669
    darryl_vault`___lldb_unnamed_symbol10$$darryl_vault:
    ->  0x555555556a8f <+669>: call   0x555555556060            ; symbol stub for: memcmp
        0x555555556a94 <+674>: test   eax, eax
        0x555555556a96 <+676>: sete   bl
        0x555555556a99 <+679>: lea    rax, [rbp - 0xc0]
    
    (lldb) mem read $rsi -c 39
    0x555555572780: 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41  AAAAAAAAAAAAAAAA
    0x555555572790: 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41  AAAAAAAAAAAAAAAA
    0x5555555727a0: 41 41 41 41 41 41 41                             AAAAAAA
    (lldb) mem read $rdi -c 39
    0x5555555727d0: 4d 65 74 61 43 54 46 7b 64 34 72 72 79 31 5f 21  MetaCTF{d4rry1_!
    0x5555555727e0: 73 5f 74 68 33 5f 42 45 24 54 5f 50 49 52 40 54  s_th3_BE$T_PIR@T
    0x5555555727f0: 45 5f 33 56 33 52 7d                             E_3V3R}
    ```

    We can see our input and the flag!

1. That's it, thanks for reading !
