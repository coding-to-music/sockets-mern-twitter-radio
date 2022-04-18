# sockets-mern-twitter-radio

# 🚀 Javascript full-stack 🚀

## MERN Stack

### React / Express / MongoDB / Redux

https://github.com/coding-to-music/sockets-mern-twitter-radio

https://sockets-mern-twitter-radio.herokuapp.com

by Andrew Bateman https://github.com/AndrewJBateman

https://github.com/AndrewJBateman/mern-project

# Trunk Server

This is the software behind OpenMhz. For a while I had thoughts that I should see if I could make a business out of OpenMHz. I have come to realize though that I don't really want to run a business, I like building things a lot more.

In that vein, let's building some awesome things. Help me make OpenMHz better. Take the code and build a scanning site for your community. Add those features you have always been looking for. We can do better together!

- Luke

_If you are using this project as part of a business, please become a sponsor. A lot of time and effort went into building this_

## Notes

This code is pretty poorly commented, I am going to work on that. I am also going to use the Wiki to document the project. Please add to the documentation as you learn things.

There are a bunch of experiments lurking in the code. There is some code for adding in Stripe payments. I was going to role out the concept of paid accounts with additional features. There is also a half completed effort to allow more than one user to be associated with a system.

I haven't done a great job of keeping all the packages up to date... and I never got around to adding tests. Both of these would be great things for folks to go after.

## Path Forward

It would be great to get the code to a place where there is base code and people can add customization on top of that. They would probably fork the base code, add additional features and design and then rebaseline the code as new things are added to the mainline.

## Architecture

There are a lot of different components that make up the system. All of the server code is written in NodeJS and the frontend code uses React. Each of the different system components is run as a seperate container. A docker-compose script is used to start everything up. Right now, it is being operated on a single machine. It wouldn't be too hard to split it over a couple machines using Kubernetes. Semantic UI React is being used to create all of the UI components.

- **account**: this frontend / server handles user account creation and profiles. When a user logins, they are re-directed to this app.
- **admin**: a logged-in user uses this frontend / server to manage their systems.
- **backend**: this just a server with no web frontend. It provides the API for uploading, filtering and fetching calls. It should also handle all of the metdata around a call.
- **frontend**: the frontend / server that general public use to look through systems and listen to calls.
- **mongo**: all of the metadata around calls is stored here, along with the user and system information.
- **nginx**: proxies all of the calls to the correct server and handles the HTTPS certs.

## Easy Install

