# Far From The Weakest Shell - MetaCTF 2021

1. After unzipping you will have a script like this:

    ```powershell
    ( [ruNtimE.iNTErOpServiCes.mARSHaL]::pTRtOSTRINgUnI( [RUnTiME.INteroPSERVIceS.mArsHAL]::sEcUrEsTrINgTOgLObalaLLoCUNICodE($('$PAYLOAD' |CONvErtTo-secURESTRInG -k  (33..2)) ) )) | . ((VaRiAbLe '*MDr*').nAme[3,11,2]-JoIn'')
    ```

    *Here I replaced the payload with `$PAYLOAD`

    Here we can see 3 PowerShell function:

    `ConvertTo-SecureString`

    `SecureStringToGlobalAllocUnicode`

    `PtrToStringUni`

    I have not worked with a PowerShell before, so I'm not sure how exactly all of the APIs works, but I can assume that here the script just deobfuscates the payload and tries to run it as a command using pipe.

1. My thinking was that we can run the part that comes before the pipe to deobfuscate the payload:

    This part:

    ```powershell
    [ruNtimE.iNTErOpServiCes.mARSHaL]::pTRtOSTRINgUnI( [RUnTiME.INteroPSERVIceS.mArsHAL]::sEcUrEsTrINgTOgLObalaLLoCUNICodE($('$PAYLOAD' |CONvErtTo-secURESTRInG -k  (33..2)) ) )
    ```

    And sure enough it yields the reall code, after some prettyfying it looks like this:

    ```powershell
    Set-Variable -Name nU0 -Value ([typE]("{4}{2}{5}{0}{6}{1}{3}{10}{9}{8}{7}"-f 'TY.CrYP','ogrAp','ystEm','H','s','.seCUrI','T','oRIThM','Alg','h','y.hAs'));

    Set-Variable -Name Clob -Value ([TyPe]("{4}{5}{3}{0}{2}{1}"-F'OnV','r','eRtE','tc','SysteM.b','i'));

    SET-iTeM  vaRiabLe:OMz  (  [tYPE]("{1}{0}{2}" -f'con','SYSTEM.','vERT'));

    SEt-ITEM vARIABlE:pO52Ew ([typE]("{1}{4}{2}{3}{0}"-F'Ng','sYstEM','t.eNc','OdI','.teX')  ) ;

    sEt-itEm  ('va'+'ria'+'BLE:'+'2L36D')  (  [typE]("{0}{1}"-F'sTrin','G'))  ;  


    function w`FKh(${S`TR}) {
        Set-Variable -Name SBvn -Value ((  VaRIAbLe  ('nU'+'0') ).vAlUE::("{0}{1}"-f'Creat','e').Invoke(("{2}{0}{1}" -f'a25','6','sh')))
        Set-Variable -Name gchekrns -Value (${Sb`Vn}."COM`pUTeh`ASH"( ( ls VaRIAblE:Po52ew ).Value::"uT`F8".("{0}{1}" -f 'GetB','ytes').Invoke(${S`Tr})))
        Set-Variable -Name ANVI -Value ($CLOB::("{2}{1}{0}" -f'g','Strin','To').Invoke(${g`C`he`KrNs}))
        return ${an`VI}.("{1}{0}{2}" -f 'c','Repla','e').Invoke('-', '')
    }

    function yu`FAs(${Bs`Nz}, ${w`cin}) {
        Set-Variable -Name uRcnB -Value (&("{0}{1}{2}"-f 'Ge','t','-Item') ${B`SnZ})
        foreach(${_} in ${UR`cNB}.("{0}{1}{2}"-f 'G','etSubkeyName','s').Invoke()){ 
            Set-Variable -Name NaMe -Value (${U`RCnB}."NA`mE" + "\" + ${_})
            Set-Variable -Name GcHEKRNS -Value (.("{1}{0}" -f'fkh','w') ${NA`Me})
            if(${w`CIN} -eq ${g`chEk`RNS}){
                return &("{1}{0}" -f'h','wfk') ${_}
            }
        }
        return ("{0}{1}"-f 'N','OPE!')
    }

    function v1(${In}) {
        Set-Variable -Name keY -Value (&("{0}{1}" -f 'yu','fas') ((("{4}{2}{1}{3}{0}" -f 's','TWA','M:dQUSOF','REdQUClasse','HKL'))."Rep`lA`cE"('dQU',[stRInG][cHar]92)) ("{4}{12}{5}{2}{3}{7}{9}{15}{10}{6}{11}{0}{14}{8}{19}{17}{1}{18}{16}{13}"-f'EE9D','92C','CAF12F','68','E','ECC8','DE5','B064E','166AA','AC31','F531','B00A2','8','C0','F2','4','F5','1','F29B6','5724'))
        Set-Variable -Name Key -Value (( variablE  ("O"+"MZ")).vAlue::"tO`BAs`e6`4sTRINg"(  ( gEt-cHIldItEM ("varIa"+"bl"+"E:P"+"O52E"+"W")  ).VaLue::"uT`F8".("{0}{1}" -f'G','etBytes').Invoke(${k`Ey})))
        Set-Variable -Name CODE -Value (@(-7, -39, -16, 18, -12, 0, 7, -34, -5, -62, -38, -11, 1, -3, -17, -53, -18, -19, 46, 58))
        Set-Variable -Name iN -Value ((  geT-cHilDIteM  VariablE:oMZ).vaLuE::"TOba`SE`6`4StR`ing"(  (VAriablePO52eW -vAl )::"uT`F8".("{0}{1}{2}"-f'G','etB','ytes').Invoke(${I`N})))
        Set-Variable -Name iN -Value (${i`N}[0 .. ${i`N}."LE`NGTh"])
        foreach(${I} in (0 .. (${i`N}."l`EN`gTH" - 1))){
            ${In}[${I}] = [char]([Int]([char]${iN}[${I}]) + ${CO`dE}[${i}])
            if (${in}[${i}] -ne ${k`ey}[${i}]){
                return ${FA`L`se}
            }
        }
        return ${Tr`Ue}
    }

    function v`2(${I`N}) {
        Set-Variable -Name keY -Value (.("{1}{0}"-f 'ufas','y') ((("{3}{2}{4}{0}{5}{1}"-f 'SOFTWARE','TKClasses','T','HKLM:P','K','P')) -crePLAce([ChAr]80+[ChAr]84+[ChAr]75),[ChAr]92) ("{5}{3}{16}{10}{17}{7}{19}{1}{9}{12}{2}{13}{8}{0}{6}{14}{11}{15}{18}{4}" -f '4','9','B','F0B','1','7D','111','F','2','A','953BAF801F','F','C','A485','6','6CBD36CE','C456','DF5703C0','1','DB2A73A1759'))
        Set-Variable -Name Key -Value ((GEt-varIABle ('O'+'mz') -VAluEo )::"TO`BaSe6`4`sTRing"( $Po52eW::"ut`F8".("{1}{2}{0}" -f 's','G','etByte').Invoke(${K`Ey})))
        Set-Variable -Name COde -Value (@(-6, 20, -35, -17, -7, -1, -21, -49, 6, 1, -8, -33, -18, -3, -23, -56, -11, 3, 5, 7))
        Set-Variable -Name iN -Value ((VaRiAbLE OMz -Va)::"TO`B`AsE6`4stRING"( ( varIablE  ('Po'+'5'+'2eW') -VALU)::"u`Tf8".("{0}{1}{2}" -f'Ge','tBy','tes').Invoke(${In})))
        Set-Variable -Name In -Value (${iN}[0 .. ${In}."leN`g`Th"])
        foreach(${i} in (0 .. (${I`N}."lenG`Th" - 1))){
            ${in}[${i}] = [char]([System.Int32]([char]${I`N}[${I}]) + ${cO`DE}[${i}])
            if (${in}[${i}] -ne ${K`Ey}[${i}]){
                return ${f`ALSE}
            }
        }
        return ${Tr`Ue}
    }

    function V`3(${I`N}) {
        Set-Variable -Name kEY -Value (&("{0}{1}"-f'yufa','s') ((("{3}{4}{1}{5}{2}{0}"-f 'EvldClasses',':vldSO','AR','HKL','M','FTW'))  -cREpLacE'vld',[CHAr]92) ("{10}{15}{4}{7}{16}{3}{6}{8}{0}{14}{2}{5}{13}{1}{12}{11}{9}{17}"-f'4BAAB1','B','7024','144D8','BB','A','1A9','AA97ECB3DE22A3','3B','4','8E0F76163','1F','7','E','6B21','2547D','41','2'))
        Set-Variable -Name key -Value ((varIaBlE  oMz -vAluE  )::"tObA`se64`STrI`NG"(  $Po52ew::"ut`F8".("{2}{0}{1}"-f 'Byt','es','Get').Invoke(${k`eY})))
        Set-Variable -Name cODe -Value (@(-20, 35, -5, 0, -11, -20, 29, -52, -17, 52, 9, 3, 4, -15, -13, 35, -24, -33, 28, 61))
        Set-Variable -Name IN -Value ((gET-item  VARIaBlE:OmZ  ).valUe::"TOba`sE6`4s`TR`Ing"( (Get-VaRIaBlE  Po52ew  ).VAluE::"uT`F8".("{0}{1}{2}" -f'G','etB','ytes').Invoke(${IN})))
        Set-Variable -Name in -Value (${i`N}[0 .. ${iN}."len`Gth"])
        foreach(${I} in (0 .. (${IN}."Leng`Th" - 1))){
            ${I`N}[${i}] = [char]([System.Int32]([char]${I`N}[${i}]) + ${c`oDe}[${I}])
            if (${i`N}[${i}] -ne ${k`Ey}[${i}]){
                return ${F`ALsE}
            }
        }
        return ${TR`UE}
    }

    Set-Variable -Name INPUT -Value (.("{0}{2}{1}" -f'Read','st','-Ho') ("{4}{5}{0}{2}{3}{8}{1}{6}{7}"-f 't','eiv','h','e flag ','P','lease enter ','e y','our flag','to rec'))

    if (${i`N`PUT}."l`eNGTh" -eq 39) {
        ${pA`RT1} =   $2l36d::"J`oIN"("", ${i`NPUT}[ 0..12])
        ${p`Art2} =   (  GeT-vaRiaBLE ("2l3"+"6d") -valUeoNlY )::"j`OiN"("", ${in`P`Ut}[13..25])
        ${PA`Rt3} =  (  Gci  ('VA'+'riA'+'blE:'+'2l36D') ).vaLue::"jO`IN"("", ${INp`Ut}[26..39])
        if ((.('v1') ${P`Ar`T1}) -and (&('v2') ${p`ART2}) -and (.('v3') ${P`ARt3})) {
            .("{2}{1}{0}"-f '-Host','te','Wri') ((("{5}{9}{7}{6}{4}{2}{0}{8}{1}{3}"-f' why e','en ask',' flag, so','?','got the','Y',' clearly already ','e','v','ouo9tv')) -cReplaCE  'o9t',[cHAr]39)
        } else {
            &("{1}{3}{2}{0}" -f 'st','Write-','o','H') ("{1}{4}{2}{0}{3}" -f'y','No',' for ','ou.',' flag')
        }
    } else {
        &("{0}{2}{1}"-f'Writ','t','e-Hos') ("{2}{0}{1}{3}{4}"-f'a','g for yo','No fl','u','.')
    }
    ```

    Here we can see lots of code mangled using mix of upper and lower cases, some strings shuffling runtime type resolving and dynamic invokation. I didn't want to got deep and demangle all of this code so I decide to run it part by part.

1. At the near end we ca see ca see:

    ```powershell
    Set-Variable -Name INPUT -Value (.("{0}{2}{1}" -f'Read','st','-Ho') ("{4}{5}{0}{2}{3}{8}{1}{6}{7}"-f 't','eiv','h','e flag ','P','lease enter ','e y','our flag','to rec'))
    ```

    Running pieces like this:

    ```powershell
    ("{4}{5}{0}{2}{3}{8}{1}{6}{7}"-f 't','eiv','h','e flag ','P','lease enter ','e y','our flag','to rec')
    ```

    Yields the corresponging string (we can do this for all of the code):

    ```powershell
    Set-Variable -Name INPUT -Value (.("Read-Host") ("Please enter the flag to receive your flag"))
    ```

    We clearly see that that is a flag prompt.

1. Following same technique we can uderstand the flow better:

    ```powershell
    if (${i`N`PUT}."l`eNGTh" -eq 39) {
    ${pA`RT1} =   $2l36d::"J`oIN"("", ${i`NPUT}[ 0..12])
    ${p`Art2} =   (  GeT-vaRiaBLE ("2l3"+"6d") -valUeoNlY )::"j`OiN"("", ${in`P`Ut}[13..25])
    ${PA`Rt3} =  (  Gci  ('VA'+'riA'+'blE:'+'2l36D') ).vaLue::"jO`IN"("", ${INp`Ut}[26..39])
    if ((.('v1') ${P`Ar`T1}) -and (&('v2') ${p`ART2}) -and (.('v3') ${P`ARt3})) {
        ...
    }
    ```

    Here it basically checks if the lenght of the input is equal to `39`, if so, it splits the input into 3 variables: `part1` from `[0..12]`, `part2` from `[26..39]` and `part3` from `[26..39]`. And validaes each part by calling `v1`, `v2` and `v3` functions and passing the corresponding parts as a parameters.

1. Let's try to better understand what each functions is doing:

    Here is `v1`:

    ```powershell
    function v1(${In}) {
        Set-Variable -Name keY -Value (&("{0}{1}" -f 'yu','fas') ((("{4}{2}{1}{3}{0}" -f 's','TWA','M:dQUSOF','REdQUClasse','HKL'))."Rep`lA`cE"('dQU',[stRInG][cHar]92)) ("{4}{12}{5}{2}{3}{7}{9}{15}{10}{6}{11}{0}{14}{8}{19}{17}{1}{18}{16}{13}"-f'EE9D','92C','CAF12F','68','E','ECC8','DE5','B064E','166AA','AC31','F531','B00A2','8','C0','F2','4','F5','1','F29B6','5724'))
        Set-Variable -Name Key -Value (( variablE  ("O"+"MZ")).vAlue::"tO`BAs`e6`4sTRINg"(  ( gEt-cHIldItEM ("varIa"+"bl"+"E:P"+"O52E"+"W")  ).VaLue::"uT`F8".("{0}{1}" -f'G','etBytes').Invoke(${k`Ey})))
        Set-Variable -Name CODE -Value (@(-7, -39, -16, 18, -12, 0, 7, -34, -5, -62, -38, -11, 1, -3, -17, -53, -18, -19, 46, 58))
        Set-Variable -Name iN -Value ((  geT-cHilDIteM  VariablE:oMZ).vaLuE::"TOba`SE`6`4StR`ing"(  (VAriablePO52eW -vAl )::"uT`F8".("{0}{1}{2}"-f'G','etB','ytes').Invoke(${I`N})))
        Set-Variable -Name iN -Value (${i`N}[0 .. ${i`N}."LE`NGTh"])
        foreach(${I} in (0 .. (${i`N}."l`EN`gTH" - 1))){
            ${In}[${I}] = [char]([Int]([char]${iN}[${I}]) + ${CO`dE}[${i}])
            if (${in}[${i}] -ne ${k`ey}[${i}]){
                return ${FA`L`se}
            }
        }
        return ${Tr`Ue}
    }    
    ```

    After some cleanup:

    ```powershell
    function v1(${in}) {
        Set-Variable -Name key -Value (&("yufas") ("HKLM:\SOFTWARE\Classes" ("E8ECC8CAF12F68B064EAC314F531DE5B00A2EE9DF2166AA5724192CF29B6F5C0")))
        
        Set-Variable -Name key -Value (( Variable ("OMZ")).vAlue::"ToBase64String"(( Get-ChildItem ("varIablE:PO52EW")).VaLue::"UTF8".("GetBytes").Invoke(${key})))
        
        Set-Variable -Name code -Value (@(-7, -39, -16, 18, -12, 0, 7, -34, -5, -62, -38, -11, 1, -3, -17, -53, -18, -19, 46, 58))
        Set-Variable -Name in -Value (( Get-ChildItem  Variable:oMZ).value::"ToBase64String"(  (VariablePO52eW -val)::"UTF8".("GetBytes").Invoke(${in})))
        
        Set-Variable -Name in -Value (${in}[0 .. ${in}."length"])
        foreach(${i} in (0 .. (${in}."length" - 1))){
            ${in}[${i}] = [char]([Int]([char]${in}[${I}]) + ${code}[${i}])
            if (${in}[${i}] -ne ${key}[${i}]){
                return ${False}
            }
        }
        return ${True}
    }    
    ```

    Here we can see, that if constructs some registry key, pass it to `yufas` function and stores the results as `key` variable.

    Using globaly declared `OMZ` wich is a reference to `System.Convert` object it converts key into `base64` string.

    `$code` variable is declered as array of constant ints.

    Also function only parameter `$in` is also converted to `base64`.

    After that we can see a for-loop that itterates over the `$in`, sums it with a corresponding element from `$code` and checks if the result is equal to element from `$key` if no functions immediatly returns `False`.

1. Running the parts of `v1` seperatly:

    ```powershell
    Set-Variable -Name key -Value (&("yufas") ("HKLM:\SOFTWARE\Classes" ("E8ECC8CAF12F68B064EAC314F531DE5B00A2EE9DF2166AA5724192CF29B6F5C0")))
    
    Set-Variable -Name key -Value (( Variable ("OMZ")).vAlue::"ToBase64String"(( Get-ChildItem ("varIablE:PO52EW")).VaLue::"UTF8".("GetBytes").Invoke(${key})))
    ```

    It yields the `base64` that our input is compared against:

    `M0FBMUU3M0NFNEREQTkwOTY1REE1QjczNTM2NzQ1RjhEQzQwMTdFRkFBREVCRjRDODhERTQ4NDk3MDdFNDExRA==`

    We can now do the reverse, and subsctract `$code` from the `$key` to get the desired input:

    ```python
    #!/usr/bin/env/python3

    import base64

    def decode(key, code):
        user_in = [ord(k) - c for (k, c) in zip(key, code)]
        user_in = base64.b64decode(''.join([chr(x) for x in user_in]))
        return user_in


    flag = []

    # v1
    key = "M0FBMUU3M0NFNEREQTkwOTY1REE1QjczNTM2NzQ1RjhEQzQwMTdFRkFBREVCRjRDODhERTQ4NDk3MDdFNDExRA=="
    code = [-7, -39, -16, 18, -12, 0, 7, -34, -5, -62, -38, -11, 1, -3, -17, -53, -18, -19, 46, 58]
    flag += decode(key, code)


    # v2
    key = "ODE5MDc5OTJERDQ4MzBDOTYyMDM3MDk4OTlGRDE4NTRFOThGQjYwQUZEMkVGQUE1OERGNzI1ODMyMDdBRTUzRA=="
    code = [-6, 20, -35, -17, -7, -1, -21, -49, 6, 1, -8, -33, -18, -3, -23, -56, -11, 3, 5, 7]
    flag += decode(key, code)

    # v3
    key = "Njg0ODg4QzBFQkIxN0YzNzQyOThCNjVFRTI4MDc1MjZDMDY2MDk0QzcwMUJDQzdFQkJFMUMxMDk1RjQ5NEZDMQ=="
    code = [-20, 35, -5, 0, -11, -20, 29, -52, -17, 52, 9, 3, 4, -15, -13, 35, -24, -33, 28, 61]
    flag += decode(key, code)


    print(''.join([chr(x) for x in flag]))
    ```

    The flag is `MetaCTF{P0w3rSHELL_!$_the_literal_B35T}`
