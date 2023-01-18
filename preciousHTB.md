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
#### Analyze pdf file
I am using exiftool to see but there are more tools like this to check metadata of file. `exiftool filename` 

![pdfaly](https://user-images.githubusercontent.com/115337317/213100415-25078f33-68aa-4a6b-9956-eaf2cfe7d24d.png)

So we can see this file is converted by pdfkit v0.8.6. After doing simple search like `pdfkit v0.8.6 exploits` on google I found that this version of pdfkit is vulnerable to os injection (CVE-2022-25765).  source:https://security.snyk.io/vuln/SNYK-RUBY-PDFKIT-2869795



