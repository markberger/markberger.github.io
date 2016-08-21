---
layout: post
title: An Introduction to Chef Solo
date: 2014-08-17 19:11:29.000000000 -07:00
---
This past weekend I was able to automatically provision my personal server using [Chef](http://www.getchef.com/chef/). Chef is a powerful tool for configuring servers, but most of the guides I found were overly complex or aimed at groups with large computer clusters. Therefore this is my attempt at writing a stupid-simple guide to Chef-solo for a small number of personal machines.

### Why should I use Chef?

I don't know any reasons for using Chef specifically, but using a configuration management tool can save you a lot of headaches. If you every break your virtual private server, you can always create a new one and redeploy your software in minutes without going through all of the motions by hand. While it might seem trivial now, you'll be happy you invested the time six months later when you've completely forgotten what you did to get nginx working just right.


### Assumptions

I'm assuming you have the following:

* A recent version of Ruby
* A recent version of RubyGems
* A linux server

If you are looking for a hosting service, I personally recommend [Digital Ocean](https://www.digitalocean.com/?refcode=58be52a52767) (referral link).

### Setup
To start, install [knife-solo](http://matschaffer.github.io/knife-solo/) and [berkshelf](http://berkshelf.com/).
```
gem install knife-solo
gem install berkshelf
```

Knife-solo is a tool which adds a few extensions to Chef's command line tool, knife, to make life easier. Berkshelf is a dependency management tool for Chef cookbooks.

### What's a cookbook?

A cookbook is a collection of scripts which focus on configuring a certain aspect of the machine or installing a certain application. So, for example, there are nginx cookbooks which focus on installing and configuring nginx. Each one of the scripts inside a given cookbook is referred to as a recipe in order to further the kitchen analogy.

### Let's get cooking

Now that we have knife-solo, create a new directory and run the following command:

```
knife solo init chef_dir/
```

This will give you the following:

```
chef_dir/
	Berksfile
	cookbooks/
	data_bags/
	environments/
	nodes/
	roles/
	site-cookbooks/
```

You'll notice that knife-solo has created two cookbook directories: `cookbooks` and `site-cookbooks`. Site cookbooks are meant for specific server configurations, like our virtual private server, while cookbooks are meant for general use cases, such as installing nginx.

Let's create a new cookbook for our server.

```
knife cookbook create -o site-cookbooks/ vps
```

Now you'll have this:

```
vps/
	attributes/
    definitions/
    files/
    	default/
    libraries/
    providers/
    recipies/
    	default.rb
    resources/
    templates/
    	default/
    metadata.rb
    README.md
    CHANGELOG.md
```

In this tutorial we are just going to install some packages, configure ssh, and create a new user, so you can ignore most of these directories.

We're going to be using the `user` cookbook and the `openssh` cookbook, so add the following lines to your `metadata.rb` file.

```
depends "user"
depends "openssh"
```

This is Chef's way of knowing which cookbooks to have before attempting to run any of the recipes inside our new cookbook.

Now let's open up `recipies/default.rb` and write our first recipe.

```ruby
node.packages.each do |pkg|
    package pkg
end

# provision user account
include_recipe 'user::data_bag'

# provision ssh
include_recipe 'openssh'
```

First, we iterate over a list of packages and install each one. Then we execute the `data_bag` recipe found in the `user` cookbook followed by the default recipe for `openssh`.

Right now we are using the default settings found in each individual cookbook, and Chef has no idea what packages we want to install. To resolve our unknown variables and change the default settings we need to use the `attributes` folder.

Inside `site-cookbooks/vps` create `attributes/default.rb` and add the following:

```ruby
default.packages = %w(vim git)

default.users = ['ourUser']
# Don't create an ssh key for us
default.user.ssh_keygen = false

default.openssh.server.permit_root_login = 'no'
default.openssh.server.password_authentication = 'no'
default.openssh.server.allow_groups = 'sudo'
default.openssh.server.port = '27984'
default.openssh.server.login_grace_time = '30'
default.openssh.server.use_p_a_m = 'no'
default.openssh.server.print_motd = 'no'
```

Great! Now Chef will install vim and git, create the ourUser account, and lock down ssh for us.

Now we need to tell `user` what we want the `ourUser` account to look like.

Go to `chef_dir/data_bags/` and create a `users` directory. Create `ourUser.json` inside the `users` directory and add the following:

```
{
	"id": "ourUser",
	"password": "your_password",
	"groups": [
		"sudo"
	],
	"ssh_keys": [
		"your_public_ssh_key"
    ]
}
```

Note that you need to generate the `"your_password"` string using `mkpasswd -m sha-512`.

Now we need to create a configuration file for Chef to use. Create a `vps.json` inside the `nodes` directory and place the following:

```
{
	"run_list": [
		"recipe[vps]"
	]
}
```

Finally, we need to add our dependencies to the `Berksfile` so that Chef will install them on our linux box before executing our recipe. Your `Berksfile` should look like this:

```
source "https://supermarket.getchef.com"

cookbook 'user'
cookbook 'openssh'
```

That's it! Now we can finally run Chef on our server.

```
knife solo bootstrap root@our_server nodes/vps.json
```

This will install Chef on the server, download the cookbooks in the `Berksfile`, and execute our default recipe in the `vps` cookbook.

Finally, we need to clean up the machine. However, we must login as `ourUser` since we've prevented root login in our Chef recipe.

```
knife solo clean ourUser@our_server
```

Hopefully this helped you set up Chef with your linux machine. Chef seems to be easy enough to use from this point once you're able to somewhat see how all the pieces fit together. This is my first time using Chef so if I've made an error or committed some bad practices, please let me know in the comments or feel free to email me.

Also if you're looking for a Chef cookbook to install [Ghost](https://github.com/tryghost/Ghost), checkout [the one I made](https://github.com/markberger/ghost-blog) to deploy this blog using sqlite3.

Cheers.
