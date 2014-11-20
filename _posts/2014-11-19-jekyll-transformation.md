---
layout: post
title:  "Jekyll Conversion"
date:   2014-09-16 16:02:00
categories: 
---

# The Jekyll Conversion

Today I converted my blog from WordPress to Jekyll. Here we are in action.

Although it wasn't quite as easy as it sounds here, It was successful and I'm happy with the results so far.

## Why Change?

## How I did It

Prepare Ruby and Rubygems

  apt-get install ruby
  update_rubygems
  apt-get install nodejs
 
Install Jekyll
 
  gem install jekyll

Create the site files

  cd /var/www/html
  jekyll new blog
  
Bring up the site for testing on the local network:

	cd blog
	sudo jekyll serve -H 172.16.20.7 -P 8080
	
	Configuration file: none
                Source: /var/www/html/blog
           Destination: /var/www/html/blog/_site
          Generating...
                        done.
	 Auto-regeneration: enabled for '/var/www/html/blog'
	Configuration file: none
        Server address: http://172.16.20.7:8080/
      Server running... press ctrl-c to stop.

Enable a new Apache config for Jekyll, turn off WordPress and kick Apache:

	vim /etc/apache2/sites-available/jekyll.conf
	a2ensite jekyll.conf
	a2dissite wordpress.conf
	service apache2 reload

Setup the Deployment Workflow:

* create deployer user
* setup ssh key
* Add git repo
* Add git post-receive hook
* Create skeleton Jekyll directory on my laptop
* Add it to git

