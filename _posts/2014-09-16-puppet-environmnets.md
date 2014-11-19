---
layout: post
title:  "Setup Puppet Environments"
date:   2014-09-16 16:02:00
categories: puppet
---

<p>If you use Puppet to manage Linux hosts it’s a good idea to use <em>environments</em> to separate development code from production, thereby mitigating risky changes. In this article I will illustrate how I use Puppet environments and how I distribute changes in each environment using Subversion. </p>

<p>I will describe how to setup these 3 Puppet environments:</p>

<ul>
<li>Development</li>
<li>Testing</li>
<li>Production</li>
</ul>

<p>Using a revision control system such as Git or Subversion is an excellent way to move changes between your environments. I use Subversion for this purpose. This provides version control and a way of syncing files between each environment. In effect, each Subversion working copy is a replica of the the Puppet code stored in a central repository.</p>

<p>A sample workflow of how I propagate code through the environments using Subversion is described in the next section.</p>

<h2>Configure Puppet to use Environments</h2>

<p>Create 3 new directories for each environment:</p>

<pre><code>mkdir -p /etc/puppetlabs/environments/{development,testing,production}
</code></pre>

<p>The preexisting Puppet system is the production environment. The keep things organized I like to link the content there to the new &#8216;environments&#8217; directory creatd above:</p>

<pre><code>ln -s /etc/puppetlabs/puppet/files /etc/puppetlabs/environments/production/files
ln -s /etc/puppetlabs/puppet/manifests /etc/puppetlabs/environments/production/manifests    
ln -s /etc/puppetlabs/puppet/modules /etc/puppetlabs/environments/production/modules
</code></pre>

<p>To make Puppet aware of these new locations, add new sections to your puppet.conf file like this:</p>

<pre><code>[development]
modulepath = /etc/puppetlabs/environments/development/modules
manifest = /etc/puppetlabs/environments/development/manifests/site.pp

[testing]
modulepath = /etc/puppetlabs/environments/testing/modules
manifest = /etc/puppetlabs/environments/testing/manifests/site.pp
</code></pre>

<p>Note: The location for the production environment is already set, since it&#8217;s the existing default setup as defined in the [main] section:</p>

<pre><code>[main]
modulepath = /etc/puppetlabs/puppet/modules:/opt/puppet/share/puppet/modules
manifest = /etc/puppetlabs/puppet/manifests/site.pp
</code></pre>

<h2>Theory of Operation</h2>

<p>The development environment is intended to be used for unrestircted development of new manifests. Once the code is stabilized it will be promoted to the testing environment where more thorogh validating can take place. Minimal changes to the code should take place here. When the code has been fully tested it will be promoted to the production environment.</p>

<p>Each discrete environment contains a Subversion working copy that&#8217;s linked to the central repo.</p>

<h3>Development Workflow</h3>

<ul>
<li>Create manifests in the <strong>Development</strong> environment and test agents it</li>
<li>Validate manifests in the <strong>Testing</strong> environment on agents with a similar configuration as production</li>
<li>Promote manifests to the <strong>Production</strong> environment</li>
</ul>

