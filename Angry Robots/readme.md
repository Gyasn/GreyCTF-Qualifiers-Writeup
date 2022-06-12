### Challenge Metadata

| Challenge Name       | Angry Robot            |
|----------------------|----------------------------------|
| Category             | Reverse Engineering              |
| MD5 (authentication.tar.gz)  | 965b5a18735af4bbfa81879e2cedc9cc |
| Solves               | 18                               |
| Description          | You have entered a top secret robot production facilities and your clumsy friend tripped the alarm. You and your friends are about to be "decontaminated". Luckily, you have unpacked the authentication firmware during previous reconaissance. Can you use them to override the decontamination process? |

### Methodology
We're given the challenge zip that contains 100 binaries and a service that we can connect to. In the description, we are also hinted that we may need to use [Angr](https://angr.io/) (this will prove useful later) 

#### Exploring the service 
> Since I don't feel like analysing 100 binaries right now, let's take a look at the service 

The service gives us this output :

``` bash
$ nc challs.nusgreyhats.org 10523
Unauthorized access detected.
Decontamination in 10 minutes
Do you wish to override? (y/n)
y
Password required for challenge code: a3f10dcecdf143a491de186e964bd2a3646c35c7fee694ae1feea4f238c13ec6
```
From this we can guess a few things, the binaries will output some sort of input (or not...) for this service and that we have a time limit of 10 minutes because of (Decontamination in 10 minutes) 

The service also asks us for random binaries each time you `nc` into it. 

> I guess we can't really do much without the inputs ...

#### Analyzing 1a0ac4eb514b129844e15c2fad569f523c5701e146fffa3d51d6e5868b304da3
> Guess I can't put this off forever 

Analyzing the first file - `1a0ac4eb514b129844e15c2fad569f523c5701e146fffa3d51d6e5868b304da3` manually , we see a few things of interest
1. The characters are validated against the custom char set defined in the file 
> ZVOi]7SsDHDARIPu5KGb34fgd_w2u=[5r75^u4dm5H[]CY8D`jEMU59l\\o
2. The input string len is 29
3. function that is called (sub_401209) 
4. sub_401176 that is called by sub_401209 ensures that the user input is not less than 31 or equal to 127 and it performs some arithmetic 
> ((a2 * a1 % 2930 + a1 + a2) % 73 + 48); [Full code snippet below]
5. The function that checks the input returns 0 when wrong 
6. The file returns 1 when the input is wrong 

##### Decompilation of main from IDA Pro 7.7
```c
_BOOL8 __fastcall main(int a1, char **a2, char **a3)
{
  char s[8]; // [rsp+0h] [rbp-30h] BYREF
  __int64 v5; // [rsp+8h] [rbp-28h]
  __int64 v6; // [rsp+10h] [rbp-20h]
  int v7; // [rsp+18h] [rbp-18h]
  __int16 v8; // [rsp+1Ch] [rbp-14h]
  unsigned __int64 v9; // [rsp+28h] [rbp-8h]

  v9 = __readfsqword(0x28u);
  *(_QWORD *)s = 0LL;
  v5 = 0LL;
  v6 = 0LL;
  v7 = 0;
  v8 = 0;
  fgets(s, 30, stdin);
  return (unsigned __int8)sub_401209(s) == 0;
}
```

##### Decompilation of sub_401209 from IDA Pro 7.7
```c
__int64 __fastcall sub_401209(__int64 a1)
{
  unsigned __int8 v2; // [rsp+1Bh] [rbp-55h]
  int i; // [rsp+1Ch] [rbp-54h]
  __int64 v4[4]; // [rsp+20h] [rbp-50h] BYREF
  __int64 v5[6]; // [rsp+40h] [rbp-30h] BYREF

  v5[5] = __readfsqword(0x28u);
  qmemcpy(v4, "ZVOi]7SsDHDARIPu5KGb34fgd_w2u", 29);
  qmemcpy(v5, "=[5r75^u4dm5H[]CY8D`jEMU59l\\o", 29);
  v2 = 1;
  for ( i = 0; i <= 28; ++i )
  {
    if ( (unsigned __int8)sub_401176((unsigned int)*(char *)(i + a1), (unsigned int)i) != *((_BYTE *)v4 + i)
      || (unsigned __int8)sub_401176((unsigned int)*(char *)(i + a1), (unsigned int)*((char *)v4 + i)) != *((_BYTE *)v5 + i) )
    {
      v2 = 0;
    }
  }
  return v2;
}
```

##### Decompilation of sub_401176 from IDA Pro 7.7
``` c
__int64 __fastcall sub_401176(char a1, int a2)
{
  if ( a1 <= 31 || a1 == 127 )
    exit(1);
  return (unsigned int)((a2 * a1 % 2930 + a1 + a2) % 73 + 48);
}
```

> Let's try analyzing the next binary - `1b2b71acb4caaa86f9b6d108537adc205997d23084f17a0741643eb216cfa264` to see if they're the same

#### Analyzing 1b2b71acb4caaa86f9b6d108537adc205997d23084f17a0741643eb216cfa264

Analyzing `1b2b71acb4caaa86f9b6d108537adc205997d23084f17a0741643eb216cfa264` manually, we can see similarities between the binaries, they do the above checks again except with a few differences. The rest of the code is the same. 

1. The input str length (in this case it's 30)
2. The custom char set
> UV@95aWRWXO5o`NLmFSFR_L=rZwc=n>?`tAR2Da9mGW4jG<x`1x:gD02EP99
3. The modulus number in the arithmetic that is performed on the string
> (a2 * a1 % 2666 + a1 + a2) % 73 + 48); 

