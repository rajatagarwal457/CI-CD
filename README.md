# CI/CD Pipeline Demo

This readme is roughly broken down into the following parts:

- Installing Jenkins
- Configuring jobs
- Making Pipeline
- Automating Pipeline

## Installing Jenkins

Use the following commands to install Jenkins on your system/cloud.

```
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt install jenkins
```

Once you have installed Jenkins start it with the following command:
```
sudo systemctl start jenkins
```

Allow Jenkins through the firewall:
```
sudo ufw allow 8080
```

Allow Jenkins to execute all sudo commands as we will need it.
Open Jenkins on localhost or your instance IP on port 8080 and finish the installation steps. Make sure you install the github plugins.

## Configuring Jobs

For this pipeline demo we will be using a very simple html page which is built and served using nginx and docker.

First click on New Item from the dashboard. Select freestyle project and give the name as Job1. You will be greeted with the config page. Under Souce Code Management, enter the url of your git repo. Enter the name of the branch you want to monitor bellow.

Then, click on Build -> Add build step -> Execute shell

Here we will add the shell commands we want to execute.
```
echo "code change detected"
```
Here, my Job1 just monitors for code change, hence a very simple shell.

Next, we will configure Job2. Follow all steps till execute shell. The shell for my Job2 was as follows:
```
echo "building container"
sudo docker build -t cicd_demo .
```
Here, a docker container is made from the Dockerfile in the github repository. Hence, scaling this project is not going to be complex at all, as long as your Dockerfile works, this project works.

Now for Job3, again, only the shell is different.
```
CONTAINER_PORT="8180"
OLD="$(sudo docker ps --all --quiet --filter publish=$CONTAINER_PORT)"
if [ -n "$OLD" ]; then
  sudo docker stop $OLD && sudo docker rm $OLD
fi
sudo docker run -it -d -p 8180:80 cicd_demo

echo "app deployed"
```
Here, I kill the current container running to free up the port, and then run the newly created container.

## Making Pipeline

Great! Now all our components have been created. We just need to put them together. 

Go back to the config page of Job1 (Job1 -> Configure)
Click on Post-build Actions -> Build other projects.
Type in Job2 and click save.

Repeat the same for Job2 and type Job3 in the Post-build Actions.

Now go back to your dashboard. Click on the plus sign above all the jobs.
Select Build Pipeline View. Enter any name for the pipe. Click OK.
On the next page, select Job1 as the initial job. Click OK.

Your pipeline is now set up. Clicking run will pull the code from git, compile it, and serve it.

## Automating Pipeline

Since I am working on localhost, configuring a githook that remotely triggers the pipeline is not possible. This can be done only on cloud instances. I created a cron job that monitors for changes every minute(For testing purposes. This time can be changed as per needed.)

To do this, go back to the config page of Job1. 
Under build triggers, select Poll SCM. 
In the text box bellow enter your cron schedule. For every minute I entered:
```
* * * * *
```
Click OK.

Done! Your pipeline is now automated for CI/CD. 



