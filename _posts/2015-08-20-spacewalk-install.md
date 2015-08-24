---
layout: post
title: "Spacewalk Install"
date: 2015-08-20
permalink: /linuxadmin/spacewalk-install
---

Description: Install a Spacewalk server. Use CentOS 6 as the distro.

Bonus: set up errata importation on the CentOS channels, so you can properly see security update advisory information.

<div style="display: inline-block">
<div class ="panel panel-info" style="border-style: solid">
<div class="panel-heading">Contents</div>
<div class="panel-body">
<a href="#todo">ToDo</a><br />
<a href="#bonus">Bonus</a>
</div>
</div>
</div>


***

###System Setup

* CentOS 6.6
	* http://mirror.centos.org/centos/6/os/x86_64/ 
	* 80 gb Drive 1
		* 500 mb /boot
		* LVM
			* 10 gb swap
			* rest /
	* minimal install

***

### <a name="todo"></a>ToDo
