- [Hosting a Discord Bot on Linux VPS](#hosting-a-discord-bot-on-linux-vps)
  * [Python](#python)
    + [Requirements](#requirements)
      - [What will you need](#what-will-you-need)
      - [Instalation](#instalation)
    + [Starting Bot](#starting-bot)
  * [JavaScript](#javascript)
    + [Requirements](#requirements-1)
      - [What will you need](#what-will-you-need-1)
      - [Instalation](#instalation-1)
    + [Starting Bot](#starting-bot-1)
  * [How to keep bot "up" after ending SSH session](#how-to-keep-bot--up--after-ending-ssh-session)
    + [Using Screen](#using-screen)
    + [System.d Service](#systemd-service)
      - [Creating service](#creating-service)
      - [Starting and Stopping service](#starting-and-stopping-service)
      - [Usefull Commands to services](#usefull-commands-to-services)

# Hosting a Discord Bot on Linux VPS

In this article you will learn:

* What you need to host the bot in Javascript or Python
* How to make bot run after closing SSH session

In this article we assume that:

* Have written and confirmed your bot is working
* Know how to establish SSH connection
* Know how to use FTP/SFTP to upload files


## Python

### Requirements

#### What will you need

* Python (in this article it's assumed that bot is written using Python 3.7, but commands will ussualy be same across board)
* Discord library - there are 2 most commonly used libraries, there are some diffrences between them in way they are started, I will it on both Disco and Discord.py
* VPS running Linux

#### Instalation

* Python  
  You need to login to your server using SSH, after that it is worth to check if you have required version of python already installed using command like `python3.7` if you will see screen similar to this:  
  ![alt text](https://cat.girlsare.life/6LQLQ81.png "Python 3.7 interpreter")  
  it means that you have this specific version of Python installed, to exit interpreter input `exit()` and press enter  
  
  If your needed version of Python isnt installed use commands  
  ```
   sudo add-apt-repository ppa:deadsnakes/ppa
   sudo apt update
   sudo apt install python3.7
  ```
  To explain what those commands do  
  `apt update` updates list of software avaible on repositories allowing to download them  
  `add-apt-repository ppa:deadsnakes/ppa` Adds deadsnakes repository which hosts Python builds, including newest ones  
  `apt install python3.7` Installs Python in version 3.7, if you need diffrent version simply replace 3.7 in command with version you need, in example for 3.5 command will look like `apt install python3.5`  
    
    You probably noticed that every command is prefixed with `sudo` this means that they will be executed with Root privilages, if you use your account not default root it will ask for your password, dont worry its normall  
    
    
* Discord library  
  Depending on library you use you neeed diffrent command, what is important to know is that you need to specify Python version you want to use, unlike windows Linux comes with Python 2.x preinstalled which is default to pip.
  
  1. Installing Discord.py library:  
      Assuming you kept SSH connection open, and made sure your needed Python version is installed now you need to install library you use for your bot, in this example we will use discord.py
      
      Discord.py has two variants, one with voice support and one without, commands to install them differ minimally
      **Library with Voice Support**
      To install discord.py with voice support use this command:  
      `
      python3 -m pip install -U discord.py[voice]
      `  
      If you ever sued pip on windows you might be suprised at lenght of this command, lets break it apart:   
         `python3` specifies that this is command for Python 3, not installed by default on ubuntu Python 2, it can be precised even more if you type `python3.7` for example, but generally there is no need for that
         `-m` tells python we intend to run a specific module, not a .py file  
         `pip install` tells python we want to install library using `pip` module (this is why we need -m flag)  
         `-U` turns following string into unicode
         `discord.py[voice]` is simply name of library
         
       **Library without Voice Support**  
       As i mentioned earlier command is really similar, in reality it is basically same apart from lack of `[voice]` part.  
       So, if you want to install discord.py without voice support you should use:  
       `
       python3 -m pip install -U discord.py
       `  
       Since command is practically same as previous one, i will not specify which part mean what, since nothing changed apart from library name
   2. Installing Disco as library  
      So, you wrote your bot using disco library and now need to download Disco onto VPS, procedure is basically same as with discord.py so open SSH connection and use command:  
      `
      python3 -m ip install disco-py
      `  
      and you are done!

### Starting Bot
   So now that we have baseline prepeared, lets get to getting bot up, There are few ways to do that per library, i will show two that i used personally for testing purposes.
   
   1. Discord.py bot  
      First i want to begin with showing example code i will be working on in this tutorial, it is basically boilerplate for logging into discord
```python
import discord
import asyncio
from discord.ext.commands import Bot
from discord.ext import commands
client = Bot(description="The GalaxyGate tutorial!", command_prefix="GG.", pm_help=False)
@client.event
async def on_ready():
    print(f'Logged in as {client.user.name} (ID:{client.user.id}) | Connected to {len(client.guilds)} servers')
    await client.change_presence(activity=discord.Game(name='GGtutorial'))
client.run("token_here")
```  
  As you can see its pretty strightforward, Bot starts, sets variable `client` with settings and then when it finishes connecting prints in console on what account it is logged in, what is is User ID, how many servers it is part. then it sets up Playing status to string `GGtutorial`

  Now, you need to upload your code to server, it is easy, most FTP software allows you to simply drag and drop files from host machine (your PC) to VPS. I recomend creating subfolder for your bot, such as in example `/home/tutorial/bots` since it allows to keep everything clean.

After uploading your code you need to start the bot, to do that use commmand `cd` in terminal, following my example folder it will be `cd /home/tutorial/bots`.  
    After that simply use  
`
python3 example.py
`  
  where `example.py` is your bot main file, after short while you will see output similar to this:   
  ![Sucessful boot of bot](https://cat.girlsare.life/6bRu1ZU.png "bot output")  
  which means that it sucessfully started and authenticated with discord.  
  
2. Disco  
   Starting bot in Disco is very similar, and using same code (just modified with Disco syntax and without token field) will yeld same result.  
   Command to start disco bot look like this:
   ```
   python3 -m disco.cli --token="MY_DISCORD_TOKEN" --run-bot --plugin simpleplugin
   ```  
   You might notice that it loads module, similar to when we used PIP to get libraries.  
     
     And thats it! your discord bot is up and running, however this is not an end yet, since if you close SSH it will end it process, we will learn how to prevent that after we discuss how to host JavaScript bot.  
       
       
## JavaScript

### Requirements  
 #### What will you need
   * Node.JS
   * Bot Library
    
 #### Instalation 
   * Node.js  
    in this example we will download node 12.x  
    Command used to do this is as follows:  
        ```
        curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
        sudo apt-get install nodejs
        ```  
This command downloads package from official distribution of Node.js for Debian based Operating systems (such as Ubuntu) afterward installs it, simple and fast, this is best way to install node.js
* Bot library  
 Once you install node you need to install library your bot uses, for Eris command looks like follows  
        `
        npm install --no-optional eris
        `
 For Discord.js Command looks like this:
       `
       npm install discordjs/discord.js
       `
 Both commands install liblaries **without** Voice Support.
 
 And that's it all baseline prerequisities! Let's head to starting it
 
 ### Starting Bot  
 So, we have all prerequisities for basic bot, lets start it! 
 It is suprisingly easy in both libraries, its literally same command.
 To start bots written in JavaScript you use command
 `
 node example.js
 `
 Where `example` is name of your bot file.
 
 
 ## How to keep bot "up" after ending SSH session  
 So, by now you have working bot, you happily close SSH client then notice with fear in eyes that it went down, you check logs, but there are no errors.
 
 Don't worry this is **normall**. In linux SSH when you run something its ran under SSH process, because of that when you close session it kills also all processes under it. I hear you asking "How to keep my bot up then?", well there are few ways, one of them is Screen
 
 ### Using Screen
 Screen is simplest solution, it requires no configuration and Ubuntu has it even preinstalled. It is also easiest to use. to start input `screen` into Terminal, you will see output like this:  
 ![Screen start]( https://cat.girlsare.life/5rau9hY.png "Screen")    
 It means that screen started sucessfully, now start your bot using instructions above. When you see that it sucessfully started press `ctrl+a` you now told screen to listen for keybind command, now if you press `ctrl+d` it will "deattach" screen into indepedent process, now you can safely end SSH session and bot will stay up.

If you will need to acess display with bot output just start SSH, input `screen` again and then input `screen -r` to view all screen, they will look like this `5051.pts-0.hostname_of_vps    (10/18/2019 02:08:53 AM)        (Detached)` notice 4 digits on begining of line, it is PID of that screen session, to access it use command `screen -r 5051` where 5051 is PID of screen you want to acess  
  
Screen has some shortcomings however: 
     - If you need to see bot output in console you need to always reattach screen
     - If Screen crash bot crash too
     - It does not provide autostart

### System.d Service
Another good solution is running your code as System Service using system.d, Setting that up is bit more complicated than using screen but it solves main shortcomings of Screen, Crashes and Autostart. Logs are saved and viewable in service information, Running code as service also makes it independent from any software apart from: Interpreter and Linux itself.

#### Creating service  
Create file using your favorite editor in path `/lib/systemd/system/exmple.service` content of file should look like this for JavaScript bot:
```
[Unit]
Description=Bot
After=multi-user.target
[Service]
WorkingDirectory=/home/username/botusername
User=username
ExecStart=/usr/bin/node /home/username/botusername/bot.js --no-prompt
Type=idle
Restart=always
RestartSec=15

[Install]
WantedBy=multi-user.target
```  
You need to replace username with your account (Linux, not discord) username (remember to do it both in `User` and `ExecStart`!) and in `ExecStart` replace `bot` in `bot.js` with name of your bot main file and edit description to whatever you want, its there just so you know what service is responsible for. Syntax is literally same on Python apart from ExecStart line which looks like this:

`ExecStart=/usr/bin/python3 /home/username/botusername/bot.py --no-prompt`  
  
same rules apply to editing lines as in JS version of Service file.

After you finish creating file, make sure everything is good then press `ctrl+x` then run command: `sudo systemctl daemon-reload` which will reload services to accept new one and `sudo systemctl enable example.service` (remember to replace example with your service file name) which will enable service, and done! service is ready to be used

#### Starting and Stopping service
So now that we have service ready we need to start it, it is easy, just use following command:  
`sudo systemctl start example.service`  
(as always, replace example with service file name) and bot should start, you can check if service is running properly using:  
`sudo systemctl status example.service`  
command to print information about it.

#### Usefull Commands to services
here is list of most usefull commands for services in one place

```
sudo systemctl stop example.service          #To stop running service 
sudo systemctl start example.service         #To start running service 
sudo systemctl restart example.service       #To restart running service 
```