##### Decompilation of sub_401209 
```c
__int64 __fastcall sub_401209(__int64 a1)
{
  unsigned __int8 v2; // [rsp+1Bh] [rbp-55h]
  int i; // [rsp+1Ch] [rbp-54h]
  __int64 v4[4]; // [rsp+20h] [rbp-50h] BYREF
  __int64 v5[6]; // [rsp+40h] [rbp-30h] BYREF

  v5[5] = __readfsqword(0x28u);
  qmemcpy(v4, "UV@95aWRWXO5o`NLmFSFR_L=rZwc=n", 30);
  qmemcpy(v5, ">?`tAR2Da9mGW4jG<x`1x:gD02EP99", 30);
  v2 = 1;
  for ( i = 0; i <= 29; ++i )
  {
    if ( (unsigned __int8)sub_401176((unsigned int)*(char *)(i + a1), (unsigned int)i) != *((_BYTE *)v4 + i)
      || (unsigned __int8)sub_401176((unsigned int)*(char *)(i + a1), (unsigned int)*((char *)v4 + i)) != *((_BYTE *)v5 + i) )
    {
      v2 = 0;
    }
  }
  return v2;
}
```

##### Decompilation of sub_401176
``` c
__int64 __fastcall sub_401176(char a1, int a2)
{
  if ( a1 <= 31 || a1 == 127 )
    exit(1);
  return (unsigned int)((a2 * a1 % 2666 + a1 + a2) % 73 + 48); // note the difference!
}
```

From these 2 binaries, we can come up with a hypothesis that the binaries take in a unique input and if it makes the program return 0, then that is our input to the service.  

#### Exploring Angr

> For this section, we'll be referencing `1a0ac4eb514b129844e15c2fad569f523c5701e146fffa3d51d6e5868b304da3`

It seemed apt to use Angr as we have 100 binaries to analyse and the name of the challenge also suggests to us to use it. 

Installing Angr is rather easy, we just do `pip install angr`. 

#### Using Angr

Referencing [John Hammond's video](https://www.youtube.com/watch?v=RCgEIBfnTEI) and the [writeup](https://github.com/Dvd848/CTFs/blob/master/2020_GoogleCTF/Beginner.md) he mentions, we can write some boiler plate code needed to start using Angr. 

Angr allows you to define your input as an unknown, while specifying the size of it and allowing you to add constraints in order for the symbolic executor to not implode on itself.

```python
# boilerplate code
import angr
import claripy

FLAG_LEN = 15
STDIN_FD = 0

base_addr = 0x100000 # To match addresses to Ghidra

proj = angr.Project("./a.out", main_opts={'base_addr': base_addr}) 

flag_chars = [claripy.BVS('flag_%d' % i, 8) for i in range(FLAG_LEN)]
flag = claripy.Concat( *flag_chars + [claripy.BVV(b'\n')]) # Add \n for scanf() to accept the input

