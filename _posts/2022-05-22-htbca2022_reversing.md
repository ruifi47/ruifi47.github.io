---
title: HTB Cyber Apocalypse CTF 2022 - Reversing - WIDE, Rebuilding, Without a Trace & Teleport writeups
author: ruifi47
date: 2022-05-22 18:00:00 +0100
categories: [CTF writeups, HackTheBox Cyber Apocalypse CTF 2022]
tags: [ctf-writeups, HackTheBox, HTBca2022, reverse-engineering]
img_path: /assets/img/posts/ctf-writeups/htbca2022/reversing/
---

# __WIDE__

### __Challenge Info__

> We've received reports that Draeger has stashed a huge arsenal in the pocket dimension Flaggle Alpha. You've managed to smuggle a discarded access terminal to the Widely Inflated Dimension Editor from his headquarters, but the entry for the dimension has been encrypted. Can you make it inside and take control?

For this challenge we are given an ELF 64-bit LSB pie executable x86-64 file called __wide__ and a file named __db.ex__.

When we execute the __wide__ file with __db.ex__ as a parameter we are prompted with the following:

![wide1](wide/wide1.png)

Since the challenge info refer to the Flaggle Alpha dimension, let's select its correspondent number (__`6`__).

![wide2](wide/wide2.png)
_Program output_

By selecting the dimension number __`6`__ (Flaggle Alpha), the program asks us for the `WIDE decryption key`.

Let's now open the executable file __wide__ in [Ghidra](https://ghidra-sre.org/ "Ghidra") to decompile the program.

After opening __wide__ in Ghidra and selecting the *main* function, let's go to the decompiler section to see if we can find some useful information.

![wide3](wide/wide3.png)
_main function in Ghidra_

In the *main* function we see some code responsible for displaying some error messages if we don't run the program with the right parameters and also some code responsible for printing the text that we see when the program is being executed.
In line 45, we can see that a function named *menu* is called so let's take a look at what that function does.

![wide4](wide/wide4.png)
_menu function in Ghidra_

