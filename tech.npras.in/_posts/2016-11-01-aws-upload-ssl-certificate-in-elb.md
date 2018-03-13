---
layout: post
title: "AWS: Uploading SSL Certificate in ELB"
excerpt: "Learn how to upload the SSL Certificate and attach it to ELB when manually copying the file contents in the ELB form isn't an option"
---

Recently while working on a client project, I faced an issue with regards to SSL certificates.

I was setting up an AWS Elastic LoadBalancer for a rails app. The client wanted the app to respond only to https requests. So the ELB was set to listen on port 443, and then forward the traffic to port 80 (http) of the app servers that are behind in a private subnet.

In the multi-page form that AWS provides to create an Classic ELB, there's a section where you have to give the SSL certificate details. This happens when you configure the port traffic as mentioned above. If you select 443 as the ELB port, you'll be asked to provide the SSL certificate details. You'll either have to choose an existing certificate from a dropdown, or give the certificate file details in the relevant textboxes.

Though you can easily paste the contents in the textbox - the second approach - doing it via the AWS CLI was nifty!

After setting AWS CLI locally, here's the command that does uploads the certificate to your AWS account:

```
aws iam upload-server-certificate --server-certificate-name myAppDomainName --certificate-body file://myAppCert.pem --private-key file://myAppPrivateKey.pem --certificate-chain file://myAppCaChain.pem
```

Once the server certificate is successfully uploaded, in the ELB's creation form, you'll be able to select this certificate from the dropdown using the name you gave in the above command - myAppDomainName.
