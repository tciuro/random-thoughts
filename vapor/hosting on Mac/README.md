# Hosting Vapor on Mac

This article outlines the steps needed to host a Vapor 3 application on Mac OS X.

## Assumptions

- Mac OS X Catalina (10.15)
- Vapor 3
- Domain's DNS already setup
 
## Agenda

1. Install Xcode
1. Install Homebrew
1. Install Vapor 3
2. Install nginx
3. Configure nginx
4. Configure Let's Encrypt certbot
5. Install Launch Services plist

### Install Xcode

Make sure you have the latest public version of Xcode installed (version 11.4 at the time of this wrinting). Head over to the App Store and download it. Once downloaded, open it and install the Internal Tools package if prompted by Xcode. Quit Xcode, open Terminal and type:

    sudo xcode-select -s /Applications/Xcode.app/Contents/Developer
    
This assures that Xcode's toolchain will be used from now on.

### Install Homebrew

Just head to https://brew.sh and install it on the Mac.

### Install Vapor 3

    brew tap vapor/tap
    brew install vapor/tap/vapor
    
Verify that it's working:

    vapor --help
    
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

    brew install nginx
    
Let's launch it:

    sudo nginx

Check that this link opens: http://localhost:8080

### Configure nginx

The configuration file is located here: /usr/local/etc/nginx/nginx.conf

I use TextEdit to edit these type of files, but you can use your favorite text editor, of course. In Terminal do:

    /usr/local/etc/nginx/nginx.conf


