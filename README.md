# What is BOMB lab ?
Bomb lab is a famous challenge that is considered as one of the best steps to enter the realm of Reverse Engineering.

In this challenge, We have 2 files: BOMB (the binary itself) and a pdb file, We must crack the binary by finding all the nessessory passwords.

**Note that** the author **DrEvil** has created lots of bombs, and can distribute as many new ones as he pleases, If you download a second bomb, it may be a different bomb.
___
___

# **Tools**
- IDA-Free.
- Any Debugger (I will use x64dbg).

___
___

# **Running the binary**
First, we run the program:

![Runing the binary](Images/runing-the-binary.png) 

So we have 6 phases that require 6 passwords to crack.

___

## **Phase 1**
Now we open it in IDA and load the pdb file with it.

In the main function we see our previous output being printed then a "ReadLine" function which will be passed as an argument to "j_phase_1" function.

![alt text](Images/open-in-IDA.png)

Diving into phase_1, we see that it calls **"j_strings_not_equal"** function to compare our input with **"I am just a renegade hockey mom."** if the are equal it returns, if not, it calls **explode_bomb**

![alt text](Images/Phase-1.png)

Lets that as input:

![alt text](Images/Phase-1-defused.png)

So, our first password is "<span style="color:#00FFFF;">**I am just a renegade hockey mom.**</span>"

___

## **Phase 2**

Now we goes to **"Phase_2"** function, We see a call to **"j_read_six_numbers"** function, be entering it we see that it ensures that our input consists of 6 integers.

![alt text](Images/Phase-2-a.png) 

![alt text](Images/Phase-2-b.png)

After that a comparison happens between our first input and '1', then a loop starts at **"loc_14001210D"** to check for the rest of the input.

Now we some manipulation then a comparison with our input.

![alt text](Images/Phase-2-c.png)

To follow the sequence of inputs more easily, We can open x64dbg and set a break point there.

![alt text](Images/Phase-2-d.png)

The resulting sequence is "<span style="color:#00FFFF;">**1 2 4 8 16 32**</span>" which is the second password, Lets Try It

![alt text](Images/Phase-2-defused.png)

**That is great !!**

___

## **Phase 3**

By entering the **"phase_3"**, we see a call to **"j_sscanf"** function to ensure that our input Consists of 2 integer numbers, a comparison with 7 happends then it enters a "Switch-cases" function, after that a comparison with 5.

![alt text](Images/Phase-3-a.png)
![alt text](Images/Phase-3-b.png)

So, We know that we have 7 cases and our first input must be less than 5.


<span style="color:#00FFFF;">**How does the switch works?**</span>
- First, jumps to the case that matches our first input which is stored in eax. 
- Second, it preform repetitive subtraction with { 0x7E, 0x274, 0x24C, 0x2B0 }  from eax to ensure the existence of a certain value in it
- Third, it checks if the value calculated in eax equals to our input

 ![alt text](Images/Phase-3-c.png)

Depending on what we know, there could be more that one password, to know some of possible values, we can set a break point at the comparison and sees the value in eax.

<span style="color:#00FFFF;">**Some of these values are:**</span>
```
 5  -126
 4   0
 2   562
 1  -26
 0   602
```

___

## **Phase 4**

Getting into the next phase we see our usual **"j_sscanf"** function, so our input is 2 integers.

A comparison with '0xE' to make sure that our input is less that  or equal '14', then we have a call to **"j_func4"** function.

![alt text](Images/Phase-4-a.png)

By entering the **"j_func4"**, we can see that it is a recursion function that does some math manipulation and repeats until eax equals our first input then it returns

![alt text](Images/Phase-4-b.png)

After that a comparison happends to ensure that the second input and the return value from **"j_func4"** are "0xA".

Now, we know that our second input is 10 ('A' in hex), So to know the first input we could just follow the math to find out that it is '3'.

So, the forth password is " <span style="color:#00FFFF;">**3 10**</span> ", Let's try it.

![alt text](Images/Phase-4-defused.png)

**Well Done !!**

___

## **Phase 5**

In this phase we have 2 integer inputs as usual, After that a loop starts.

The first input is used as index to a hard-coded array, the first element in
the indexed array is compared to '0xF', if not it does some math operations then iterates to the next element, our first element must makes the loop iterates for 15 times ('F' in hex). 

![alt text](Images/Phase-5-a.png)

Following the math, We conclude that the first input is <span style="color:#00FFFF;">**5**</span> and the second is <span style="color:#00FFFF;">**115**</span> ('73' in hex).

![alt text](Images/Phase-5-b.png)

**let's try that**

![alt text](Images/Phase-5-defused.png)

**Good job !!**

___

