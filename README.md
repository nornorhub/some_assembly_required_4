# some_assembly_required_4

<!-----

Yay, no errors, warnings, or alerts!

Conversion time: 0.278 seconds.


Using this Markdown file:

1. Paste this output into your source file.
2. See the notes and action items below regarding this conversion run.
3. Check the rendered output (headings, lists, code blocks, tables) for proper
   formatting and use a linkchecker before you publish this page.

Conversion notes:

* Docs to Markdown version 1.0β33
* Sun Mar 27 2022 10:32:31 GMT-0700 (PDT)
* Source doc: Some Assembly Required 4
----->


This write-up is adapted from Martin Carlisle's solution. Check it out here: [https://www.youtube.com/watch?v=EsnzsnIN0YI](https://www.youtube.com/watch?v=EsnzsnIN0YI).

picoCTF

Some Assembly Required 4

Web Exploitation

200 points

I will skip over some steps because you should be familiar with them from the previous Some Assembly Required challenges. If you're not, try to solve them or google write-ups for them.

In Some Assembly Required 3, we had to reverse engineer the copy() function in the wat file to find the flag. However, in Some Assembly Required 4, the function we need to reverse engineer, check_flag(), is very complicated so we won’t do that. Instead, we will use [wasmtime](https://docs.wasmtime.dev/lang-python.html) to brute force the flag locally.

Disclaimer: follow the steps mentioned, line by line, or it won’t make sense.

We can see in the wat file that our input is being modified then strcmp is called so let’s set a breakpoint at the first line in strcmp in Chrome Dev Tools. After submitting “picoCTF{aaaaa}” and stopping at the breakpoint, we can examine the module memory in the buffer of type Uint8Array. We know that our input is stored at memory location 1072 from the previous challenges and the actual flag at memory location 1024 so we examine both locations. We can see that the first 8 characters, or values rather, match because the first 8 characters are part of the flag (“picoCTF{”). After experimenting with changing the values inside the input, we can see that changing one character changes the value of a pair of characters. For example, changing the previous input to “picoCTF{baaaa}”, changes the value of memory locations [1072+9] and [1072+10]. We decide to brute the flag, pair by pair, and check if the pair is correct using check_flag() and counting the matching values. check_flag() here is used because our input needs to be modified in a certain way so we can compare the values. This way is only implemented in check_flag(). Then, after check_flag() alters our input, we check if the pair is correct by comparing the values of the string at 1024 and 1072 and making sure they match. The code should make this clearer :)

Code: 


```
#!/usr/bin/python3
import string
import itertools

from wasmtime import Store, Module, Instance

store = Store()
module = Module.from_file(store.engine, "assembly4.wat")
instance = Instance(store, module, [])

exports = instance.exports(store)
memory = exports["memory"]
check_flag = exports["check_flag"]

def init_flag(pair):
   for i in range(len(flag)):
      memory.data_ptr(store)[1072+i]=ord(flag[i])
   for j in range(len(flag_so_far)):
      memory.data_ptr(store)[1072+8+j]=ord(flag_so_far[j])
   memory.data_ptr(store)[1072+8+len(flag_so_far)]=ord(pair[0])
   memory.data_ptr(store)[1072+8+len(flag_so_far)+1]=ord(pair[1])

def print_test_flag():
   g=''
   for j in range(1072,1072+8+2+len(flag_so_far)):
      g=g+chr(memory.data_ptr(store)[j])
   print(g)

def count_match():
   count = 0
   while(memory.data_ptr(store)[1024+count]==memory.data_ptr(store)[1072+count]):
      count=count+1
   return count

flag="picoCTF{0123456789abcdef0123456789abcdef}"
flag_chars=string.ascii_lowercase+"_"+string.digits+"}\x00"
flag_so_far=""
# flag has 33 characters, ignoring "picoCTF{", so 16 pairs and 1 leftover (17 pairs)
for i in range(17):
   # repeat=2 because we want to try all possible pairs
   for j in itertools.product(flag_chars,repeat=2):
      init_flag(j)
      check_flag(store)
      if(count_match()>=10+len(flag_so_far)):
         flag_so_far+=(j[0]+j[1])
         print("picoCTF{"+flag_so_far)
         break

