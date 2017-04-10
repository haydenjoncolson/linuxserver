# Linux Server Configuration on Amazon Lightsail

1. ssh into your server as ubuntu user
* Download the private key from the account page
* Copy the file to your local /.ssh/ directory ```bash
cp ~/Downloads/LightsailDefaultPrivateKey.pem ~/.ssh/Udacity.pem
```
* ```bash
ssh ubuntu@YourPublicIP -i
```
