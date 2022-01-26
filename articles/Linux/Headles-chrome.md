 
How to run headless Chrome with extensions in Ubuntu 16.04 using xvfb in Nodejs
Robin Ding
Robin Ding

Jan 16, 2019·2 min read




There are many different ways to run chrome in headless mode within Nodejs on a Ubuntu server. (No Ubuntu GUI installed)
a. Use puppeteer (https://github.com/GoogleChrome/puppeteer), this doesn’t suport chrome extensions.
b. Use xvfb (X virtual framebuffer), this support chrome extensions.
In this post, let’s take a look the 2nd option, using xvfb.
XVFB
From wikipedia xvfb
Xvfb or X virtual framebuffer is a display server implementing the X11 display server protocol. In contrast to other display servers, Xvfb performs all graphical operations in virtual memory without showing any screen output. From the point of view of the client, it acts exactly like any other X display server, serving requests and sending events and errors as appropriate. However, no output is shown.
Install Chrome
Add Key
wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | sudo apt-key add
Set Repository
echo 'deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main' | sudo tee /etc/apt/sources.list.d/google-chrome.list
Install Package
sudo apt-get update 
sudo apt-get install google-chrome-stable
Install XVFB
Installing xvfb is pretty easy:
sudo apt-get install xvfb
If you want to run chrome with extensions, you can run
xvfb-run -a --server-args="-screen 0 1280x800x24 -ac -nolisten tcp -dpi 96 +extension RANDR" command-that-runs-chrome.
Run Chrome with XVFB
In order to run chrome successful with xvfb in headless mode, we need to
Add xvfb-run in front of any command which we want to run with chrome.
Start Chrome with the --disable-gpu, --no-sandbox, and --disable-setuid-sandbox flags
xvfb-run google-chrome --disable-gpu --no-sandbox --disable-setuid-sandbox
Done !!!