Looking at the function *menu*, we can see that in line 152, a string comparison is being made using [`wcscmp`](https://www.cplusplus.com/reference/cwchar/wcscmp/ "wcscmp") to see if our input matches the string __`sup3rs3cr3tw1d3`__.
So now let's input the string __`sup3rs3cr3tw1d3`__ when the program asks us for the `WIDE decryption key`.

![wide5](wide/wide5.png)
_Program output the flag_

And we get the flag __`HTB{str1ngs_4r3nt_4lw4ys_4sc11}`__

<br>

# __Rebuilding__

### __Challenge Info__

> You arrive on a barren planet, searching for the hideout of a scientist involved in the Longhir resistance movement. You touch down at the mouth of a vast cavern, your sensors picking up strange noises far below. All around you, ancient machinery whirrs and spins as strange sigils appear and change on the walls. You can tell that this machine has been running since long before you arrived, and will continue long after you're gone. Can you hope to understand its workings?

The file that we get for this challenge is an ELF 64-bit LSB pie executable x86-64 called __rebuilding__.

When we execute the program with a password as a parameter we are prompted with the following: 

![rebuilding1](rebuilding/rebuilding1.png)

Looking at the output, we see that the program points out that the `Password length is incorrect`. So the first thing we are going to do is find the password length. Let's open the executable file __rebuilding__ in [Ghidra](https://ghidra-sre.org/ "Ghidra").

![rebuilding2](rebuilding/rebuilding2.png)
_main function in Ghidra_

Looking at the line 19 in the _main_ function above we can see a conditional statement to check `if (sVar1 == 0x20)`. 0x20 in hexadecimal is equal to 32 in Decimal. So the length of the password is 32 characters. Let's run the program with a random 32 characters password to see what happens.

![rebuilding3](rebuilding/rebuilding3.png)
_program output_

The program takes a few seconds `Calculating` and then returns `The password is incorrect`. Let's go back to Ghidra.

In line 38 of the _main_ function we see a conditional statement that check `if (local_14 == 0x20)` the program outputs _`The password is correct`_.

![rebuilding4](rebuilding/rebuilding4.png)
_Analyzing the main function_

Analyzing the variable `local_14` previous appearances in the _main_ function we note that in line 32 an array called `encrypted` is being XORed with a key.

![rebuilding5](rebuilding/rebuilding5.png)
_Analyzing the `local_14` variable_

By clicking on `encrypted` we can see some data in hex.

![rebuilding6](rebuilding/rebuilding6.png)
_Analyzing the encrypted data_

To quickly extract the hexadecimal data to build the `encrypted` array, select all the data of the picture above, right click and select _`Copy Special...`_.

![rebuilding7](rebuilding/rebuilding7.png)
_Copying the hex data_

Then select _`Byte String`_ and click _`OK`_.

![rebuilding8](rebuilding/rebuilding8.png)
_Copying the hex data 2_

We extracted the following encoded string in hex: __`29 38 2b 1e 06 42 05 5d 07 02 31 42 0f 33 0a 55 00 00 15 1e 1c 06 1a 43 13 59 36 54 00 42 15 11 00`__

Going back to the main function, by clicking on key we are able to view the string __`humans`__ that supposedly is used as a key to XOR the hex string above and decode the data.

![rebuilding9](rebuilding/rebuilding9.png)
_Finding the key `humans`_

Now we go to [CyberChef](https://cyberchef.org/ "CyberChef") and use the key __`humans`__ to try to decode the hex string extracted earlier.

![rebuilding10](rebuilding/rebuilding10.png)
_Decoding the hex string with the key `humans`_

As we can see from the picture above, it didn't work. Let's go back to Ghidra.

After analyzing more in-depth the program in Ghidra we find a very interesting function called *_INIT_1*.

![rebuilding11](rebuilding/rebuilding11.png)
_Finding the key `aliens`_

Looking at the assembly code on the left we can see that the __`humans`__ string is being replaced with the string __`aliens`__.  The decompiler section on the right helps us to better visualize each letter that is going to form this string. Let's test this string to see if we found the correct key to decrypt the hex data.

Go to CyberChef again and use the key __`aliens`__ to decode the hex string.

![rebuilding12](rebuilding/rebuilding12.png)
_Decoding the hex string with the key `aliens`_

And we get the flag __`HTB{h1d1ng_1n_c0nstruct0r5_1n1t}`__

<br>

# __Without a Trace__

### __Challenge Info__

> Draeger's mothership has suddenly vanished, he could be readying an attack! You need to track him down before disaster strikes...

The file that we get for this challenge is an ELF 64-bit LSB pie executable x86-64 called __without_a_trace__.

When we run the executable we are prompted with the following output:

![without_a_trace1](without-a-trace/without_a_trace1.png)

Let's run the executable again but now using [ltrace](https://www.tutorialspoint.com/unix_commands/ltrace.htm "ltrace") to see if we can get some useful information.

![without_a_trace2](without-a-trace/without_a_trace2.png)
_Executing the program with ltrace_

With ltrace we can see a string comparison the program makes between our input (`password1`) and the string __`IUCzus5b2^l2^tq^c5^t^f1f1|`__ to check if the password is correct.

Let's go to [CyberChef](https://cyberchef.org/ "CyberChef") and try to decode the string __`IUCzus5b2^l2^tq^c5^t^f1f1|`__.

![without_a_trace3](without-a-trace/without_a_trace3.png)
_Decoding the flag in CyberChef_

And we get the flag __`HTB{tr4c3_m3_up_b4_u_g0g0}`__

<br>

# __Teleport__

### __Challenge Info__

> You've been sent to a strange planet, inhabited by a species with the natural ability to teleport. If you're able to capture one, you may be able to synthesise lightweight teleportation technology. However, they don't want to be caught, and disappear out of your grasp - can you get the drop on them?

For this challenge we are given an ELF 64-bit LSB pie executable x86-64 stripped file called __teleport__.

When excuting the program with a password as a parameter, we get the following:

![teleport1](teleport/teleport1.png)
_Program output_

Let's open [Ghidra](https://ghidra-sre.org/ "Ghidra") to analyze the program.

To start our analysis we go to the Symbol Tree section in Ghidra where we can see all the functions.

![teleport2](teleport/teleport2.png)
_Symbol Tree section in Ghidra_

Let's analyze *FUN_00100b2a*.

![teleport3](teleport/teleport3.png)
_`FUN_00100b2a` in ghidra_

In this function we can see an if statement on line 11 to check if the character `t` is in the address `0x00303291`.

When we go through the other functions, we see that they have a similar logic to *FUN_00100b2a* where the if statement checks if a character is in the correct address.

Take for example *FUN_00100f6a*

![teleport4](teleport/teleport4.png)
_`FUN_00100f6a` in ghidra_

and *FUN_00100dd2*

![teleport5](teleport/teleport5.png)
_`FUN_00100dd2` in ghidra_

Looking at these functions we can see similar character checks to the one shown earlier.

When we double click on `DAT_00303281`, Ghidra takes us to the following part in the listing section:

![teleport6](teleport/teleport6.png)

We can see a range of addresses from `0x00303280` to `0x003032aa` in which each of those addresses refer to a function. 

As we seen earlier each function should contain a character that is being checked if it's in the correct address with an if statement.

If we click on the function (in green) next to each address on that part of the listing section (of the last picture), we get to see the respective function and the corresponding character present in the if statement.

By putting together all the letters in the functions starting in address `0x00303280` and ending on the `0x003032aa` address, we get the flag:

__`HTB{h0pp1ng_thru_th3_sp4c3_t1m3_c0nt1nuum!}`__