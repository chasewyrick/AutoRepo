# AutoRepo
Cydia repositories - simpler than ever

# What is this?
AutoRepo is a PHP script that automatically generates an up-to-date Packages.bz2-file whenever a new debian package is uploaded to a given directory. 
That means, whoever hosts the repository does not have to do anything anymore, except for placing their .deb file on their webspace. 

# Motivation
Earlier this day, I read this on Reddit
https://www.reddit.com/r/jailbreakdevelopers/comments/3ecucb/discussion_cydia_repo_creator_beta/
and just like /u/superchilpil I thought, nobody needs an application to create a repository if they are able to create something worth being distributed. However, the OP has a point there, saying it is a nifty thing to just have to drag and drop. 
Considering he revealed to us that it took him 100 hours of work to create his tool and that he wanted to make his tool paid, I felt the urge to compete with him and develop my own concept for an easy "drag-and-drop repository" which I am currently implementing. 

# State
Up to now, I quickly implemented a little function to get the contents of the "control"-file of debian packages, 
I didn't really care about good error handling yet. 

**Update 25.07.2015, 21:40 (9:40 pm), Central European Time**
This is basically finished, there is still a bit of polishing to be done and a .htaccess file for rewriting requests to /Packages.bz2 to /handler.php to provide. 

**Update**
Completed. Check further below for more information

# How does it work?
The PHP script is executed every time someone tries to access /Packages.bz2 (which is accessed by Cydia to know what Packages are available on your repository). 
The script first checks whether the provided Packages.bz2 is up to date, for that it compares the "last modified"-timestamps of the file Packages.bz2 (the real one, not the script itself!) and of the directory "debs". 
If  Packages.bz2 is older than the latest file in the "debs" directory, the script will return the old Packages.bz2 (or a fake, empty one if there is no Packages.bz2, yet) to the user and then generate a new Packages.bz2 in the background after locking write access to that file for other instances of handler.php. 
The idea behind that is that users should not have to wait while the new Packages.bz2 is generated, for the users it means: whenever a new package is updated, the first one who refreshes his sources will not receive the latest version of Packages.bz2 (the same thing applies for those who refresh their sources while Packages.bz2 is locked). 
However, considering the first one to refresh his sources is usually the repository maintainer, no problems should occur in a real life scenario. 
To generate the Packages.bz2, the script looks for all debian packages (.deb) in the "debs" directory, using my PHP implementation of an ar-unarchiver the control.tar(.gz) is extracted, which in turn is decompressed if needed. Afterwards my PHP implementation of a tar-unarchiver (only supports the old tar format, no need for ustar here) extracts the "control" file. 
This file holds basic information about the given package and is what makes up most of the Packages(.bz2) file. 
From that information, the script determines whether there are duplicate packages and if so, which one of them to use. 
Obviously, the package with the higher version number is always preferred - that also means that you never have to delete old packages again, as long as you have enough space on your hard disk; the script will automatically check which one is the newest. 
If you are interested in how debian package versioning works, check https://www.debian.org/doc/debian-policy/ch-controlfields.html#s-f-Standards-Version
No, really, read that! 
When the script has collected all required information on all packages, the information is bzip2 encoded and stored as Packages.bz2 for further use. 
Simple, is it not? 

# Features
I am aware of the fact that not everybody cares to read the explanation on how it works, therefore here is a short list of the basic features: 
* Drag-and-drop repository
* Easy to use, easy to maintain, basically nothing to do
* Script automatically determines which package is the latest by comparing version numbers of all files
* No need for special PHP modules

# Requirements
AutoRepo requires basically nothing to be setup
* Webspace with PHP >= 5.4.0 and mod_rewrite (most webspaces support/include these, just grab any free webspace and let's go)
* The files from this repository
* Packages to host on your own Cydia repository
* A minute or two to give me feedback

# How to set this up?
1. Get a (free) webspace which has PHP >= 5.4.0 and mod_rewrite
2. Clone this repository or just manually download handler.php and .htaccess
3. Create a directory called "debs" on your webspace
4. Copy over handler.php and .htaccess to your webspace
5. Place your debian packages (.deb files) in the "debs" directory
6. Navigate to <yourdomain>/handler.php to manually trigger the first Packages.bz2 generation (see How does it work? for more information)
7. Let me know how it works for you

# License
Check the LICENSE-File. 
