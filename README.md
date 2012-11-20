EC2-snapshot - create snapshots from all mounted EBS volumes
============================================================

EC2-snapshot is gem that allows you to easily create snapshots for mounted EBS volumes on
an EC2 instance.

The idea originated from the wish of automating snapshot creation for EC2 instances and making
it easy to integrate it into Chef.

EC2-snapshot was written with the idea of it running only on the EC2 instance for which snapshots
need to be created. Because we wanted to use it with Chef, we needed to keep configuration to a minimum.
There is no need to provide an instance id or volume ids, as that will already be retrieved on the 
instance itself. The only necessities are the AWS credentials, region and an option to set which types
of volumes need to be snapshotted.

Therefore, the current implementation doesn't support the case of using a snapshotting server that creates snapshots 
for all volumes. It is meant to run on all servers that require snapshots to be created of its volumes.


Features
--------

* No need to specify instance id and volume ids while using the gem
* Recognizes XFS filesystems, and looks up mount points automatically
* Freezes XFS filesystems while creating the snapshot, resulting in a consistent snapshot. If running on kernels > 2.6.29, freezes a set of supported filesystems
* Easy integration within Chef, requiring a very simple recipe that only requires (globally defined) AWS credentials
* Automatically detects instance region
* Easy integration within your own scripts with either the executable or by instantiating the `Ec2Snapshot::Instance` class yourself
* Custom actions that need to be executed before and/or after the snapshot is created can be easily configured
* Allows temporal (i.e. snapshots older than 1 month) and numeric (i.e. the last 7 snapshots) snapshot rotation
* Skips snapshots if there is a pending one for a specific block device

Requirements
------------

There are some requirements for using this gem:

* The gem only works on Linux, as it has dependencies on files such as `/proc/mounts`
* `wget` needs to be installed. This is required to automatically retrieve the current instance id
* `xfs_freeze` (included in xfsprogs package) needs to be installed in order to be able to freeze a XFS filesystem to get a consistent snapshot
* `fsfreeze`(included in util-linux-ng package) needs to be installed in order to be able to freeze a generic filesystem to get a consistent snapshot (supported for kernels > 2.6.29)


Getting started
---------------

Installing the Gem is pretty straightforward:

	gem install ec2-snapshot

Note that `hostname -f` is used to get the name of the current instance.
Also `/proc/mounts` is used to retrieve information on the filesystems to be snapshotted.


Using the executable
--------------------

An executable has been provided to easily use the gem within your own scripts.

The executable requires a few mandatory details to be able to use your AWS account. These are:

* `AWS Access Key`: The access key defaults to `ENV["AWS_ACCESS_KEY_ID"]`. 
If the environment variable is not set the value should be provided as an option while using the executable, 
ie. `--aws-access-key KEY`
* `AWS Secret Access Key`: The secret access key defaults to `ENV["AWS_SECRET_ACCESS_KEY"]`.
If the environment variable is not set the value should be provided as an option while using the executable, 
ie. `--aws-secret-access-key KEY`
* `AWS Region`: The region on which the volumes have been created. Needs to be provided as an option, ie. `--aws-region eu-west-1`.
If you do not provide it, the script will automatically try to detect it via standard AWS methods.

The gem makes a distinction between root volumes and data volumes. The root volume is the volume on which the OS 
is installed, while the data volumes are other volumes mounted on the same instance that could for example be used to store 
application specific data.

By default, EC2-snapshot will attempt to create snapshots of all volumes mounted on the 
current instance. In case you only need snapshots of the data volumes, which could be a valid case when using Chef, 
you can easily specify that by using the `--volume-type` option:

	ec2-snapshot --aws-access-key ACCESS_KEY --aws-secret-access-key KEY --aws-region us-west-1 --volume-type data

If you want however, you can fully define the devices you may need snapshotted in a full path form (ie. /dev/sdb). 
The script will take care of device naming conventions for PVOPS kernels (sd -> xvd).

For a complete list of supported options, please execute
	
	ec2-snapshot -h

Take note that `xfs_freeze` or `fsfreeze` requires the process to run with root privileges (sudo), otherwise the XFS filesystem 
cannot be frozen.


Security
--------

The most secure way of using EC2-snapshot is to create a new user with AWS IAM and allow that user to only execute a 
specific set of actions.

The following actions are required:

* ec2:CreateSnapshot
* ec2:CreateTags
* ec2:DeleteSnapshot
* ec2:DescribeInstances
* ec2:DescribeSnapshots

By restricting the user to only these actions, you can make sure that the access keys required for EC2-instance cannot 
be misused to manipulate the instances themselves.


Inspiration
-----------

EC2-snapshot was inspired by [ec2-consistent-snapshot](https://launchpad.net/ec2-consistent-snapshot) and 
its Ruby port [ec2-consistent-snapshot-rb](http://rubygems.org/gems/ec2-consistent-snapshot-rb).


About EC2-snapshot
------------------

EC2-snapshot was created by Mattijs van Druenen (m.vandruenen *at* voormedia.com)

Copyright 2011 Voormedia - [www.voormedia.com](http://www.voormedia.com/)

Additional contributions by Panagiotis Papadomitsos (pj *at* ezgr.net)

License
-------

EC2-snapshot is released under the MIT license.