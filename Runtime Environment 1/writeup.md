### Challenge Metadata

| Challenge Name       | Runtime Environment 1            |
|----------------------|----------------------------------|
| Category             | Reverse Engineering              |
| MD5 (gogogo.tar.gz)  | 5515f1c3eee00e4042bf7aba84bbec5c |
| Solves               | 28                               |
| Description          | GO and try to solve this basic challenge. FAQ: If you found the input leading to the challenge.txt you are on the right track |

### Methodology

> From the challenge description we are hinted that this is most likely a GO binary. We should first confirm our suspicions

A simple way to check if your binary is built with GO and also the version of it is to use this command `strings binary | grep 'go1\.'`

For me I got this `
        stack=[beEfFgGvcgocheckgo1.13.8infinityno anodereadlinkrunnableruntime.scavengestrconv.unknown( (forced) -> node= blocked= defersc= in use)
`
So from this we've enumerated that this is a GO binary and it's version is `1.13.8`.

> Since this is RE, let's open this in IDA/your favourite disassembler. I am using IDA Pro 7.7 as it has some IDAPython scripts that can make your life a bit easier: [AlphaGolang](https://github.com/SentineLabs/ AlphaGolang), [IDAGolangHelper](https://github.com/sibears/IDAGolangHelper) 

``` C
// main.main
void __cdecl main_main()
{
// IDA Pro's decompiler
  __int64 v0; // [rsp+10h] [rbp-98h]
  __int64 v1; // [rsp+18h] [rbp-90h]
  __int64 v2; // [rsp+20h] [rbp-88h]
  __int128 v3; // [rsp+20h] [rbp-88h]
  __int64 v4; // [rsp+28h] [rbp-80h]
  __int64 v5; // [rsp+30h] [rbp-78h]
  __int64 v6; // [rsp+38h] [rbp-70h]
  __int64 v7; // [rsp+40h] [rbp-68h]
  __int64 v8; // [rsp+48h] [rbp-60h]
  __int64 v9[4]; // [rsp+50h] [rbp-58h] BYREF
  __int64 v10; // [rsp+70h] [rbp-38h]
  __int64 *v11; // [rsp+78h] [rbp-30h]
  __int64 v12[2]; // [rsp+80h] [rbp-28h] BYREF
  __int64 v13[2]; // [rsp+90h] [rbp-18h] BYREF

  v11 = (__int64 *)runtime_newobject((__int64)&RTYPE_string);
  v13[0] = (__int64)&RTYPE__ptr_string;
  v13[1] = (__int64)v11;
  v4 = fmt_Fscanln((__int64)&go_itab__ptr_os_File_comma_io_Reader, os_Stdin, (__int64)v13, 1LL, 1LL);
  v8 = 4
     * (((__int64)(((unsigned __int128)((v11[1] + 2) * (__int128)(__int64)0xAAAAAAAAAAAAAAABLL) >> 64) + v11[1] + 2) >> 1)
      - ((v11[1] + 2) >> 63));
  v10 = runtime_makeslice((__int64)&RTYPE_uint8, v8, v8);
  v1 = runtime_stringtoslicebyte((__int64)v9, *v11, v11[1]);
  v5 = main_Encode(v10, v8, v8, v1, v2, v4);
  v3 = runtime_slicebytetostring(0LL, v5, v6, v7);
  v0 = runtime_convTstring(v3, *((__int64 *)&v3 + 1));
  v12[0] = (__int64)&RTYPE_string;
  v12[1] = v0;
  fmt_Fprintln((__int64)&go_itab__ptr_os_File_comma_io_Writer, os_Stdout, (__int64)v12, 1LL, 1LL);
}
```

<insert screenshot of imports>