<p>During testing, an agent can be pointed at a particular environment by running <code>puppet agent -t --environment development</code>. (Scheduled Puppet runs will still run against production unless you change the agent&#8217;s config file.)</p>

<p>Code is moved through each stage of the development process in a linear fashion. My workflow for creating a new Puppet module goes like this:</p>

<h2>Create Module in Development Environment</h2>

<ol>
<li><p>Log in to the puppet master and change to the <code>environments</code> directory</p>

<pre><code>cd  /etc/puppetlabs/environments
</code></pre></li>
<li><p>Update the subversion working copy to pull down the latest production code from the repository.</p>

<pre><code>svn up development/*

Authentication realm: &lt;https://svn.virginam.local:443&gt; VX Subversion server
Password for 'pete.cornell':
At revision 190.
At revision 190.
At revision 190.
</code></pre></li>
</ol>

<p>All 3 environments are at the same revision.</p>

<ol>
<li><p>Create the directory structure for a new module:</p>

<pre><code>cd development/modules/
mkdir -p mysql/{manifests,templates,files}
touch mysql/manifests/init.pp
</code></pre></li>
<li><p>Create the module as needed.</p></li>
<li><p>Validate the manifest for correct syntax.</p>

<pre><code>puppet parser validate mysql/manifests/init.pp
</code></pre></li>
<li><p>Assign the mysql module to a test system by including it in <code>nodes.pp</code>:</p>

<pre><code>node 'lab-sqldev1b.virginam.local' inherits basenode-rhel5-centos5 {
    include mysql
}
</code></pre></li>
<li><p>Add the new module to svn and commit the changes.
 svn add mysql/</p>

<pre><code>A         modules/mysql
A         modules/mysql/templates
A         modules/mysql/manifests
A         modules/mysql/manifests/init.pp
A         modules/mysql/files
A         modules/mysql/files/my.cnf

svn ci -m &quot;Adding module to manage mysql server&quot; --username pete.cornell

Adding         mysql
Adding         mysql/files
Adding         mysql/files/my.cnf
Adding         mysql/manifests
Adding         mysql/manifests/init.pp
Adding         mysql/templates
Transmitting file data ..
Committed revision 191.
</code></pre></li>
<li><p>Commit your changes to <code>nodes.pp</code>
 cd ~/environments</p>

<pre><code>svn ci -m &quot;Updating nodes.pp with mysql module.&quot; development/manifests/nodes.pp --username pete.cornell

Transmitting file data ..
Committed revision 191.
</code></pre></li>
</ol>

<h2>Run Agent in the Development Environment</h2>

<p>Do a puppet run on a test system to validate the module. We can run the puppet agent against the development environment using the <code>--environment</code> option, and you can run puppet in non-operation mode using <code>--noop</code> to see what changes would be applied without actually making changes.</p>

<pre><code>puppet agent -t --environment development

info: Retrieving plugin
.
.
info: /Stage[main]/Mysql::Config/File[/etc/my.cnf]: Filebucketed /etc/my.cnf to main with sum   740258c758e3ef283d2fd5f862bc9e4c
notice: /Stage[main]/Mysql::Config/File[/etc/my.cnf]/content: content changed '{md5}740258c758e3ef283d2fd5f862bc9e4c' to '{md5}d41d8cd98f00b204e9800998ecf8427e'
notice: /Stage[main]/Mysql::Config/File[/etc/my.cnf]/owner: owner changed 'root' to 'mysql'
notice: /Stage[main]/Mysql::Config/File[/etc/my.cnf]/group: group changed 'root' to 'mysql'
notice: /Stage[main]/Mysql::Config/File[/etc/my.cnf]/mode: mode changed '0644' to '0664'
info: /Stage[main]/Mysql::Config/File[/etc/my.cnf]: Scheduling refresh of Class[Mysql::Service]
info: Class[Mysql::Service]: Scheduling refresh of Service[mysqld]
notice: /Stage[main]/Mysql::Service/Service[mysqld]/ensure: ensure changed 'stopped' to 'running'
notice: /Stage[main]/Mysql::Service/Service[mysqld]: Triggered 'refresh' from 1 events
notice: Finished catalog run in 42.83 seconds
</code></pre>

<h3>Validate Manifest in the Testing Environment</h3>

<ol>
<li><p>Promote the code to the testing environment</p>

<p>cd ~/envrionemnts/testing
svn up mysql</p>

<p>Authentication realm: <a href="https://svn.virginam.local:443">https://svn.virginam.local:443</a> VX Subversion server
Password for &#8216;pete.cornell&#8217;:
At revision 191.
At revision 191.
A modules/mysql
A modules/mysql/files
A modules/mysql/files/my.cnf
A modules/mysql/manifests
A modules/mysql/manifests/init.pp
A modules/mysql/templates
Updated to revision 191. </p></li>
<li><p>Login to a test system and run the puppet agent</p>

<p>puppet agent -t &#8211;environment testing</p></li>
<li><p>Validate the manifest in the testing environment using a host that mirrors it&#8217;s production counterpart as much as possible. Manifests that make significant changes should be applied and left in place for a period of several days and all other subsystems that are managed by Puppet should be verified as well.</p></li>
</ol>

<h3>Promote Manifest to Production Environment</h3>

<p>After passing the testing phase above, promote the code to the production environment by commiting the changes and doing a checkout in production.</p>

<pre><code>svn ci -m &quot;Promote mysql manifest to production.&quot; --username pete.cornell

cd ~/environments/production
svn up mysql

Authentication realm: &lt;https://svn.virginam.local:443&gt; VX Subversion server
Password for 'pete.cornell':
At revision 191.
At revision 191.
A    modules/mysql
A    modules/mysql/files
A    modules/mysql/files/my.cnf
A    modules/mysql/manifests
A    modules/mysql/manifests/init.pp
A    modules/mysql/templates
Updated to revision 191.   
</code></pre>

<p>Log in to a production system and run the puppet agent, or simply wait for the puppet run to occur automatically.</p>

<pre><code>puppet agent -t
</code></pre>

<h2>Detailed Svn Workflow</h2>

<p>Below is a more detailed explanation of using svn to promote code through the 3 Puppet environments. This section focuses entirely on the specific svn commands needed. Hopefully svn will stay out of the way and let us focus on more interesting things like writing Puppet manifests.</p>

<p>Assume you have made a change to the <code>nodes.pp</code> file and are ready to promote it to production. The entire workflow would be as follows:</p>

<ul>
<li>Edit <code>nodes.pp</code> in the <strong>dev</strong> environment</li>
<li>Commit the change to the repo</li>
<li>Pull the change into the <strong>testing</strong> environment</li>
<li>Pull the change into the <strong>prod</strong> environment</li>
</ul>

<p>Here is a full example of the workflow. </p>

<p>Access the Puppet master using your own account, not root or pe-puppet:</p>

<pre><code>Holocron:~ petec$ ssh admin03-sc9
Last login: Wed Jun 19 10:58:29 2013 from 172.18.100.180
[petec@admin03-sc9 ~]$
[petec@admin03-sc9 ~]$ cd environments/
</code></pre>

<h4>Edit nodes.pp and Save Changes</h4>

<pre><code>petec@admin03-sc9 environments]$ vim development/manifests/nodes.pp
</code></pre>

<h4>Check Status</h4>

<p>[petec@admin03-sc9 environments]$ svn st development/manifests/nodes.pp
M development/manifests/nodes.pp</p>

<h4>Commit Changes to the Repo</h4>

<pre><code> svn ci -m &quot;Removed reference to old server web09&quot; development/manifests/nodes.pp --username pete.cornell
Transmitting file data .
Committed revision 264.
</code></pre>

<h4>Pull Changes from the Repo into Testing</h4>

<pre><code>svn up testing/manifests/nodes.pp
Authentication realm: &lt;https://svn.virginam.local:443&gt; VX Subversion server
U    testing/manifests/nodes.pp
Updated to revision 264.
</code></pre>

<h4>Pull Changes from the Repo into Production</h4>

<pre><code>svn up production/manifests/nodes.pp
Authentication realm: &lt;https://svn.virginam.local:443&gt; VX Subversion server
U    production/manifests/nodes.pp
Updated to revision 264.
</code></pre>

<h4>Merging Changes</h4>

<p>If separate changes were made to the same file(s) in multiple environments without following the linear progression above, you may need to merge changes, commit the merged data back to the repo, then pull down the merged file from the repo into your other working copies.</p>

<p>To work through an example of this process, assume that <code>nodes.pp</code> was changed in <strong>dev</strong> and committed to the repo. Meanwhile a separate change was made to the copy of <code>nodes.pp</code> in the <strong>testing</strong> environment but has not yet been committed to the repo. The changes can be merged as follows:</p>

<p>Merge the working copy in <strong>testing</strong> with the copy in the repo.</p>

<pre><code>svn merge https://svn.virginam.local/repos/systems/configs/puppet/puppetlabs/puppet/manifests/nodes.pp testing/manifests/nodes.pp --username pete.cornell
--- Merging r263 through r264 into 'testing/manifests/nodes.pp':
U    testing/manifests/nodes.pp
</code></pre>

<p>After merging you need to commit a copy of that merged file back into the repo:</p>

<pre><code>svn ci -m &quot;Merging r263 through r264&quot; testing/manifests/nodes.pp --username pete.cornell
Transmitting file data .
Committed revision 264.
</code></pre>

<p>At this point the nodes.pp should be in sync between <strong>testing</strong> and the copy in the repo. </p>

<p>Next, propagate the change to <strong>dev</strong> and <strong>prod.</strong></p>

<pre><code>cd production/manifests/

svn up .
Authentication realm: &lt;https://svn.virginam.local:443&gt; VX Subversion
G    nodes.pp
Updated to revision 264.

cd ../testing/manifests

svn up .
Authentication realm: &lt;https://svn.virginam.local:443&gt; VX Subversion
G    nodes.pp
Updated to revision 264.
</code></pre>