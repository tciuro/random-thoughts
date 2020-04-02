# Hosting Vapor on Mac

This article outlines the steps needed to host a Vapor 3 application on Mac OS X.

## Assumptions

- Mac OS X Catalina (10.15)
- Vapor 3
- The domain's DNS has already been setup (e.g. `yourdomain.com`)
 
## Agenda

1. Install Xcode
1. Install Homebrew
1. Install Vapor 3
2. Install nginx
3. Configure nginx
4. Install Let's Encrypt certbot
5. Let's Encrypt: Choose how to set up automatic renewal
5. Set up automatic renewal via Launch Services
7. Set up automatic nginx relaunch
7. Set up automatic the Vapor app relaunch
6. SSL: a final inspection
6. Some parting thoughts...

### Install Xcode

Make sure you have the latest public version of Xcode installed (version 11.4 at the time of this wrinting). Head over to the App Store and download it. Once downloaded, open it and install the Internal Tools package if prompted by Xcode. Quit Xcode, open Terminal and type:

    $ sudo xcode-select -s /Applications/Xcode.app/Contents/Developer
    
This ensures that the Xcode toolchain will be used from now on.

### Install Homebrew

Install Homebrew on your Mac:

    $ /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"

### Install Vapor 3

    $ brew tap vapor/tap
    $ brew install vapor/tap/vapor
    
Verify that it's working:

    $ vapor --help
    
You should see a long list of available commands:

    Usage: vapor command

	Join our team chat if you have questions, need help,
	or want to contribute: http://vapor.team

	Commands:
		   new Creates a new Vapor application from a template.
			   Use --template=repo/template for github templates
			   Use --template=full-url-here.git for non github templates
			   Use --web to create a new web app
			   Use --auth to create a new authenticated API app
			   Use --api (default) to create a new API
		 build Compiles the application.
		   run Runs the compiled application.
		 fetch Fetches the application's dependencies.
		update Updates your dependencies.
		 clean Cleans temporary files--usually fixes
			   a plethora of bizarre build errors.
		  test Runs the application's tests.
		 xcode Generates an Xcode project for development.
			   Additionally links commonly used libraries.
	   version Displays Vapor CLI version
		 cloud Commands for interacting with Vapor Cloud.
		heroku Commands to help deploy to Heroku.
	  provider Commands to help manage providers.

	Use `vapor command --help` for more information on a command.
    
### Install nginx

We'll use Homebrew to install nginx. In Terminal, type:

    $ brew install nginx
    
Let's launch it:

    $ sudo nginx

Check that this link opens: http://localhost:8080

To stop nginx:

    $ sudo nginx -s stop

### Configure nginx

The nginx configuration file is located here: `/usr/local/etc/nginx/nginx.conf`

The default nginx config file has a bunch of commented out options. If you want to start small and grow from there as your needs require it, you may want to check [this template](https://github.com/tciuro/random-thoughts/blob/master/vapor/hosting%20on%20Mac/nginx.conf).

If you decide to go with the template, there are a few things to note:

- nginx listens on port 80
- you should replace `yourdomain.com` and `yourdomain_com` with your own domain.
- the template already contains the paths to the Let's Encrypt files. Make sure you replace `yourdomain.com` on each path with your own domain.
- the SSL cyphers are pretty decent.
- the location /api section redirects all requests with https://yourdomain.com/api* to your Vapor app, which typically runs on http://localhost:8080. 
- finally, the location / section are handled by nginx itself, serving requests/files from `/Users/you/www/yourdomain_com`. Change `you` and `yourdomain_com` according to your specifications/requirements/preference.

### Install Let's Encrypt certbot

First, we need to install Let's Encrypt:

    $ brew install letsencrypt
    
The following command will retrieve and install the certificates. Before you do, make make sure you enter the same domain you used to replace `yourdomain.com`. Let's Encrypt will verify that the domain points back to your machine, otherwise it'll error out. To install the certificates:

	$ sudo certbot --nginx
	
Once nginx has been installed and its config file set up, let's do some sanity check:

- make sure that ports 80 (HTTP) and 443 (HTTPS) are configured on your router so that these requests get routed to your machine runing the Vapor app.
- stop nginx, if it's running: `$ sudo nginx -s stop`
- run your Vapor app (from Xcode or `$ vapor run`)
- open a browser window and check that your app responds (e.g. http://localhost:8080/hello). If it does, awesome. Your Vapor app is ready to go.
- put some boilerplate html file in `/Users/you/www/yourdomain_com`.
- launch nginx: `$ sudo nginx`
- open a browser window and check that nginx serves the boilerplate file when hitting https://yourdomain.com.
- final check: verify that your Vapor app receives the request, only this time we'll issue the request which will go through nginx and then redirected to http://localhost:8080 (e.g. check that this works: https://yourdomain.com/hello).

### Let's Encrypt: Choose how to set up automatic renewal

The official instructions set up automatic renewal via `crontab`. There's also the option to use `Launch Services`, which I personally prefer. These are the options:

- `crontab` [the official certbot instructions](https://certbot.eff.org/lets-encrypt/osx-other.html). Make sure you select `nginx` from the `My HTTP website is running` popup.
- `Launch Services` (see section *Set up automatic renewal via Launch Services* below).

### Set up automatic renewal via Launch Services

Check the contents of the plist and adjust the calendar section to your liking. Next, copy the [the certbot auto renewal plist](https://github.com/tciuro/random-thoughts/blob/master/vapor/hosting%20on%20Mac/local.certbot.renew.plist) to `/Library/LaunchDaemons` and load it:

    $ sudo launchctl load -w /Library/LaunchDaemons/local.certbot.renew.plist

### Set up automatic nginx relaunch

Copy the [the nginx keepalive plist](https://github.com/tciuro/random-thoughts/blob/master/vapor/hosting%20on%20Mac/local.nginx.keepalive.plist) to `/Library/LaunchDaemons` and load it:

    $ sudo launchctl load -w /Library/LaunchDaemons/local.nginx.keepalive.plist
    
### Set up automatic the Vapor app relaunch

Copy the [the run Vapor app script](https://github.com/tciuro/random-thoughts/blob/master/vapor/hosting%20on%20Mac/launch_vapor.sh) to some location (e.g. `/Users/you/scripts/`).

Check the contents of [the Vapor app relaunch plist](https://github.com/tciuro/random-thoughts/blob/master/vapor/hosting%20on%20Mac/local.vapor.keepalive.plist) and set the script location chosen in the previous step. Next, copy the Vapor app relaunch plist to `/Library/LaunchDaemons` and load it:

    $ sudo launchctl load -w /Library/LaunchDaemons/local.vapor.keepalive.plist

### SSL: a final inspection

Reboot the Mac. If everything is OK, we should have nginx running. 

Now that everything is running, let's double check that the connection is in tip-top condition. Head over to [Qualys SSL Labs](https://www.ssllabs.com/ssltest/) and enter your domain (e.g. `yourdomain.com`) and wait for the report. Hopefully it'll come back with an excellent report card.

### Some parting thoughts...

So... how did it go? If you have feedback or just want to say hello, please let me know!
