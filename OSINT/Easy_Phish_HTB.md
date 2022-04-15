I fist googled "protect website from phising emails" and after reading I came across various website talking about domain spoofing and how to prevent it. I finally came 
across two very useful sites which ended up helping me solve and understand this challenge. The first was  https://dmarc.org/overview/ which helped me understand what is 
DMARC, what is it based on and how it works in great detail and the second was https://knowledge.ondmarc.redsift.com/en/articles/1519838-looking-up-spf-dkim-and-dmarc-records-in-dns 
which helped me find the SPF and DMARC records. 

I ended up trying only the SPF and DMARC records as I didn't know what to put as selector value in order to check DMIK. My terminal looked something like this: 





![156899135-4b0d3d5b-8245-46dd-ae9d-ef8358c95ffc](https://user-images.githubusercontent.com/68371827/163560866-5eb0fdc9-1cc2-4933-8fe9-b57168235ceb.png)

...which was half the flag 


and then the other half...


![156899129-7e89b6ea-4af3-4010-9062-31ab1c2b0759](https://user-images.githubusercontent.com/68371827/163560899-ec3118ed-8b45-4b8d-afa1-a13bfe48b921.png)



so finally the flag is both of the 2 parts combined. 
