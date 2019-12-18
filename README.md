## What is this repo?
This is a repo for a Telegram bot writen in python that uses aria2 to mirror files from the internet into Google Drive. This can be deployed onto a personal server using Docker or on Heroku.

## Features:
- Mirroring direct download links to Google Drive
- Mirroring torrent magnet links to Google Drive
- Mirroring downloads into the archived .tar format to Google Drive
- Upload progress
- Download/upload speeds and ETAs

## Disclaimer
This process wil take apporoximately 30 minutes to setup
You will need a Linux terminal to set this up. Windows user can also complete this using something called WSL (research this)
You will need to deploy this bot either on your own server using Docker or on Heroku using a free Heroku account
I do not reccomend using this bot on Telegram groups with large amounts of people as it will overload the bot
All of this tutorial will be done for Ubuntu systems

# Tutorial
### Pre-Requirements
Before beginning on the Terminal you will need to run the following commands:
```
sudo apt-get update
sudo apt-get upgrade
sudo apt install python3
sudo apt install npm
sudo apt install snapd
sudo apt install python-pip
sudo apt install git-all
curl https://cli-assets.heroku.com/install-ubuntu.sh | sh
```
Clone this repo:
```
git clone https://github.com/eliasbenb/TorrentBot/
cd TorrentBot
```
For WSL users:
- If you are on WSL you will need to install a file explorer
- You will also need to learn how to use that file explorer
- I reccomend this one:
```
sudo apt-get install mc
```
### Setting up config.ini
Now you will need to create a config.ini file. Run this command to copy my template into a new config.ini file
```
cp bot/config_sample.ini bot/config.ini
```
Open your file explorer and open ./bot/config.ini
Remove the second line saying:
```
_____REMOVE_THIS_LINE_____=True
```
Fill the rest of the fields with the information needed
Make sure to read all the notes below before filling it in
- BOT_TOKEN : Get BOT_TOKEN from @BotFather after creating your bot
- GDRIVE_FOLDER_ID : This is the end of a GDrive folder link
- DOWNLOAD_DIR : The path to the local folder where the downloads should be downloaded to, if you are doing this on Heroku set the path to: /home/username/mirror-bot/downloads
- DOWNLOAD_STATUS_UPDATE_INTERVAL : The time in seconds that the bot will update
- OWNER_ID : The Telegram user ID (not username) of the owner of the bot
- AUTO_DELETE_MESSAGE_DURATION : The time in seconds that the bot will keep a message before deleting it. Set to -1 to never delete messages

Note: You can limit maximum concurrent downloads by changing the value of MAX_CONCURRENT_DOWNLOADS in aria.sh. By default, it's set to 2
 
### Getting Google OAuth API credential file

- Visit the Google Cloud Console
- Go to the OAuth Consent tab, fill it, and save.
- Go to the Credentials tab and click Create Credentials -> OAuth Client ID
- Choose Other and Create.
- Use the download button to download your credentials.
- Move that file to the root of TorrentBot, and rename it to credentials.json
- Finally, run the script to generate token file (token.pickle) for Google Drive:
```
pip install google-api-python-client google-auth-httplib2 google-auth-oauthlib
python3 generate_drive_token.py
```
Note: for WSL users this won't work. You have to upload the credentials.json file to a cloud storage and download the file on WSL using 
```
wget www.insertlinkhere.com/downloadlink/credentials.json
```
### Deploying on Docker (SKIP IF DOING HEROKU)

- Start docker daemon (skip if already running):
```
sudo dockerd
```
- Build Docker image:
```
sudo docker build . -t mirror-bot
```
- Run the image:
```
sudo docker run mirror-bot
```

### Deploying on Heroku
- Login into your heroku account with command and follow on screen instructions:
```
heroku login
```
- Create a new heroku app:
```
heroku create appname	
```
- Select This App in your Heroku-cli: 
```
heroku git:remote -a appname
```
- Change Dyno Stack to a Docker Container:
```
heroku stack:set container
```
- Add Private Credentials and Config Stuff:
```
git add -f credentials.json token.pickle ./bot/config.ini
```
- Commit new changes:
```
git commit -m "Added Creds."
```
- Push Code to Heroku:
```
git push heroku master --force
```
- Restart Worker by these commands:
```
heroku ps:scale worker=0
```
```
heroku ps:scale worker=1	 	
```
Heroku-Note: Doing authorizations ( /authorize command ) through telegram wont be permanent as heroku uses ephemeral filesystem. They will be reset on each dyno boot. As a workaround you can:
- Make a file authorized_chats.txt and write the user names and chat_id of you want to authorize, each separated by new line
- Then force add authorized_chats.txt to git and push it to heroku
```
git add authorized_chats.txt -f
git commit -asm "Added hardcoded authorized_chats.txt"
git push heroku master
```
