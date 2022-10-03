# Jstet IAC

Ansible script I use to deploy some apps. Most of what this script does is done by an ansible role: [deploy_base](https://github.com/jstet/deploy_base. The idea behind the script is that it allows me to host multiple apps on one server, meaning I save hosting costs. 


## Set up

### Setting up the server 

1. Your server needs to meet the requirements describes in the README for the ansible role. I take care of that with a cloud config file "cloud_config/user_data". You also need to set up domains for grafana and plausible.

2. You need to set up DNS records that point your domain to your server. 

2. Change the IP in the hosts file (see hosts_example. rename to "hosts") 

3. Adjust the Ansible variables. For unecrypted vars see "/vars/main.yml". Add all domains you want to set up web apps for. For encrypted vars "vars/vault.yml" see this example:

- GRAFANA_PW_VAULT: pw
- SLACK_WEBHOOK_VAULT: https://hooks.slack.com/services/code

4. Run the main.yml script.

5. You should now have two files in the folder "output". You need these for the automatic deployment. Details later.

6. Uncomment `ssh_args=-o ForwardAgent=yes` to be able to access your private repos. Not necessary for public repos. I dont use this option for the main setup because it slows down script execution.

Now your server is ready for initial deployment of web apps and to set up automatic deployment with Github Actions.

### Initial Deployment

Requirement: your web app needs to be dockerized

1. In the folder "/templates" you can find some Docker Compose files. Use these as an example for writing your own. 
  - Containers that you want to be reachable from the outside should be in the network "proxy". 
  - Pay attention to the Traefik options. Adjust the domains, the port and replace he apps name with your own. E.g. traefik.http.routers.homepage -> traefik.http.routers.your_app

2. Adjust the existing or write your own tasks for deploying your web apps (see e.g. tasks/homepage.yml)

3. Adjust the deploy_app.yml to your needs and execute it

Now your web apps are deployed.

### Setting up automatic deployment

1. In the repo of your web app, create a folder called ".github" and in this folder a folder called "workflows". Create a .yml file and name it as you want your workflow to be called. For the contents of this file, see this [example](https://github.com/jstet/Exoplanets/blob/main/.github/workflows/ci.yml).

2. For the workflow to work, you need to add secrets to your repo. Got to Settings/Secrets/Actions and add the Secrets you used in your workflow file
  - HOST is the IP of your server
  - USERNAME is the user you want to use ssh with
  - SSHKEY is the private ssh key that should be in the folder "/output"


## Sources
- https://github.com/appleboy/ssh-action
- https://stackoverflow.com/questions/70282635/how-do-i-build-a-docker-image-from-a-sveltekit-app
- https://betterprogramming.pub/limit-resources-for-swarm-services-249dcebed833