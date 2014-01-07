---
layout: post
title: "Vagrant: Your Own Personal Cloud Provider"
description: ""
category: ""
tags: []
---
{% include JB/setup %}

In this blog post, I'd like to highlight the potential and use of the [Vagrant](http://vagrantup.com) virtualization tool as the first step in moving to and automating applications in the "cloud." When you are developing DevOps code, and adapting application code and deployment tools, the build cycle usually involves starting from a base disk image, and configuring all the way up the stack: system, daemons, applciation runtime, application, and application code. Doing this in the "cloud" can be pricey and slow, as well as unnecessary: debugging a puppet build in Amazon EC2 or on a local VM will be same the vast majority of the time. Using Vagrant as a development environment can make this development cycle faster and more convenient.

## Vagrant Overview

[Vagrant](http://vagrantup.com) functions as an abstraction and convenience wrapper around virtualization environments such as VirtualBox and VMware, with plugins for other environments such as [Amazon EC2](http://aws.amazon.com/ec2/). It provides a unified way to startup, interact with, and manage VMs on your local machines programmatically or from the command line. At this time, Vagrant is cross platform, but guest support is pretty much Linux-focussed, and while you can start Windows or other OS images with it, it doesn't manage or configre those VMs.

Configuration and setup is done via the `Vagrantfile`, a ruby-syntax configuration file through which you specify the state of the deployed system. Vagrant is also well-integrated with configuration management toools like Puppet or Chef, and system builds are intended to be automatically started through the `Vagrantfile` itself. This lets you capture as code the state and configuration of a set of VM resources, and programmtically manage applications within the started VMs, turning the setup of a set of services on a server, or even a set of servers, into a straightforward build process.

A typical Vagrant session looks like this:

    $> vagrant init mybox http://(url to my).box
    
    $> vagrant up 
     
    $> vagrant provision
     
    $> vagrant ssh
     
    $> vagrant halt
     
    $> vagrant destroy

Here we've created a basic template `Vagrantfile` that uses a specific disk image ( i.e. a vagrant "box" ). We then started `up` a VM using that disk image (this step will take a while, since ). The next step will `provision` the VM lby running any puppet or chef code that we may have specified in the 
`Vagrantfile` (in our case nothing will happen). 

Once up and provisioned, we can `ssh` into the VM; vagrant manages all the keys and sshd port details for us. Once done, we `halt` and then `destroy` the VM.

## Vagrant as Cloud Provider

Vagrant is a local system that lets you easible automate the setup of VMs and related resources. While far from as feature complete as a true cloud platform, it provides services that come close enough to serve as useful local stand-ins for more complete services, paving the way for transitioning later on in the development cycle to true cloud providers. Let's compare the functionality of Vagrant to [Amazon Web Services](http://aws.amazon.com)offerings, and to the services provided via the [OpenStack](http://www.openstack.org) platform.

**Compute**: Vagrant lets you start up any number of virtual machines, and choose the # of cores, RAM, and network interfaces, and provides the basic compute resource capabilities of Amazon EC2 or OpenStack Nova.

**Image**: Vagrant "boxes" serve the role of Amazon AMIs or OpenStack Glance-managhed disk images. Via the "box" functionality, one can specify and manage a base system image and architecture.

**Networking**: Vagrant uses the virtualization providers fucntionality to setup a basic NAT'd network on the local host, and for the core providers can also create additional networks and interfaces on VMs, allowing for the specificatin of private networks between VMs and a "public" networks with access to the local machine. This lets you specify basic multi-tier, multi-VM applications within vagrant, providing some of the basic virtual datacenter functionailty of Amazon's VPC services, or of OpenStack's (Neutron)[http://www.openstack.org/software/openstack-networking/]  (nee Quantum).

**Block Storage**: Vagrant doesn't provide a full networked block storage capability, but does allow local directories to be [mounted on deployed VMs](http://docs.vagrantup.com/v2/getting-started/synced_folders.html). This is particularly useful for running Puppet or Chef code, and for deploying application on a VM. This simple functionality is a weaker equivalent to Amazon's EBS persistent block storage, or OpenStack's Cinder block storage services.

**Access & Security**: Both Amazon EC2 and OpenStack's Nova alow you to specify an SSH key to be installed at boot time, and provide an external IP address for the user to SSH to once a machine is up. Vagrant configures, manages and tracks a key on your behalf as well, allowing you to `vagrant ssh` from the directory in which the `Vagrantfile` lies.

In addition to SSH terminal access, Amazon EC2 and OpenStack allow you to specify "security groups" which provide a general port mapping table from external clients to the guest. Vagrant allows this management as well via port-forwarding directives in the `Vagrantfile`.

**Orchestration**: The `Vagrantfile` itself represents the spefication file for deploying and managing any number of VM and other resourcs within vagrant, while the hooks into puppet or chef allow you to further orchestrate the system and application configuration. The equivalent capability in Amazon Web Services is through [CloudFormation](http://aws.amazon.com/cloudformation/), and for OpenStack it's the emerging [Heat](https://wiki.openstack.org/wiki/Heat) service.

## Examples 

Here's a basic example of building a simple webserver and adding a basic 
`index.html` file. This uses basic scripting for simplicity; more advanced setups would use a 

### Vagrant

Here we build a system based on CentOS 6.5, run Apache, and expose the web port to localhost on port 8080. After saving this to a file named `Vagrantfile` and running `vagrant up` you should be able to connect to

- (http://127.0.0.1:8080)

on your local machine to see a simple one-liner webpage.

```ruby
VAGRANTFILE_API_VERSION = "2"

box      = 'centos6p5'
url      = 'https://github.com/2creatives/vagrant-centos/releases/download/v6.5.1/centos65-x86_64-20131205.box'
ram      = '615'

$script = <<SCRIPT
yum -y install httpd
chkconfig httpd on
service httpd start
f=/var/www/html/index.html
rm -f $f
echo "<html><head><title>Vagrant built example system</title><body>" > $f
echo "<pre>System $(hostname) built on $(date).</pre>" >> $f
echo "</body></html>" >> $f
SCRIPT

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  config.vm.box_url = url
  config.vm.box     = box

  config.vm.network "forwarded_port", guest: 80, host: 8080

  config.vm.synced_folder ".", "/vagrant_data"

  config.vm.provider :virtualbox do |vb|
    vb.customize [ 'modifyvm', :id, '--memory', ram]
  end

  config.vm.provision "shell", inline: $script

end
```

### Amazon CloudFormation

Here we do the same, but in Amazon EC2 using CloudFormation (see various examples [here](http://aws.amazon.com/cloudformation/aws-cloudformation-templates/) ). For simpicity, much of the settings are hard-coded, which makes the templates less useful, much significantly shorter.

Using the newer AWS CLI tools, launch this "stack" using the command:

     aws cloudformation create-stack \
         --stack-name example \
         --parameters  ParameterKey=KeyName,ParameterValue=parrott \
         --template-body "$(cat cf.json)" 

where you've saved the following CloudFormation JSON template file as `cf.json` and you have replaced the SSH key name `parrott` with our own value as configured in Amazon EC2. 

The template follows below. Note that because Amazon is a multi-tenant web-services environment instead of a local wrapper environment, this specification file tends to be _much_ longer and complicated.

```json
{
  "AWSTemplateFormatVersion" : "2010-09-09",

  "Description" : "AWS CloudFormation simple website.",

  "Parameters" : {

    "KeyName": {
      "Description" : "Name of an existing EC2 KeyPair to enable SSH access to the instance",
      "Type": "String",
      "MinLength": "1",
      "MaxLength": "255",
      "AllowedPattern" : "[\\x20-\\x7E]*",
      "ConstraintDescription" : "can contain only ASCII characters."
   }
  },

  "Resources" : {

    "Ec2Instance" : {
      "Type" : "AWS::EC2::Instance",
      "Properties" : {
        "SecurityGroups" : [ { "Ref" : "InstanceSecurityGroup" } ],
        "KeyName" : { "Ref" : "KeyName" },
        "ImageId" : "ami-1b814f72",

        "UserData"       : { "Fn::Base64" : { "Fn::Join" : ["", [
          "#!/bin/bash\n",
          "yum update -y aws-cfn-bootstrap\n",

          "/opt/aws/bin/cfn-init -s ", { "Ref" : "AWS::StackId" }, " -r Ec2Instance ",
          "         --region ", { "Ref" : "AWS::Region" }, "\n",
          "yum -y install httpd\n",
          "chkconfig httpd on \n",
          "service httpd start\n",
          "f=/var/www/html/index.html\n",
          "rm -f $f\n",
          "echo \"<html><head><title>Vagrant built example system</title><body>\" > $f\n",
          "echo \"<pre>System $(hostname) built on $(date).</pre>\" >> $f\n",
          "echo \"</body></html>\" >> $f\n",

          "/opt/aws/bin/cfn-signal -e $? '", { "Ref" : "WaitHandle" }, "'\n"
        ]]}}        
      }
    },

    "WaitHandle" : {
      "Type" : "AWS::CloudFormation::WaitConditionHandle"
    },

    "WaitCondition" : {
      "Type" : "AWS::CloudFormation::WaitCondition",
      "DependsOn" : "Ec2Instance",
      "Properties" : {
        "Handle" : {"Ref" : "WaitHandle"},
        "Timeout" : "300"
      }
    },

    "InstanceSecurityGroup" : {
      "Type" : "AWS::EC2::SecurityGroup",
      "Properties" : {
        "GroupDescription" : "Enable SSH access and HTTP access on the inbound port",
        "SecurityGroupIngress" :
          [{ "IpProtocol" : "tcp", "FromPort" : "22", "ToPort" : "22", "CidrIp" : "0.0.0.0/0"},
           { "IpProtocol" : "tcp", "FromPort" : "80", "ToPort" : "80", "CidrIp" : "0.0.0.0/0"} ]
      }
    },

    "EIP" : {
      "Type" : "AWS::EC2::EIP",
      "Properties" : {
        "InstanceId" : { "Ref" : "Ec2Instance" }
      }
    }
  },

  "Outputs" : {
    "URL" : {
      "Description" : "URL of the sample website",
      "Value" : { "Fn::Join" : [ "", [ "http://", { "Ref" : "EIP" }]]}
    }
  }
}
```

## Behold: The Power of Nepho!

One issue with the configs above is that the _same_ script is duplicated in each orchestration file. Thus any change must be duplicated. Now let's add in Google Compte Engine, and add in another cloud provider, and a much more copmlex, multi-host environment, and pretty soom we've radically grown in complexity. If only we had a tool to reduce this complexity, and decouple the infrastructure setup from the system and application setup.

Enter [nepho](http://github.com/huit/nepho).

The nepho tool is intended to help decouple system and application configuration management from infrastructure configuration (i.e. IaaS). It does this by defining a set of bootstrap hooks where the build process can "hand-off" the build to a largely cloud-agnostic configuration tool, but also keeping these cloud provider configuration bundled into a single "cloudlet" codebase.

To see how this works, take a look at the sample cloudlet that implements the example above, but using nepho:

- https://github.com/cloudlets/simple-website.git

Look for more posts on how this works in the future.
