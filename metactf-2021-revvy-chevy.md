# Revvy Chevy - MetaCTF 2021

1. Let's load binary into Ghidra and navigate to `main` funaction at `0x101208`.

    Decompiled view of this function, after fixing some data types and renaming couple of variables looks like this:

    ```c
    int FUN_00101208(void) {
        char cVar1;
        int iVar2;
        char *pcVar3;
        size_t sVar4;
        long lVar5;
        long in_FS_OFFSET;
        char input_buf [64];
        long local_20;
        
        local_20 = *(long *)(in_FS_OFFSET + 0x28);
        __printf_chk(1,"What\'s the flag? ");
        input_buf._0_8_ = 0;
        input_buf._8_8_ = 0;
        input_buf._16_8_ = 0;
        input_buf._24_8_ = 0;
        input_buf._32_8_ = 0;
        input_buf._40_8_ = 0;
        input_buf._48_8_ = 0;
        input_buf._56_8_ = 0;
        pcVar3 = fgets(input_buf,0x40,stdin);
        if (pcVar3 == (char *)0x0) {
            puts("no!!");
            iVar2 = 1;
        }
        else {
            sVar4 = strcspn(input_buf,"\n");
            input_buf[sVar4] = '\0';
            lVar5 = 0;
            do {
                cVar1 = FUN_001011e9();
                input_buf[lVar5] = input_buf[lVar5] ^ cVar1 + (char)lVar5;
                lVar5 = lVar5 + 1;
            } while (lVar5 != 0x40);
                iVar2 = memcmp(input_buf,PTR_ARRAY_00104010,0x40);
                if (iVar2 == 0) {
                    puts("You got it!");
                }
            else {
                puts("That\'s not it...");
                iVar2 = 1;
            }
        }
        if (local_20 == *(long *)(in_FS_OFFSET + 0x28)) {
            return iVar2;
        }
    }
    ```

    As we can see here:

    `pcVar3 = fgets(input_buf,0x40,stdin);`

    `64` bytes are read from the input and stored into a an `input_buf`.

1. After that each byte in `input_buf` is XORed with value returned from `FUN_001011e9` + current itterator

    It's better viewed from the disassembly:

    ```
                            LAB_001012ae                                    XREF[1]:     001012c6(j)  
    001012ae b8 00 00        MOV        EAX,0x0
                00 00
    001012b3 e8 31 ff        CALL       FUN_001011e9                                     undefined FUN_001011e9()
                ff ff
    001012b8 01 d8           ADD        EAX,EBX
    001012ba 30 44 1d 00     XOR        byte ptr [RBP + RBX*0x1]=>input_buf,AL
    001012be 48 83 c3 01     ADD        RBX,0x1
    001012c2 48 83 fb 40     CMP        RBX,0x40
    001012c6 75 e6           JNZ        LAB_001012ae
    ```

1. And a decompilation of `FUN_001011e9` is next:

    ```c
    void FUN_001011e9(void) {
        DAT_0010402c = DAT_0010402c * 0x41c64e6d + 0x3039 & 0x7fffffff;
        return;
    }
    ```

    Here, some value from gloval variable stored at `0x10402c` (wich is initialized with `0` as shown below) is multipied by `0x41c64e6d`, summed up with another constant `0x3039` and bitwise added with `0x7fffffff`:

    ```
                                DAT_0010402c                               XREF[2]:     FUN_001011e9:001011ed(R), 
                                                                                        FUN_001011e9:00101201(W)  
    0010402c 00 00 00 00     undefined4 00000000h
    ```

1. After all of this is done the `memcpy` is called:

    ```c
    iVar2 = memcmp(input_buf,PTR_ARRAY_00104010,0x40);
    if (iVar2 == 0) {
        puts("You got it!");
    }
    ```

1. We can reverse all of this with a simple python script and capture the flag:

    ```python
    #!/usr/bin/env python3

    enc_bytes = [
        0x74, 0x1A, 0x95, 0x4E, 0xBA, 0xDB, 0x47, 0x64,
        0x09, 0x2D, 0xD1, 0xBF, 0x8A, 0x9D, 0xDE, 0x5A,
        0xD7, 0x5C, 0x93, 0x16, 0x09, 0x3B, 0x30, 0x6F,
        0x97, 0x40, 0xD0, 0x7C, 0x57, 0xDB, 0xDE, 0x0C,
        0x09, 0xA0, 0x84, 0x9B, 0x8A, 0x76, 0x2F, 0xB1,
        0x57, 0xA2, 0xE1, 0x4F, 0xB9, 0x6F, 0x81, 0xBF,
        0xB9, 0xBF, 0xE1, 0xEF, 0x79, 0xCF, 0x01, 0xDF,
        0xF9, 0x9F, 0xE1, 0x8F, 0x39, 0x2F, 0x81, 0xFF
        ]

    x = 0
    res = []
    for i in range(0, 64):
        x = ((x * 0x41c64e6d) + 0x3039) & 0x7fffffff
        r = ((x + i) & 0x000000ff)  ^ enc_bytes[i]
        res.append(r)

    print(bytearray(res).decode())
    ```

    ```sh
    MetaCTF{pr0p3r_encrypt10n_1snt_s0_e4sy...}
    ```

    That's it.