## **Phase 6**

In this phase there is a recall to **"j_read_six_numbers"** function to ensure that our input consists of 6 integers, After that we encounter 3 loops.

The first one ensures that our 6 integer-inputs are between 1 and 6.

![alt text](Images/Phase-6-loop1.png)

The second one ensures that there is no duplication.

![alt text](Images/Phase-6-loop2.png)

The third one re-order our input's elements in another linked list.

![alt text](Images/Phase-6-loop3.png)

Now we know that our password is { 1, 2, 3, 4, 5, 6 }, we just need to know the order.

By checking the content of the nodes we see that 

![alt text](Images/Phase-6-a.png)  
![alt text](Images/Phase-6-b.png)

If we ordered our nodes by their values descendingly we get <span style="color:#00FFFF;">**5 4 3 1 6 2**</span>, Let's try it.

![alt text](Images/Phase-6-defused.png)

That is incredible !!

Don't be too happy, there's still the <span style="color:#FF0000;">**Secret_Phase**</span>.

___

## **Secret Phase**

Inside the **"phase_defused"** function, we see that it calls the previously mentioned **"j_sscanf"** function at the end of evey phase, it checks if one of our previous inputs was consisting of 2 integers and a string.

 If that exists, it checks if that string equals **"DrEvil"**, if true the secret phase is defused along with phase-6, If not, only phase-6 is defused.

![alt text](Images/Secret-phase-a.png)

To know which input it checks for, We can use x64dbg to see that the first arguments to the **"j_sscanf"** are **"3 10"** which were the password for phase-4

![alt text](Images/Secret-phase-b.png)

Now we know that phase-4 can trigger the secret phase if we used the **"%d %d %s"** format in our input.

We know
that  that our 2 integers must be (3 10) to make sure the phase-4 is defused, and the string must be "DrEvil" as previously mentioned.

So, Let's test that 

![alt text](Images/Secret-phase-triggered.png)

Great, We triggered it successfully, Now we have to defuse it.

The **"Secret_phase"** function firstly converts our input from ASCII to integer then compares it with '1' then with '0x3E9' (which is 1001 in decimal), then calls a function named **"j_func7"** and compares it's return value with '5', if equals then phase is defuse.

So, our input must be between (1 : 1001 ) and the return of the **"j_func7"** must be 5.

![alt text](Images/Secret-phase-c.png)

By getting into the **"j_func7"**, it's another recursion.

Like the one in phase four, it does some math manipulation and repeats until eax equals our input then it returns.

![alt text](Images/Secret-phase-d.png)

The brute-force is more difficult this time and the math is tricky, so let's simplify the math way.

The **"j_func7"** takes 2 arguments, arg1 is n1 and arg2 is our input.

First it compares between our input an the value in n1
- If > n1, it goes to the yellow code, which changes the value in n1 then recall the **"j_func7"**.
- If = n1, it goes to the blue code, which will set eax to 0 then ends the repetition.
- If < n1, it goes to the red code, which changes the value in n1 then recall the **"j_func7"**.

But how the changing in n1 works?

If we looked inside the n1 we can see that it is a tree that each node points to two other nodes.

As we see, n1 has a value of '36' (0x24 in hex) and two pointers to n21 and n22, which have the values of (8 and 50).

![alt text](Images/Secret-phase-e.png)

If we continued like that we can conclude that our tree could be illustrated like that 

![alt text](Images/Secret-phase-f.png)

Now how does the return value change?

- The yellow code makes the eax equals to ( eax + eax + 1), we will called right.
- The red code makes the eax equals to ( eax + eax ) , we will call it left.
- Thn blue code sets eax to '0' and ends the repetition.

Since these function calls are being made recursively, the math being applied to eax is executed LIFO (Last In First Out), executing the last modifications to eax first. 

The value in eax could be '5' like that:
- eax = 1 (0 + 0 + 1) --->  # Right
- eax = 2 (1 + 1    ) --->  # Left
- eax = 5 (2 + 2 + 1) --->  # Right

Which gives us the following order R --> L -->R in our tree, which gives us the value of '<span style="color:#00FFFF;">**47**</span>'.

Let's try this.

![alt text](Images/Secret-phase-defused.png)

Finally, we did it !!!

___
___

## **Passwords**

- **I am just a renegade hockey mom.**
- **1 2 4 8 16 32**
- **4 0**
- **3 10 DrEvil**
- **5 115**
- **5 4 3 1 6 2**
- **47**

___

## **conclusion**

I think that this one was as fun as it was tiring, I have learnt a lot from it and I hope you did to. 

Stay tuned for more reversing.

___
___

<p align="center"><span style="color:#00FFFF;">THE END</span></p>


___
___