## IA Summit Bingo!

*Note:* This is NOT live yet. Hold your horses. And tweets.

To play, tweet at [@iasBingo] and you will receive a bingo card with randomly generated squares and a link to the interactive version. 

Each bingo card square has a description (and accompanying hashtag) of something an attendee might encounter at the IA Summit.

To claim a square, take a photo and tweet it with the matching hashtag, mentioning [@iasBingo]. The bot will fill in the square with the photo and send you an updated bingo card. Fill out a complete row or column and, bingo, you win!

Meanwhile, the [leaderboard] will keep track of those closest to getting a bingo, list the latest dozen square submissions and, of course, those who have already done it!

*IAS-bingo was forked from [NICAR Bingo] and adapted for the [Information Architecture Summmit].*

### Local Development Setup ###

`bin/setup`

Will:

* python using homebrew
* virtualenv with pip
* create a virtualenv directory 've' which is in .gitignore
* activate ve environment
* install dependencies from `requirements.txt` with pip
* copy config-sample.json to config.json and open with `$EDITOR`, if set

# Getting started
(still unchanged from original repo)

*Note: [@NICARBingo] runs on an ubuntu server on digitalocean.com*

### Set up your server
Get the right libraries (You might need to apt-get some of these individually)
```sh
$ sudo apt-get update
$ sudo apt-get install -y mysql-server libmysqlclient-dev git python-pip python-dev phantomjs
$ sudo apt-get install python-mysqldb
$ sudo apt-get install mysql-server
```
Access the mysql shell with your login (your username might be root) and set up a mysql server named after the bingo game you want

```sh
$ mysql -u USER -pPASSWORD
mysql> create database nicarbingo
```
### Twitter
- Create a Twitter account
- Create New App (https://apps.twitter.com/)
- Generate the API Keys and Access Tokens (Make sure access level is Read, Write)
- Make note of the consumer key/secret, and the access token/secret

### HTML template
- Download this [repo]
- ```nicar-bingo/website/static/siteart/star.svg``` - SVG for the logo found here
- ```nicar-bingo/website/static/templates/card.html``` and ```leaderboard.html``` - The markup for player cards and leaderboard
- ```nicar-bingo/website/static/css/bingo.css``` - Where to adjust the style
- edit ```daemon.py``` in ``` nicar-bingo``` and change the website on **line 249** to the domain you want (Right now it's nicarbingo.com)

### Credentials
- Create a file called config.json and copy over the contents of config-sample.json
- Fill out config.json with the MySQL database details and the Twitter handle’s API keys
- Note: mysql default host might be 127.0.0.1 depending on your settings

### Creating the squares
- Create a [spreadsheet] of goals following the included header format and export as a CSV
- Note: There must be at least 24 goals.
- Note: Only columns A and B must be filled in. The other columns are optional.
- Note: It is recommended that hashtags be no longer than 12 characters or they get cut off in the cards.
- Export the spreadsheet as a CSV file (```nameofcsv.csv```) and save it into the downloaded repo folder

### Setting up your mysql server
- Upload the repo with all your new files over to your server.
- cd into the nicar-bingo directory
- Import the bingo mysql schema: 
    - ```$ mysql -u USER -pPASSWORD nicarbingo < bingo.sql```
- Import the goals from the CSV to your mysql server:
    - ```$ python load_goals.py nameofcsv.csv```

### Bringing over Flask, Twitter bot files, etc
*Note: make sure you're in the nicar-bingo directory*
```sh
$ sudo pip install virtualenv
$ virtualenv ve
$ . ve/bin/activate
$ pip install -r requirements.txt
```
**Wait, wait, wait. You need to adjust a python file-- uploading image data causes unicode problems. Sorry. **

Go to ```nicarbingo/ve/lib/python2.7/site-packages/twitter/api.py``` and comment out the second line: ```from future import unicode literals```

###Run those beautiful python files (for testing)
First tab
(*Note: make sure you're in the root/nicar-bingo directory*)
```sh
$ . ve/bin/activate
$ python daemon.py
```
Second tab
(*Note: make sure you're in the root/nicar-bingo directory*)
```sh
$ . ve/bin/activate
$ cd/website
$ python website.py
```
###Run those beautiful python files (forever)
```sh
$ . ve/bin/activate
$ nohup python daemon.py &
$ cd/website
$ nohup python website.py &
```

[leaderboard]:http://iasbingo.com
[NICAR Bingo]:https://github.com/andrewbtran/nicar-bingo/
[@iasBingo]:http://twitter.com/iasBingo
[Information Architecture Summmit]:http://iasummit.org
[Daniel McLaughlin]:http://www.twitter.com/mclaughlin
[David Putney]:http://www.twitter.com/putneydm
[Andrew Ba Tran]:http://www.twitter.com/abtran
[@NICARBingo]:http://www.twitter.com/nicarbingo
[experiment]:https://github.com/danielsmc/twitter-bingo
[hackathon]:https://blog.twitter.com/2014/hacking-journalism-at-the-mit-media-lab
[wrote]:http://hackingjournalism.challengepost.com/submissions/24265-news-bingo
[@mbtabingo]:http://www.twitter.com/mbtabingo
[leaderboard]:http://nicarbingo.com:5000/leaderboard
[new app]:https://apps.twitter.com
[spreadsheet]:https://docs.google.com/spreadsheets/d/1Ywr7XJ2QQVSeAvDBAfIo87fUYaVgj0NbOn5d4XkXMmA/edit?usp=sharing
[repo]:https://github.com/andrewbtran/nicar-bingo/archive/master.zip

## Provisioning ##

Provisioning uses chef (chef-solo via knife-solo/librarian-chef) which requires ruby. We suggest also using bundler.

### Setup ###

`cd provision`
`bundle`
`librarian-chef install`

### Run ###

1. Create your digital ocean account
2. Create a new ubuntu instance, version 14.04 at the time of this writing
3. Make sure to setup a key for the root user so you don't need to remember a password
4. Update `.ssh/config` with a new `Host` entry for your new server. It may look like these:

```
Host ias-bingo-server
  HostName <your new ip address>
  User deploy
  ForwardAgent yes
  IdentityFile ~/.ssh/id_rsa

Host ias-bingo-server
  HostName <your new ip address>
  User root
  IdentityFile ~/.ssh/do_id_rsa
```

5. Set your public keys in the `provision/data_bags/public_keys/keys.json` file
6. Update the `provision/data_bags/passwords/mysql.json` file to set your own password
7. `knife solo prepare root@ias-bingo-server`
8. `knife solo cook root@ias-bingo-server`

## Common Server Actions ##

### Update from the git repo (root) ###

```bash
$ cd /u/apps/ias-bingo/current
$ git pull origin master
$ restart ias-bingo-server
$ restart ias-bingo-daemon
```

### Tail the log files for errors (root) ###

```bash
$ tail -f /var/log/upstart/ias-bingo-server
```

and for the daemon:

```bash
$ tail -f /var/log/upstart/ias-bingo-daemon
```

### Clear the whole database of data (deploy) ###

```bash
cat /u/apps/ias-bingo/current/config.json # get the deploy user mysql password
mysql -u deploy -h 127.0.0.1 -P 3306 -p ias_bingo_production

truncate daub_tweets;
truncate goals;
truncate user_card_squares;
truncate users;
```

### Load goals (deploy) ###

```bash
. /u/apps/ias-bingo/shared/ve/bin/activate
python load_goals.py iasbingo.csv
```
