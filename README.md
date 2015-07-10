# LAMP-CookBooks
Chef cookbook to configure LAMP stack with WordPress installed
For this project, we shall use a list of pre-built cookbooks that the chef community has created. Librarian-chef can be used to install these cookbooks and their dependencies for us automatically. This is by far the easiest way to maintain cookbooks.
cd  ~/Projects
mkdir chef-demo
cd chef-demo
knife solo init .

To configure
1.	Apache
2.	MySQL
3.	PHP

librarian-chef init
vi cheffile
add the following  below the line:  site ‘http://community.opscode.com/api/v1’

cookbook ‘apache2’
cookbook ‘mysql’
cookbook ‘php’
	
Now run 
librarian_chef install
and, it would go and grab your cookbooks and dependencies for you.

On the Node

Change into your nodes directory and create a new node file.
cd  nodes
touch {hostname}.json
Now open the file in your text editor and add the following:
{
  "run_list": [
    "recipe[apache2]",
    "recipe[mysql]",
    "recipe[php]"

cd ~/Projects/chef-demo
knife solo bootstrap root@hostname

You can add an optional 3rd parameter here which is the path to your nodefile if you didn't name it in the recommended format.
You should soon see knife logging into the server, then downloading, installing, and running Chef.
 knife solo bootstrap
This will login to your server, download and install chef, copy across your cookbooks, and then run chef. This is the composite of the two commands below.
1.	knife solo cook
This will login to your server, copy across your cookbooks, and then run chef
1.	knife solo prepare
This will login to your server, then download and install chef.
Once Chef has finished its run open up a browser and visit your IP Address or Hostname for this Droplet and you should see something like this:
 
That means that Chef has worked. Normally you would expect to see something like "It works!" here, but the apache2 Chef recipe doesn't install the "Default Site" that apache normally comes with.
Now we can login to the server to do this because any manual changes you make to the server will be overwritten every time you run Chef. Remember: We want to end up with a set of recipes we can run over and over again to get a server to the exact same specification each time.

Even though we have a working apache server, but we don't have any sites loading. Therefore, we need to enable the default site in apache, and we do that by editing our "node attributes" for apache.
If you open cookbooks/apache2/attributes/default.rb you will see a whole bunch of attributes for different platforms. The one we're interested in is default_site_enabled under centos, which is currently set to false.
Do not edit this here as node attributes can be over-ridden on a node-by-node basis in our node file. Basically, any key in our node file's json which is not run_list is a node attribute. The attributes file for each recipe (which matches the recipe name in the attributes directory in the cookbook)  shows what attributes are available for us to over-ride. We're only interested in this one today, so update your node file like so:
{
  "apache": {
    "default_site_enabled": true
  },
  "run_list": [
    "recipe[apache2]",
    "recipe[mysql]",
    "recipe[php]"
  ]
}
You can see that the key names and hierarchy matches that of the attribute file we looked at above. Now we re-run Chef using knife, and this time we use the cook command instead of bootstrapas we don't need to install Chef on the server again.
Just a quick note: Chef is smart enough to know when it's already been run, so it's fairly safe to run it multiple times and it will only modify things that have changed in your local cookbooks.
knife solo cook root@82.196.8.99
Now reload your browser and you should see the familiar "It works!" page:
 
Test PHP And MySQL
SSH to your server run 
php -v.	

Success! Now you're probably thinking you'd like to test MySQL. But there's a problem. If you run mysql without any parameters on your new server you cannot connect. That's because the default recipe for the  mysql cookbook only installs the MySQL client, not the MySQL server.
If you take a look in ./cookbooks/mysql/recipes you should see a recipe called server.rb. It's a good bet that that's what installs We need to add MySQL Server to our run_list after recipe[mysql]. We'll leave the default MySQL recipe there as we are going to need the client too. Our node file should now look like this:
{
  "apache": {
    "default_site_enabled": true
  },
  "run_list": [
    "recipe[apache2]",
    "recipe[mysql]",
    "recipe[mysql::server]",
    "recipe[php]"
  ]
}
Exit out of your SSH session on your server and run 
knife solo cook root@your.ip.or.hostname
as root to re-run Chef with our new settings. We need to set a MySQL root password for this node! Update the node file to look like the following.
{
  "apache": {
    "default_site_enabled": true
  },
  "mysql": {
    "server_root_password": "yoursecretsecurepassword",
    "server_repl_password": "yoursecretsecurepassword",
    "server_centos_password": "yoursecretsecurepassword"
  },
  "run_list": [
    "recipe[apache2]",
    "recipe[mysql]",
    "recipe[mysql::server]",
    "recipe[php]"
  ]
}
Re-run the knife cook command and once it's complete ssh back into your server. 
Run 
mysql -u root -yoursecretsecurepassword 
and you should be seeing a nice mysql>prompt, showing  that MySQL Server is now installed and working!

Finally, we have one last thing to do, test PHP on apache.
 Still in your ssh session, type
 exit to quit MySQL, 
then add an info.php
 to your default site:
mysql> exit
Bye
cd /var/www
touch info.php
nano info.php
Now enter:
<?php phpinfo();
and save and exit
Open http://yourserver/info.php
 in your browser and it should ask you to download the file.
Add a modphp5 recipe to the `runlist`.


{
  "apache": {
    "default_site_enabled": true
  },
  "mysql": {
    "server_root_password": "yoursecretsecurepassword",
    "server_repl_password": "yoursecretsecurepassword",
    "server_centos_password": "yoursecretsecurepassword"
  },
  "run_list": [
    "recipe[apache2]",
    "recipe[apache2::mod_php5]",
    "recipe[mysql]",
    "recipe[mysql::server]",
    "recipe[php]"
  ]
}

Notice how we put mod_php5 before php in our run_list. Normally Chef will run things in order, but the mod_php5 recipe requires the php recipe, so Chef is smart enough to know to run that first.
Now we can exit our SSH session and re-run knife cook and refresh our info.php in the browser.



