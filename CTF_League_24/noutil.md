# Write up for Lalaland

## Overview

For this challenge we are trying to find a flag in random subfolder in a system with extremely limited utils. 
Running help, we see our available commands are:
```
job_spec [&]                          history [-c] [-d offset] [n] or hi>
 (( expression ))                      if COMMANDS; then COMMANDS; [ elif>
 . filename [arguments]                jobs [-lnprs] [jobspec ...] or job>
 :                                     kill [-s sigspec | -n signum | -si>
 [ arg... ]                            let arg [arg ...]
 [[ expression ]]                      local [option] name[=value] ...
 alias [-p] [name[=value] ... ]        logout [n]
 bg [job_spec ...]                     mapfile [-d delim] [-n count] [-O >
 bind [-lpsvPSVX] [-m keymap] [-f fi>  popd [-n] [+N | -N]
 break [n]                             printf [-v var] format [arguments]>
 builtin [shell-builtin [arg ...]]     pushd [-n] [+N | -N | dir]
 caller [expr]                         pwd [-LP]
 case WORD in [PATTERN [| PATTERN]..>  read [-ers] [-a array] [-d delim] >
 cd [-L|[-P [-e]] [-@]] [dir]          readarray [-d delim] [-n count] [->
 command [-pVv] command [arg ...]      readonly [-aAf] [name[=value] ...]>
 compgen [-abcdefgjksuv] [-o option]>  return [n]
 complete [-abcdefgjksuv] [-pr] [-DE>  select NAME [in WORDS ... ;] do CO>
 compopt [-o|+o option] [-DEI] [name>  set [-abefhkmnptuvxBCEHPT] [-o opt>
 continue [n]                          shift [n]
 coproc [NAME] command [redirections>  shopt [-pqsu] [-o] [optname ...]
 declare [-aAfFgiIlnrtux] [name[=val>  source filename [arguments]
 dirs [-clpv] [+N] [-N]                suspend [-f]
 disown [-h] [-ar] [jobspec ... | pi>  test [expr]
 echo [-neE] [arg ...]                 time [-p] pipeline
 enable [-a] [-dnps] [-f filename] [>  times
 eval [arg ...]                        trap [-lp] [[arg] signal_spec ...]>
 exec [-cl] [-a name] [command [argu>  true
 exit [n]                              type [-afptP] name [name ...]
 export [-fn] [name[=value] ...] or >  typeset [-aAfFgiIlnrtux] name[=val>
 false                                 ulimit [-SHabcdefiklmnpqrstuvxPRT]>
 fc [-e ename] [-lnr] [first] [last]>  umask [-p] [-S] [mode]
 fg [job_spec]                         unalias [-a] name [name ...]
 for NAME [in WORDS ... ] ; do COMMA>  unset [-f] [-v] [-n] [name ...]
 for (( exp1; exp2; exp3 )); do COMM>  until COMMANDS; do COMMANDS-2; don>
 function name { COMMANDS ; } or nam>  variables - Names and meanings of >
 getopts optstring name [arg ...]      wait [-fn] [-p var] [id ...]
 hash [-lr] [-p pathname] [-dt] [nam>  while COMMANDS; do COMMANDS-2; don>
 help [-dms] [pattern ...]             { COMMANDS ; }

```

## Solution

### Flag

It turns out we can find the flag using essentially just ```echo```. When we run ```echo */flag.txt```, it uses glob to find if flag.txt is in the current
directory, if it is, it prints its path. Then ```echo */*/flag.txt``` recursively searchs the current directory and immediate subdirectories. 
Thus, we can use an arbitrarily large number of ```*/```s to find the path. As it turns out, we need at least 3 to find it. Next, to print the flag, 
as we don't have ```cat```, we need to pipe the contents of the file to echo. Note that ```<```, opens the file, and ```$``` accesses the value of a 
variable. Thus ```echo $(<*/*/*/flag.txt)``` will find flag.txt and print out the flag contained within. Getting us the flag!