> Most people should not have much experience reversing GO binaries (like myself) - but you can read up a bit by reading [this](https://medium.com/@nishanmaharjan17/reversing-golang-binaries-part-1-c273b2ca5333) (I chose this because I am using IDA)

> From that article we can tell that GO has many helper functions and compiler added code, these functions start with fmt and runtime respectively. 

From the functions defined in IDA, we can gather that the binary has 2 main functions, `main_main` and `main_Encode`. We also know from the disassembly that the file calls `main_Encode()` on the user input and then prints the result of the function using `fmt_Fprintln()`

> From here we can do 2 things, either take a deep dive into the `main_Encode` function or we can explore challenge.txt

I chose to do the latter and explore challenge.txt which looks like `GvVf+fHWz1tlOkHXUk3kz3bqh4UcFFwgDJmUDWxdDTTGzklgIJ+fXfHUh739+BUEbrmMzGoQOyDIFIz4GvTw+j--`

Hmm, seems a bit suspicious with the `--`, anybody familiar with base64 encoding would immediately raise some suspicion that this could be some custom base64 maybe(?)

> Now we'll explore the other route, trying to explore the `main_Encode` function

##### Interesting code snippets
``` C
qmemcpy(v21, "NaRvJT1B/m6AOXL9VDFIbUGkC+sSnzh5jxQ273d4lHPg0wcEpYqruWyfZoM8itKe", sizeof(v21));
```
An interesting string of length 64... 

##### Full decompilation from IDA Pro 7.7
``` C
while ( (__int64)v5 < 3 * ((__int64)a5 / 3) )
{
    if ( v5 >= a5 )
``` 
A for loop ... ?

``` C
*(_BYTE *)(a1 + v6) = *((_BYTE *)v21 + ((v7 >> 18) & 0x3F)); // note that 0x3F == 63 
```

``` C
// main.Encode
__int64 __usercall main_Encode@<rax>(__int64 a1, unsigned __int64 a2, __int64 a3, __int64 a4, unsigned __int64 a5)
{
  unsigned __int64 v5; // rax
  unsigned __int64 v6; // rcx
  unsigned __int64 v7; // r10
  unsigned __int64 v8; // r11
  char v9; // r10
  char v10; // r10
  char v11; // r11
  char v12; // r10
  unsigned __int64 v13; // rbx
  unsigned __int64 v14; // rdx
  unsigned __int64 v15; // r8
  char v16; // dl
  unsigned __int64 v17; // rdx
  char v18; // r8
  char v20; // dl
  __int128 v21[4]; // [rsp+10h] [rbp-48h] BYREF

  qmemcpy(v21, "NaRvJT1B/m6AOXL9VDFIbUGkC+sSnzh5jxQ273d4lHPg0wcEpYqruWyfZoM8itKe", sizeof(v21));
  v5 = 0LL;
  v6 = 0LL;
  while ( (__int64)v5 < 3 * ((__int64)a5 / 3) )
  {
    if ( v5 >= a5 )
      runtime_panicIndex();
    if ( v5 + 1 >= a5 )
      runtime_panicIndex();
    if ( v5 + 2 >= a5 )
      runtime_panicIndex();
    v7 = ((unsigned __int64)*(unsigned __int8 *)(a4 + v5) << 16) | ((unsigned __int64)*(unsigned __int8 *)(a4 + v5 + 1) << 8) | *(unsigned __int8 *)(v5 + a4 + 2);
    if ( v6 >= a2 )
      runtime_panicIndex();
    *(_BYTE *)(a1 + v6) = *((_BYTE *)v21 + ((v7 >> 18) & 0x3F));
    v8 = v7;
    v9 = *((_BYTE *)v21 + ((v7 >> 12) & 0x3F));
    if ( v6 + 1 >= a2 )
      runtime_panicIndex();
    *(_BYTE *)(v6 + a1 + 1) = v9;
    v10 = v8;
    v11 = *((_BYTE *)v21 + ((v8 >> 6) & 0x3F));
    if ( v6 + 2 >= a2 )
      runtime_panicIndex();
    *(_BYTE *)(v6 + a1 + 2) = v11;
    v12 = *((_BYTE *)v21 + (v10 & 0x3F));
    if ( v6 + 3 >= a2 )
      runtime_panicIndex();
    *(_BYTE *)(a1 + v6 + 3) = v12;
    v5 += 3LL;
    v6 += 4LL;
  }
  v13 = a5 - v5;
  if ( a5 == v5 )
    return a3;
  if ( v5 >= a5 )
    runtime_panicIndex();
  if ( v13 == 2 )
  {
    if ( v5 + 1 >= a5 )
      runtime_panicIndex();
    v14 = ((unsigned __int64)*(unsigned __int8 *)(a4 + v5) << 16) | ((unsigned __int64)*(unsigned __int8 *)(a4 + v5 + 1) << 8);
  }
  else
  {
    v14 = (unsigned __int64)*(unsigned __int8 *)(a4 + v5) << 16;
  }
  v15 = v14;
  v16 = *((_BYTE *)v21 + ((v14 >> 18) & 0x3F));
  if ( v6 >= a2 )
    runtime_panicIndex();
  *(_BYTE *)(a1 + v6) = v16;
  v17 = v15;
  v18 = *((_BYTE *)v21 + ((v15 >> 12) & 0x3F));
  if ( v6 + 1 >= a2 )
    runtime_panicIndex();
  *(_BYTE *)(v6 + a1 + 1) = v18;
  if ( v13 == 1 )
  {
    if ( v6 + 2 >= a2 )
      runtime_panicIndex();
    *(_BYTE *)(v6 + a1 + 2) = 45;
    if ( v6 + 3 >= a2 )
      runtime_panicIndex();
    *(_BYTE *)(v6 + a1 + 3) = 45;
  }
  else if ( v13 == 2 )
  {
    v20 = *((_BYTE *)v21 + ((v17 >> 6) & 0x3F));
    if ( v6 + 2 >= a2 )
      runtime_panicIndex();
    *(_BYTE *)(v6 + a1 + 2) = v20;
    if ( v6 + 3 >= a2 )
      runtime_panicIndex();
    *(_BYTE *)(v6 + a1 + 3) = 45;
  }
  return a3;
}
```

> This disassembly is quite complex... luckily we still have not exhausted all our explorable paths

Let's figure out if this is base64 or at least has some semblance to base64. After some digging, I found this [article](https://nachtimwald.com/2017/11/18/base64-encode-and-decode-in-c/). 


``` c
// code snippet from article
for (i=0, j=0; i<len; i+=3, j+=4) {
		v = in[i];
		v = i+1 < len ? v << 8 | in[i+1] : v << 8;
		v = i+2 < len ? v << 8 | in[i+2] : v << 8;

		out[j]   = b64chars[(v >> 18) & 0x3F];
		out[j+1] = b64chars[(v >> 12) & 0x3F];
		if (i+1 < len) {
			out[j+2] = b64chars[(v >> 6) & 0x3F];
		} else {
			out[j+2] = '=';
		}
		if (i+2 < len) {
			out[j+3] = b64chars[v & 0x3F];
		} else {
			out[j+3] = '=';
		}
	}
```

We can do some extrapolation and see the similarities between the decompilation and this code snippet from the article.

Taking a closer look at the last else block from the article, we can see that this is the code responsible for doing the padding. Taking a similar snippet from the decompilation, we can see that it is assigning 45 to the string array. Using python or looking up an ascii table, we can see that the value 45 maps to the character '-'.

``` c
    *(_BYTE *)(v6 + a1 + 2) = 45;
```

Now with some pretty strong proof that this block of code is base 64 encode with the custom substitution box at the start of the decompilation, while also taking note of the loop at the top. We can come up with a hypothesis and then test it.

#### Solution

Using the challenge.txt file as the input along with [CyberChef](https://gchq.github.io/CyberChef/#recipe=From_Base64('NaRvJT1B/m6AOXL9VDFIbUGkC%2BsSnzh5jxQ273d4lHPg0wcEpYqruWyfZoM8itKe',true,false)From_Base64('NaRvJT1B/m6AOXL9VDFIbUGkC%2BsSnzh5jxQ273d4lHPg0wcEpYqruWyfZoM8itKe',true,false)From_Base64('NaRvJT1B/m6AOXL9VDFIbUGkC%2BsSnzh5jxQ273d4lHPg0wcEpYqruWyfZoM8itKe',true,false)From_Base64('NaRvJT1B/m6AOXL9VDFIbUGkC%2BsSnzh5jxQ273d4lHPg0wcEpYqruWyfZoM8itKe',true,false)&input=R3ZWZitmSFd6MXRsT2tIWFVrM2t6M2JxaDRVY0ZGd2dESm1VRFd4ZERUVEd6a2xnSUorZlhmSFVoNzM5K0JVRWJybU16R29RT3lESUZJejRHdlR3K2otLQo), we can use the `From base64` blocks, 4 times with the custom substitution box given to us as `NaRvJT1B/m6AOXL9VDFIbUGkC+sSnzh5jxQ273d4lHPg0wcEpYqruWyfZoM8itKe`. Which will give us the flag, `grey{B4s3d_G0Ph3r_r333333}`