state = proj.factory.full_init_state(
        args=['./a.out'],
        add_options=angr.options.unicorn,
        stdin=flag,
)
```

At this point, I was quite confused how BVS, BVV and claripy worked and also how to use the find and avoid parameters.  I looked for more videos and stumbled upon this one by [NUS greyhats](https://www.youtube.com/watch?v=mHbOo6p7mMo). 

Now with a bit more understanding, let me try to explain how Angr works. 

Simply put, `BVS` is an unknown whereby you can specify some constraints on, such as input size or characters matched. `Find` is the memory address or path you want the symbolic executor to take. `Avoid` is the memory address or path you want the symbolic executor to avoid. 


##### general framework for identifying memory addresses for `find` and `avoid` 

We can use angr more effectively by employing this mini framework that I've created through this challenge

1. Identify your success condition 
2. Work backwards from that success condition that you want to hit
3. Identify the line of code that affects the success condition, e.g. mov eax, 0
4. Find the other branch that the code will take in the event you don't run the success branch, this will be your avoid memory address
5. Your find address can be any line of code that runs after the success branch 

##### running Angr on this binary 

1. Identify success condition 
> We want the program to return 0

2. Work backwards 
> the return value of sub_401209 is causing the return value to change, in this case 1 will cause the program to exit with exitCode 0, whereas 0 will cause the program to exit with exitCode 1.

3. Identify line of code that we're interested in 
> We want to set eax to 1 in this function in order to let the program return 0.

4. Find the other branch 
> In this case it's the other branch is setting eax to 0, which we don't want. As that will cause the instruction `jz short loc_40139C` to jump to `loc_40139C` which makes the program return 1. The address that we want to avoid is `0x401305`

5. Setting a find address 
> I used the memory address `0x4013A1`, after the function returns. 


##### Putting it all together

```python
import angr
import claripy
import sys

base = 0x00400000
password_len = int(sys.argv[2]) + 1

project = angr.Project(sys.argv[1], main_opts = {"base_addr" : base})
password_chars = [ claripy.BVS(f"{i}", 8 ) for i in range(password_len )] // sets up the password "buffer"/unknown when trying to solve

password = claripy.Concat(*password_chars + [claripy.BVV(b"\n")])

state = project.factory.entry_state(
    stdin = password
)

for k in password_chars:
  state.solver.add(k > 31)
  state.solver.add(k < 127)

SM = project.factory.simgr(state)

# gotta change avoid and find to appropriate places
# for avoid, find the instruction where the checking function makes eax = 0 i.e. mov eax, 0
# for find, you can optimize this, but I chose the first basic block after the test al, al in main 
avoid = {0x401305}

find = {0x4013A1} 

SM.explore(find=find, avoid=avoid)

if (len(SM.found) > 0):
    for found in SM.found:
        print(found.posix.dumps(0))
```

Running this with `python ./sol.py 1a0ac4eb514b129844e15c2fad569f523c5701e146fffa3d51d6e5868b304da3 29` we get the input `s7k24zbu2o6nr5vzojh1d112s9cjn` for this binary. Great! Except we need it for all 100... 

> Being lazy I thought I could just run the same code on all binaries but this didn't work out obviously from our previous testing

##### allowing Angr to work on all binaries 
So we need to generalize Angr to run on all these binaries using some complex algorithm ... No. Let's return to monke. I decided to get the offsets manually for each binary that the service requests for. 

Knowing I needed to do this within the time limit, I tried to speed up Angr by reading their [docs](https://docs.angr.io/advanced-topics/speed) and found two low effort settings that I could enable immediately and ran the script on my host instead of a VM to squeeze all the computing power I could get. Unsurprisingly, this allowed me to answer all 5 passwords and get the flag. 

##### Angr script with faster flags

```python
import angr
import claripy
import sys

base = 0x00400000
password_len = int(sys.argv[2]) + 1
angr.options.FAST_MEMORY
angr.options.FAST_REGISTERS
project = angr.Project(sys.argv[1], main_opts = {"base_addr" : base})
password_chars = [ claripy.BVS(f"{i}", 8 ) for i in range(password_len )] // sets up the password "buffer"/unknown when trying to solve

password = claripy.Concat(*password_chars + [claripy.BVV(b"\n")])

state = project.factory.entry_state(
    stdin = password
)

for k in password_chars:
  state.solver.add(k > 31)
  state.solver.add(k < 127)

SM = project.factory.simgr(state)

# gotta change avoid and find to appropriate places
# for avoid, find the instruction where the checking function makes eax = 0 i.e. mov eax, 0
# for find, you can optimize this, but I chose the first basic block after the test al, al in main 
avoid = {0x401305} # remember to change this according to your binary

find = {0x4013A1} # remember to change this according to your binary

SM.explore(find=find, avoid=avoid)

if (len(SM.found) > 0):
    for found in SM.found:
        print(found.posix.dumps(0))
```

#### Solution 

1. Install Angr
2. Develop an Angr script (or you can copy mine)
3. Nc to the service 
4. Find the constraints and offsets necessary for `find` and `avoid` for the specific binary requested 
5. Run Angr on your binary with the appropriate options changed
6. Repeat steps 4 to 5 til the service gives you the flag
7. Get flag





