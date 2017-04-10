# Linux Server Configuration on Amazon Lightsail

## Live Version
34.203.129.144

## Configuration

1. ssh into your server as ubuntu user
    * Download the private key from the account page
    * Copy the file to your local /.ssh/ directory
    ```bash
    cp ~/Downloads/LightsailDefaultPrivateKey.pem ~/.ssh/Udacity.pem
    ```
    * change permissions for user ubuntu
    ```bash
    chmod 600 ~/.ssh/Udacity.pem
    ```
    * ssh into server using private key
    ```bash
    ssh ubuntu@YourPublicIP -i ~/.ssh/Udacity.pem
    ```
2. Add new user grader with password grader
    ```bash
    sudo adduser grader
    ```
    * install finger to check if user grader is there
    ```bash
    sudo apt-get install finger
    finger grader
    ```
3. Give grader sudo access
* Copy sudoers.d file for grader
```bash
cp /etc/sudoers.d/90-cloud-init-users /etc/sudoers.d/grader
```
* Change ubuntu to grader
```bash
sudo nano /etc/sudoers.d/grader
```
* Copy root line, paste below and change root to grader
```bash
sudo visudo
```
4. Generate keygen in your terminal's home directory
```bash
