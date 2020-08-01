# **H@cktivityCon CTF 2020**

<div align='center'>
  <img width=500 src='assets//images//logo.gif'>
</div>

This is my writeup for the challenges in H@cktivityCon CTF 2020, I'll try adding as many challenges as I can during the next few days, as of now it contains the only challenge I managed to write about during the CTF.


# Table of Content

# Web

## Ladybug
Want to check out the new Ladybug Cartoon? It's still in production, so feel free to send in suggestions!

Connect here:\
http://jh2i.com:50018

**`flag{weurkzerg_the_worst_kind_of_debug}`**

**Solution:** With the challenge we are given a url to a webserver:

![](assets//images//ladybug_1.png)

The page seems pretty bare, there are some links to other pages in the webserver and an option to search the website or to contact ladybug using a form, I first tried checking if there's an XXS vulnerability in the contact page or an SQLi vulnerability / file inclusion vulnerability in the search option, that didn't seem to work, then I tried looking in the other pages in the hope I'll discover something there, none of them seemed very interesting, but, their location in the webserver stood out to me, all of them are in the `film/` directory, the next logical step was to fuzz the directory, by doing so I got an Error on the site:

![](assets//images//ladybug_2.png)

This is great because we now know that the site is in debugging mode (we could infer that also from the challenge description but oh well), also we now know that the site is using Flask as a web framework, Flask is a web framework which became very popular in recent years mostly due to it simplicity, the framework depends on a web server gateway interface (WSGI) library called Werkzeug, A WSGI is a calling convention for web servers to request to web frameworks (in our case Flask).\
Werkzeug also provides a web server with a debugger and a console to execute Python expression from, we can navigate to the console using by navigating to `/console`:

`

![](assets//images//ladybug_3.png)

From this console we can execute commands on the server (RCE), let's first see which user we are on the server, I used the following commands for that:

```python
import subprocess;out = subprocess.Popen(['whoami'], stdout=subprocess.PIPE, stderr=subprocess.STDOUT);stdout,stderr = out.communicate();print(stdout);
```

the command simply imports the subprocess library, creates a new process which execute `whoami` and prints the output of the command, by doing so we get:

![](assets//images//ladybug_4.png)

The command worked!, now we can execute`ls` by changing the command in order to see which files are in the current directory, by doing so we see that there's a file called flag.txt in there, and by using `cat` on the file we get the flag:

![](assets//images//ladybug_5.png)

**Resources:**
* Flask: https://en.wikipedia.org/wiki/Flask_(web_framework)
* Flask RCE Debug Mode: http://ghostlulz.com/flask-rce-debug-mode/

## Bite
Want to learn about binary units of information? Check out the "Bite of Knowledge" website!

Connect here:\
http://jh2i.com:50010

**`flag{lfi_just_needed_a_null_byte}`**

**Solution:** With the challenge we get a url for a webserver:

![](assets//images//bite_1.png)

the site is about binary units, a possible clue for the exploit we need to use, it's seems we can search the site or navigate to other pages, by looking at the url we can see that it uses a parameter called page for picking which resource to display, so the format is:

`http://jh2i.com:50010/index.php?page=<resource>`

we possibly have a local file inclusion vulnerability (LFI), as I explained in my writeup for nahamCon CTF:

>  php allows the inclusion of files from the server as parameters in order to extend the functionality of the script or to use code from other files, but if the script is not sanitizing the input as needed (filtering out sensitive files) an attacker can include any arbitrary file on the web server or at least use what's not filtered to his advantage

let's first try including the php file itself, this will create some kind of a loop where the file included again and again and will probably causes the browser to crash...but it's a good indicator that we have an LFI vulnerability, navigating to `/index.php?page=index.php` gives us:

![](assets//images//bite_2.png)

index.php.php? it's seems that the php file includes the resource with a name  matching the parameter given appended to .php, lets try `/index.php?page=index` :

![](assets//images//bite_3.png)

It worked! so we know that we have an LFI vulnerability where the parameter given is appended to a php extension, this is a common defense machanism against arbitrary file inclusion as we are now limited only to php file, but there are ways to go around this.\
In older versions of php by adding a null byte at the end of the parameter we can terminate the parameter string, a null byte or null character is a character with the code `\x00`, this character signifies the end of a string in C and as such strings in C are often called null-terminated strings, because PHP uses C functions for filesystem related operations adding a null byte in the parameter will cause the C function responsible with including the resource to only consider the string before the null byte.

with that in mind let's check if we can use a null byte to display an arbitrary, to mix things let's try to include `/etc/passwd`, this file exists in all linux servers and is commonly accesible by all the users in the system (web applications running are considered as users in linux) and as such it's common to display the content of this file in order to prove access to a server, we can represent a null byte in url encoding using `%00`, navigating to `/index.php?page=/etc/passwd%00` gives us:

![](assets//images//bite_4.png)

It worked...but where is the flag located...\
At this point I tried a lot of possible locations until I discovered that the flag is in the root directory in a file called `file.txt` by navigating to `/index.php?page=/flag.txt%00` we get the flag:

![](assets//images//bite_5.png)

**Resources:**
* File Inclusion Vulnerabilities: https://www.offensive-security.com/metasploit-unleashed/file-inclusion-vulnerabilities/
* Payload all the things - File Inclusion: https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/File%20Inclusion
* Null byte: https://en.wikipedia.org/wiki/Null_character
* Null byte issues in PHP: https://www.php.net/manual/en/security.filesystem.nullbytes.php
