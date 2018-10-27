# Hack The Box - The Art Of Reversing - Writeup
I couldn't find any writeups about this particular challenge in english so i thought i might aswell write one up.
## The description and initial analysis
The description of the challenge is:
>This is a program that generates Product Keys for a specific Software Brand.

>The input is the client UserName and the Number of Days that the sofware will remain active on the client.
>The output is the product key that client will use to activate the software package.
>We just have the following product key 'cathhtkeepaln-wymddd'

>Could you find the corresponding Username say A and the number of activation days say B given as input?
>The flag you need to enter must follow this format: HTB{AB}

So we have to reverse engineer the string _"cathhtkeepaln-wymddd"_ and find what the original input was. Seems like some pretty standard stuff. 
Loading up the program we see a basic interface with a couple of input forms, the product key output, and the create key button.

![](https://i.imgur.com/IRVKl44.png)

We'll start by just doing some basic testing and see how the output is created based on the input:

![](https://i.imgur.com/Sn6SfZj.png)

Few things we can instantly note before starting to decompile the program:

- The output appears to have 2 independent strings, one for the username and one for the days
- The string generated from the username doesn't remove or add any characters, even keeping spaces in the key
- The username does not appear to effect the days string in any way

Te algorithm for generating the key based on the username, doesn't actually modify the characters in any way, purely moving them around. This opens up the possibility that the username we're trying to find, _"cathhtkeepaln"_, is just an anagram and not encrypted. We plot the string into an [anagram solver](https://anagram-solver.net/cathhtkeepaln) and lo and behold:

![](https://i.imgur.com/oyBqPPk.png)

That's a flag if i've ever seen one. Putting it into the program confirms that this is the correct string.

![](https://i.imgur.com/FBlGvyj.png)

## Actually decompiling the program
While the first string was possible to find without actually doing any real decompiling, there appears to be no real decernable pattern for the key based on the days, and for that we need to decompile. Loading up the file in IDA tells us that the program is a .net assembly binary, which means if we try to open it as a normal x86 the program is gonna look very empty. 

![](https://i.imgur.com/FjC3mzI.png)

Since .net programs compile to an intermediate language we can have a way nicer time decompiling it by using tools like dnSpy or ILSpy. I'll be using dnSpy cause i like it more. After decompiling the program we can find the method `buttonCreateProductKey_Click` which is in charge of calling the operations that create the key. 

As we correctly predicted the key is composed of 2 separate strings that are calculated in different ways, but since we've already found the username string we'll focus on how the date string is calculated.
![](https://i.imgur.com/o1umexZ.png)

The date string is calculated by taking the input in the days text box, storing it in the variable `num`, then creating a new string called `text2` that is calculated by sending `num` to the method `ToR(num);`

`ToR();` is a recursive algorithm that converts the input number to roman numerals. Below is illustrated how the algorithm would turn the number 1324 into roman numerals:

![](https://i.imgur.com/fxZ8umG.png)

Once the string has been converted to roman numerals it is then sent to the method `DoR();`. The  `DoR();` method starts by reversing the array, then it loops through and pushing all characters once to the right. This means that e.g a C turns into a D.

![](https://i.imgur.com/9K58Sx6.png)

If the days are, for instance, 311 it will turn into CCCXI in roman numerals, then turn into `jyddd ` as the key. Since we now know the algorithm we can reverse engineer the second part of the key, `wymddd`. If we move all the characters one spot to the left and reverse the string again we get the roman numerals: CCCNXV, which converted to regular numbers is 365. We now have the username + password and we double check it with the program:

![](https://i.imgur.com/Il9qzmi.png)

The key is then **_hacktheplanet365_** 

## Finding the username a real way

WIP shit's confusing 