I put together an [Ansible Script](https://github.com/openmhz/trunk-server-install) to help make it easier to setup an OpenMHz server. You still need to get yourself a Droplet from Digital Ocean, a Domain Name and some storage. The scripts helps download and build everything.

## Operations

You can run things in 2 different modes, **test** and **prod**. The big difference is that **prod** expects all of the calls to be https and gets cranky when they are not. I have gotten **test** to run fine on my laptop, so that is probably a good starting point.

### DNS Entries

You should have a domain name pointing to the IP address of the server you are going to use.
CNAMEs also need to be created for the various services. Create the CNAMEs below with your DNS host:

- api
- account
- admin

After doing this, you should have the following domains: `api.domain.com`, `account.domain.com`, `admin.domain.com`

### S3 Storage

Currently, both **test** and **prod** expect to use S3-based storage instead of local storage. Switching to use local storage would be relatively easy - but for the sake of testing, let's just say use something S3-compatible. Make sure that ~/.aws/credentials has the credentials you'd like to use with your S3-compatible storage provider.. IE:

```
[default]
aws_access_key_id = [..]
aws_secret_access_key = [..]
```

### Configure

The configurations for both **test** and **prod** come from environment variable files that are read in before the containers are started.
Copy the example files and fill in the required info:

```bash
cp test.env.example test.env
cp prod.env.example prod.env
```

Fill in:

- MailJet information
- S3 information
- Site name
- Admin Email (The email will appear to come from this email. You should make sure it matches the domain MailJet is configured for)
- How many days calls will be archived for.... this is just a UI thing, you need to create some S3 rules to make sure they are deleted

### Scripts

`./docker-test.sh` - sets up docker-compose to run with the correct environment variables for testing

`./docker-prod.sh` - sets up docker-compose to run with the correct environment variables for production

### Service I Like

- 💻 I use Digital Ocean and they have been pretty great. If you need hosting, give them a try and use my referal code: https://m.do.co/c/402fa446f7a6
  You get $100 of credit to use in 60 days.

- 💾 If you need storage, give Wasabi a try. The have been mostly reliable and you can't beat the price: https://wasabi.com

- 📨 I use Mailjet... and you need to also. It makes it easy to send out email address confirmations: https://www.mailjet.com

- 🔒 I use Let's Encrypt. It works.

## Local Testing

### Local DNS

It helps to have similar subdomains mapping to localhost/127.0.0.1. I used the domain `openmhz.test`. Feel free to use this, or come up with something clever. If you do something different, make sure you use that instead in the commands below.

On MacOS, make the changes in: _/etc/hosts_

```
##
# Host Database
#
# localhost is used to configure the loopback interface
# when the system is booting.  Do not change this entry.
##
127.0.0.1       localhost
127.0.0.1       openmhz.test api.openmhz.test admin.openmhz.test media.openmhz.test
```

Then load these values into local DNS: `dscacheutil –flushcache`

- In root dir, run: `./docker-test.sh build`
- `./docker-test.sh up -d`

You can then browser to:

- openmhz.test
- admin.openmhz.test

_Interesting Note_: Safari 13+ does not like the **.test** TLD and doesn't seem to want to store cookies from the TLD. It seems to work fine in production when you are using a real TLD. I guess use Chrome for local testing or a different TLD for testing, like local.
https://stackoverflow.com/questions/62023857/sharing-cookies-across-test-sub-domains-in-safari-13-not-possible

### Debugging React Apps using Hot Reloading

If you are trying to make changes to any of react frontends, it is a huge pain to have to compile to site and rebuild the container each time you make a change. Instead, simply run the react app in development mode. This will work for the frontends for the:

- admin frontend
- account frontend
- frontend... frontend

First, start up all of the containers as described above. You will still the backend APIs they provide.

Now go into the respective sub directory for the component you are interested in and run:

```
source ../test.env
yarn install #only need to do this the first time, it installs the Node packages locally
yarn start
```

This should build the frontend and open a browser. In order to have all the cookies work correctly, you have to use the same domain name. Make sure you have setup the local domains as described above. Then goto the base domain, for me that is openmhz.test, at port 3000 `openmhz.test:3000`

## Managing MongoDB

MongoDB is used in the backend to store data. It is pretty fast, flexible and has worked well enough for me.
All of the files that MongoDB uses to store the DB are in the _/data_ directory, which gets mapped into the contianer.
Mapping this directory makes sure that the data persists each time you run the mongo container.

### Working with the MongoDB Container

From the Host OS run:

```bash
docker exec -i -t $(docker ps -a | grep mongo | awk '{print $1}') /bin/bash
```

There are a few scripts included with the container:

- **clean.js** This script removes all Calls that are over 30 days old
- **totals.js** Lists different system stats

### Compact a collection

When you run clean.js it doesn't actually remove the files off storge. You can use this command from the mongo cli tool.

First, launch the tool: `mongo`
Then switch to the scanner db: `use scanner`
And then the compact command on the **Calls** collection:

```
db.runCommand({compact:'calls'})
```

This blocks all calls to the DB, so the site will not work while this is being run.

### Add an Index

Adding an index will make it quicker to search calls by date, talkgroup and whether there are stars.

First, launch the tool: `mongo`
Then switch to the scanner db: `use scanner`
And then add an index:

```
db.calls.createIndex( {shortName: 1, time: -1,  star: -1, talkgroupNum: 1})
```

### Upgrading MongoDB

It is a huge pain to upgrade MongoDB in place. It turns out to be easier to dump backup of the database, wipe everything out and then restore into a new database for the latest version of Mongo.

Rough Playbook (use common sense, I may not have this exact):

- get into the shell of the mongo container
- `mongodump --db scanner —out /data/db/backup`
- exit contianer and go back to host machine
- `cd data/db`
- `rm *` erase everything... but not the sub-directories because that is where the backup is
- upgrade to the latest version of mongo
- build and launch the mongo container, which will create an empty DB
- get into the shell of the mongo container
- `mongorestore --db scanner --drop /data/db/backup/`

## Setting up Logging

I have had good luck with Loggly. Their free tier provides enough capabilities for most small sites.
The Doocker Logging Driver works well and is easy to install:

https://documentation.solarwinds.com/en/Success_Center/loggly/Content/admin/docker-logging-driver.htm

## Move a site to a new server

Here is the general list of things to do:

- Copy over ~/.aws
- Copy over ~/.secrets
- Do a mongodump on old machine
- Scp the mongo backup to the new machine
- Launch the MongoDB container on the new machine
- Do a mongorestore
- Launch all the conatiners
- Change the Floating IP to point to the new machine

## GitHub

```java
git init
git add .
git remote remove origin
git commit -m "first commit"
git branch -M main
git remote add origin git@github.com:coding-to-music/sockets-mern-twitter-radio.git
git push -u origin main
```

## Heroku

```java
heroku create sockets-mern-twitter-radio
```

## Heroku MongoDB Environment Variables

```java
heroku config:set

heroku config:set PUBLIC_URL="https://sockets-mern-twitter-radio.herokuapp.com"
```
