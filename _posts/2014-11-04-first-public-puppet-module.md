---
layout: post
title:  "My First Public Puppet Module, cornellio-riak"
date:   2014-11-04 11:02:00
categories: puppet riak
---

Today I published my first Puppet module to the public.

* [cornellio-riak on Github](https://github.com/Cornellio/puppet-riak)
* [cornellio-riak on the Puppet Forge](https://forge.puppetlabs.com/cornellio/riak)

It provisions a Riak cluster. Riak is a distributed, key-value datastore released by Basho. The module performs the install from your existing yum repos and handles the cluster join, plan and commit operations automatically. I designed it for production systems in mind, so I coded the module to tune Linux for good performance as recommended by Basho for high speed data access.

I think it’s nice start on my first public module, although I need to make it more generic, allow further customization through class parameters and support more operating systems.

I’ve been using it to manage a Riak cluster that feeds data to a production, public facing web site for a few months now so it seems pretty solid.