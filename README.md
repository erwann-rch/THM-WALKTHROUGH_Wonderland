# [THM-WALKTHROUGH] Wonderland

## *$ First Step : Enumeration*
We start to enumerate open ports with `nmap` and do a SYN scan, with `-sS`, over all 65536 ports, with `-p-`, to speed up the next scans.

![image](https://user-images.githubusercontent.com/52162856/214850344-eab03b51-bda3-4e0b-beac-8f253523eea9.png)

This scan indicate that only 2 ports (22 and 80) are open... This will considerably speed up the next, more complete, scan.

See ?! This scan is pretty aggressive but still executed quickly.
<ul>
<li> <code>-sC</code> : Use for default scripts of NSE.
<li> <code>-sV</code> : Try to get versions of the services running.
<li> <code>-p22,80</code> : Concentrate the efforts on the only two port that are open.
<li> <code>-o</code> : Write the output of the nmap can into a file.
</ul>

![image](https://user-images.githubusercontent.com/52162856/214854603-37fc7f9b-b8a4-4291-bae2-b63154136518.png)

With the second scan, we don't see much information, so let's start a `dirsearch` scan and during this time let's take a look at the web server running on the port 80.
It may take a time.
<ul>
<li> <code>-u</code> : IP/url of the target </li>
<li> <code>-t</code> : Number of concurrent threads </li>
<li> <code>-r</code> : Recursive mode </li> 
<li> <code>-o</code> : Output file </li> 
</ul>

![image](https://user-images.githubusercontent.com/52162856/214856271-9cbe4ea3-98c1-4e86-bd09-46e7bdf46a8e.png)

To get cleaner result to show in this walkthrough, I used gobuster too, but that's the same thing as dirsearch.

![image](https://user-images.githubusercontent.com/52162856/214859499-f13b6c3d-dfad-40e8-9d1c-04e95ce3e07b.png)

Here is the main page. Nothing relevant. 

![image](https://user-images.githubusercontent.com/52162856/214859422-f741894b-d85f-490b-9942-9523cbff4c3b.png)

Beside the `/img` that was revealed by the scan ... again nothing revelant in the source code.

![Capture](https://user-images.githubusercontent.com/52162856/214861047-7e74d5e0-d68f-4c9a-9f73-16c07c0bd19a.PNG)

Let's take a look to that file.

![image](https://user-images.githubusercontent.com/52162856/214861549-ff54b27b-c5e8-4fce-be58-ae354fb04174.png)

I tried to use `steghide` to see if their was any hidden message. And bingo, with an empty passphrase I got something.

![image](https://user-images.githubusercontent.com/52162856/214862420-bde1f29b-37c5-482c-ac1e-23098028dc1a.png)

I tried lasts two but nothing so let's take a look at this hint.

![image](https://user-images.githubusercontent.com/52162856/214862661-c2f582db-c172-4232-b6a9-6e185b6b2531.png)

Nothing came up to my mind at the moment, but I was sure this hint will somehow be useful during the game.

Now, I was going to the `/r` directory, on the web page.

![image](https://user-images.githubusercontent.com/52162856/214863706-dd293bdc-1a50-40d7-9e4e-b50666a73829.png)

Here I was kinda blocked during few minutes and then I realised, that I did some recursive enumeration on the directories and I found that : 

![image](https://user-images.githubusercontent.com/52162856/214864236-9b70ce42-81b4-416c-ada9-48844af7f195.png)

I tried and there is what I found:

![image](https://user-images.githubusercontent.com/52162856/214864430-8f0681f6-7e26-4448-8844-2fa5d1aa1baa.png)

This 'Keep Going' remind me the hint hidden in the .jpg I found so I tried ..  Fantastic ! Found something new.

![image](https://user-images.githubusercontent.com/52162856/214865479-76eef418-a102-4f71-b41c-783404ccbf55.png)

There it is ! Found Something that looks like credentials and I remembered SSH port was open so I decided to test them.

![Capture](https://user-images.githubusercontent.com/52162856/214866279-aead576c-0487-463f-80c7-2f0c00cba087.PNG)

I was able to get access to the machine.

![image](https://user-images.githubusercontent.com/52162856/214867124-c9b4bf52-79cf-466f-ba5c-2ebce614eb19.png)

## *$ Second Step : Privilège Escalation*
Tried some listing but was quickly facing a wall. 

![image](https://user-images.githubusercontent.com/52162856/214868287-e154867d-11ab-4738-8f91-0eff7631b012.png)

So I tried `sudo -l` command to see if I was able to escalate to root... Unfortunatly only a lateral PrivEsc is possible for now.

![image](https://user-images.githubusercontent.com/52162856/214868511-a1b09dcf-e4cd-41f7-9f66-49d3b0beebb6.png)

Let's see what that .py file is.

![image](https://user-images.githubusercontent.com/52162856/214869141-ff092317-c882-4a57-a580-a7951d959c4f.png)

We can an import so I though of python lib hijacking but wasn't sure at all.

![image](https://user-images.githubusercontent.com/52162856/214869528-2baefd97-19ec-4e79-8b93-c573b175e599.png)
Here we can see that the *.choice()* method is used so we can try to hijack this by creating a random.py in the current directory. 
I just put some very basic payload in it.
```python
import pty

def choice(input):
	pty.spawn('/bin/bash')
```
Try to execute it as rabbit ... And it worked !

![image](https://user-images.githubusercontent.com/52162856/214874267-13ed6991-ceee-4262-96ec-6feb0179a79e.png)

I found something interesting searching for files with SUID.

![image](https://user-images.githubusercontent.com/52162856/214875169-551306d5-99f7-45b1-9d83-b132a4780a08.png)

There is a binary in the wroking directory of rabbit.

![image](https://user-images.githubusercontent.com/52162856/214874836-47fb5de3-506e-412b-bf48-09f02381e272.png)

I decided to execute it but ... **Segmentation fault**, at the moment I guessed it was a buffer overflow so I wanted to know how this happened.
![image](https://user-images.githubusercontent.com/52162856/214877867-657365ff-1628-4507-8578-42e24444a64e.png)

After a little nc to copy the file to my machine, I started analysing it with `Ghidra`. (BTW, now the IP of the machine will be diffrent because I forget to add extra time to the machine and it shut down.)

![image](https://user-images.githubusercontent.com/52162856/214886638-ec114899-2420-487d-a617-48399b28e791.png)

I don't know much about C language but I get what just happened ... I lost time over a simple `puts` I was so much angry at myself to have been trolled by this :(
Anyway let's see the good thing, I saw that the program use `date` binary but with the command and not the absolute path so I thought it was possible to do the same thing as I did for the python lib. Plus I did a quick conversion on the setuid and get 1003... Guess what; it's hatter's uid !

![image](https://user-images.githubusercontent.com/52162856/214890485-8f2b6947-2531-4322-873e-b386cad6eb30.png)

I put :
```
#!/bin/bash

/bin/bash

``` 
into date file and made it executable with chmod. Then changing $PATH variable.

![image](https://user-images.githubusercontent.com/52162856/214895099-0647bdde-9193-4fa0-b6ec-427d5282d61a.png)

And Bingo ! 

![image](https://user-images.githubusercontent.com/52162856/214895457-5b5ba905-6e34-4c0e-8ca5-d05a0b5e6d73.png)

Found a password so I tried in ssh.

![Capture](https://user-images.githubusercontent.com/52162856/214900319-f0194c56-e66e-484f-928f-abf6668ba30a.PNG)

I tried to find any vector of escalation to root but got nothing on the SUID. To have a good conscience before trying automated script I search for capabilities.
![image](https://user-images.githubusercontent.com/52162856/214901130-8ef59475-04c6-44be-a0ec-6c5513d78473.png)

And guess what; I found something interesting !

![image](https://user-images.githubusercontent.com/52162856/214902542-7aa55d9d-7402-4c97-b542-3ce297b33cbb.png)

A quick search on GTFOBins for perl and I found something I can try.

![image](https://user-images.githubusercontent.com/52162856/214903162-c828cb3e-6239-4a01-ba6c-fe1ffdb5bf84.png)

So let's try !

![image](https://user-images.githubusercontent.com/52162856/214903476-94d82503-9af5-45d5-8a02-c2528fceca1b.png)

By the bless of God it worked !
I search few time where was the flag and now I get what the TryHackMe hint meant by "Everything is upside down here" : the flags are switched in the working directories of users !

![Capture](https://user-images.githubusercontent.com/52162856/214904681-ed63457f-1068-4afa-8c8e-4c6c6e89c5c2.PNG)

This room was impressive ! Hard enough to not be annoyed and easy enough to not abandon. Plus the searches on problems I encountered taught me a lot in linux PrivEsc.
