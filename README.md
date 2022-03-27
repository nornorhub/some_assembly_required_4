# some_assembly_required_4

This write-up is adapted from Martin Carlisle's solution. Check it out here: https://www.youtube.com/watch?v=EsnzsnIN0YI.

picoCTF
Some Assembly Required 4
Web Exploitation
200 points

I will skip over some steps because you should be familiar with them from the previous Some Assembly Required challenges. If you're not, try to solve them or google write-ups for them.

In Some Assembly Required 3, we had to reverse engineer the copy() function in the wat file to find the flag. However, in Some Assembly Required 4, the function we need to reverse engineer, check_flag(), is very complicated so we won’t do that. Instead, we will use wasmtime to brute force the flag locally.

Disclaimer: follow the steps mentioned, line by line, or it won’t make sense.

We can see in the wat file that our input is being modified then strcmp is called so let’s set a breakpoint at the first line in strcmp in Chrome Dev Tools. After submitting “picoCTF{aaaaa}” and stopping at the breakpoint, we can examine the module memory in the buffer of type Uint8Array. We know that our input is stored at memory location 1072 from the previous challenges and the actual flag at memory location 1024 so we examine both locations. We can see that the first 8 characters, or values rather, match because the first 8 characters are part of the flag (“picoCTF{”). After experimenting with changing the values inside the input, we can see that changing one character changes the value of a pair of characters. For example, changing the previous input to “picoCTF{baaaa}”, changes the value of memory locations [1072+9] and [1072+10]. We decide to brute the flag, pair by pair, and check if the pair is correct using check_flag() and counting the matching values. check_flag() here is used because our input needs to be modified in a certain way so we can compare the values. This way is only implemented in check_flag(). Then, after check_flag() alters our input, we check if the pair is correct by comparing the values of the string at 1024 and 1072 and making sure they match. The code should make this clearer :)
