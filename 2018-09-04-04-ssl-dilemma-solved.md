# SSL Dilemma Solved

I received some assistance from my host today regarding the inability to generate a certificate because I did not register my domain
name with them.

I needed to temporarily point my domain to *their* public nameservers so that it'd look to them like my domain was legit. They call it
registering my domain as an external domain. I followed the instructions until I was able to obtain download links to 3 files:

- certificate file (.cer)
- intermediate certificate file (.cer)
- private key file (.key)

So I first downloaded them to my local machine, and then WinSCPed them to a convenient location on my VPS. From there, I had to rename
the files so that they're shorter. Then I had to concatenate the intermediate certificate file into the certificate file with the `cat`
commaned. I needed to be careful, because the -----END CERTIFICATE----------BEGIN CERTIFICATE----- had to be split in the middle to make
two lines, which the `cat` command just strung together in one line.

Then I made proper modifications to all of my Nginx server files to do the following:

- force all requests to be forwarded to https.
- add a port 443 server entry to each subdomain I have.

Code to be placed here later...

I restarted nginx.

I went back to my registrar and had my domain name point back to my custom name server on my VPS.

Now, when I go to the browser, users are directly forwarded to https!

If I understood correctly, my certificate is good for 1 year, but I can initiate a reissue any time to extend it, and I'm supposed to be
able to do that from my account control panel.