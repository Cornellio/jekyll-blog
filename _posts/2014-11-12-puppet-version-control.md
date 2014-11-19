---
layout: post
title:  "Doctor Jekyll has arrived"
date:   2014-06-12 11:02:00
categories: puppet
---

If you use Puppet for configuration management, placing your code under version control is a good idea. In fact this is step 0. Just do it. You can use Git or Subversion for this purpose. In this post I explain how to do it using Subversion since that's what we have at work.

This example assumes the use of Puppet Enterprise and that you already have a Subversion system setup. If you’re using Puppet open source, simply adjust the Puppet paths and daemon names accordingly.

In this workflow, the Subversion server setup as follows:
<ul>
  <li>There is an svn repository called <code>systems</code></li>
  <li>Puppet code will be stored in this repository under <code>configs/puppet</code></li>
</ul>
<strong>Place Puppet into version control as follows</strong>
<ul>
  <li>Log into your Subversion server as root</li>
  <li>Make a directory in your repository to store your code insvn mkdir -m “New dir for puppet” https://svn.pretendco.com/repos/systems/configs/puppet/</li>
</ul>
<strong>Enable Auto Props</strong>

Enabling Subversion auto props allows you to add ID tag to your files so you can track author, revision #, etc.

For each person who will be contributing Puppet code, enable auto props for puppet files by editing their <code>~/.subversion/config</code> as follows:
<pre><code>[miscellany]
enable-auto-props = yes

[auto-props]
*.pp = svn:keywords=Id
*.erb = svn:keywords=Id
*.rb = svn:keywords=Id
*.cnf = svn:keywords=Id
*.cfg = svn:keywords=Id
*.conf = svn:keywords=Id
*.sh = svn:keywords=Id
</code></pre>
<strong>Stop Puppet Services</strong>

The next step is to convert Puppet directories where your manifests are stored into svn working copies. Before doing so, stop all Puppet daemons on the master:
<pre><code>for i in pe-activemq pe-httpd pe-mcollective pe-memcached pe-puppet pe-puppet-dashboard-workers ; do service $i stop ; done 
</code></pre>
<strong>Convert Puppet Directory into a Subversion Working Copy</strong>

Move aside the Puppet config directory <code>/etc/puppetlabs/puppet</code> and check out an empty repository in it’s place:
<pre><code>mv /etc/puppetlabs/puppet /etc/puppetlabs/puppet.original
mkdir /etc/puppetlabs/puppet
cd /etc/puppetlabs
svn checkout https://svn.pretendco.com/repos/systems/configs/puppet/ .
</code></pre>
Copy puppet files into the working copy with permissions and ownership intact
<pre><code>rsync -a /etc/puppetlabs/puppet.original/* /etc/puppetlabs/puppet/
</code></pre>
Insert the svn property ID tag into Puppet files by placing <code># @version $Id: $</code> at the top of each file:
<pre><code>find /etc/puppetlabs/puppet -regextype posix-egrep -regex ".*\.(pp|conf|erb|rb)$" -exec sed -i '1s/^/\# @version $Id: Exp $\n/' {} \;
</code></pre>
<strong>Add Files to Subversion</strong>
<pre><code>cd /etc/puppetlabs/puppet
svn add *
</code></pre>
<strong>Commit to the Repository</strong>
<pre><code>svn ci -m "Initial commit of Puppet configs." 
</code></pre>
<strong>Start Puppet Services</strong>
<pre><code>for i in pe-activemq   pe-httpd   pe-mcollective   pe-memcached   pe-puppet   pe-puppet-dashboard-workers; do service $i start ; done
</code></pre>
Contratulations! Now you have proper version control for your Puppet code. To take it to the next level, consider using Puppet <em>environments</em> to separate your development code from production. In <a href="http://epiphia.com/2013/10/puppet-environs/">my other post</a> I explain how to use Puppet <em>environments</em>.
