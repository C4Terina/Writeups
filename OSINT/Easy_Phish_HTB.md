I fist googled "protect website from phising emails" and after reading I came across various website talking about domain spoofing and how to prevent it. I finally came 
across two very useful sites which ended up helping me solve and understand this challenge. The first was  https://dmarc.org/overview/ which helped me understand what is 
DMARC, what is it based on and how it works in great detail and the second was https://knowledge.ondmarc.redsift.com/en/articles/1519838-looking-up-spf-dkim-and-dmarc-records-in-dns 
which helped me find the SPF and DMARC records. 

I ended up trying only the SPF and DMARC records as I didn't know what to put as selector value in order to check DMIK. My terminal looked something like this: 



![](images/2ndhalf.png)

...which was half the flag 

and then the other half...

![](images/1sthalf.png)


so finally the flag is `HTB{RIP_SPF_Always_2nd_F1ddl3_2_DMARC}`
