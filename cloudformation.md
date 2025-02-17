How we IaC/CD as it stands today @Monowar Mukul 

Customer uses a self-hosted Bitbucket VM hosted in their legacy AWS organisation.

Access is restricted to Customer devices which in turn are locked down pretty hard. 

To accelerate things, we have opted to use vendor GitHub repos to store code and collaborate

Repos should be segregated by application/workload/tool


https://github.com/vendoraustralia/Customer-nonprod-db 



All repos use a standard Makefile structure for testing and deployment 

Workflow as follows

TBA Bootstrapping Cloudshell [to be expanded as part of next stage]

install yamllint, cfn lint 

this is important to test code, can be done locally 

Pip install works in Cloudshell 

so now we can install yamlint and cfnlint

pip install -r requirements.txt

pip install yamllint

pip install cfn-lint

 

python etc already in place.

Should be done once, preliminary acccount stuff has been completed. 

But how do we develop? 

set up SSH keys for Github 

ls -l .ssh 

create key in cloudshell

ssh-keygen -t ed25519

cat .ssh/id_ed25519.pub

Go back to github

click on your user icon, go to settings

ssh and gpg keys 

add new key

Give it a name 

copy the newly generated key 

save

go back to the repository

click code dropdown

choose ssh

copy the link

in cloudshell, cd to home directory 

git clone <ssh clone linke>



Create a branch for the work you’re doing (should be a <Jira ticket><jira title>)

git checkout <branch name> 

Git/dev workflow

Develop

Test

publish branch

Create PR and notify reviewers

Launch Cloudshell

in the management account. 

Leverage access key into the relevant account. 

please don’t screenshot aws access key stuff please!!

run the following commands as per access key page

Git clone via ssh

get the git clone URL from the github repo

in cloudshell, run git clone (SSH)

1st run, we always need to clone

Deploy

in Cloudshell, git clone, switch to branch, run make build

Verify - monitor in cloudformation console

Cloudformation Stack Naming Convention

examples:

thing1-database-rds

thing1-ec2-app

thing1-sftp

thing1-s3-customerdata

Integrating Environment

thing1-nonprod-database-rds

But how about the repo and how it’s set up with makefiles, config etc?

<config> folder contains the parameters

formatted as <accountid>.json

there are makefiles that execute code and pulls params from the config folder

https://makefiletutorial.com/ 

But how do I run it?

review the makefile, we’ve gone with a parameterised approach. 

the make file essentially runs shell commands to deploy the cloudformation pulling in parameters 
