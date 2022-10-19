# How to deploy NodeJs Application on EC2 instance using GitHub actions

In the previous article, we learned how to deploy React Application using GitHub Action.
Since most applications have backend and NodeJs is one of the popular frameworks for developing backend for application.
So we will learn
how we can deploy NodeJs based application on AWS EC2 instane using GitHub Actions and automate our deployment.

Let's start with installing nodejs on ec2 instance by executing the following in your terminal
```
sudo apt install curl 
curl https://raw.githubusercontent.com/creationix/nvm/master/install.sh | bash 
```
The nvm installer script creates an environment entry to the login script of the current user. You can either log out and log in again to load the environment or execute the below command to do the same.

```
source ~/.bashrc   
```
let's install nodejs with newly installed nvm:

```
nvm install node
```

once the above command is executed you can verify your node installation by using

```
node -v
```
now let's install GitHub:

```
type -p curl >/dev/null || sudo apt install curl -y
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg \
&& sudo chmod go+r /usr/share/keyrings/githubcli-archive-keyring.gpg \
&& echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null \
&& sudo apt update \
&& sudo apt install gh -y
```

now generate an SSH public key and add that it to your GitHub repository.

let's move to GitHub repository and setup repository for actions (will create later).

**GitHub steps:**

1) copy contents of our private key to the GitHub secrets. and add hostname as well usename of ec2 instance into GitHub secrets.
2) create new action work flow on GitHub and type the following:

```
name: Deploy latest code to ec2 instance
on: [push]
jobs:

  build:
    name: Build
    runs-on: ubuntu-latest
    steps:
    - name: executing remote ssh commands using private key
      uses: appleboy/ssh-action@master
      with:
        host: ${{ secrets.HOST }}
        username: ${{ secrets.USERNAME }}
        password: ${{ secrets.PASSWORD }}
        script: |
        pm2 stop server.js
        git -C "your-target-directory" pull || git clone https://github.com/yogi-dad/github-actions-deploy-2-s3 your-target-directory 
        cd your-target-directory
        npm install
        pm2 start server.js
```

**Note:** in this tutorial we are using pm2 so please install pm2 using:

```
npm i -g pm2
```

since we are using nvm, and it's not installed at **/usr/local/bin** thus we need to create symbolic link for **npm** **node** and **pm2**.

```
sudo ln -s "$NVM_DIR/versions/node/$(node version)/bin/node" "/usr/local/bin/node"
sudo ln -s "$NVM_DIR/versions/node/$(node version)/bin/npm" "/usr/local/bin/npm"
sudo ln -s "$NVM_DIR/versions/node/$(node version)/bin/pm2" "/usr/local/bin/pm2"
```

That's it. once its done you can commit the workflow file and test your deployment.

Thanks for reading and Happy deployment!