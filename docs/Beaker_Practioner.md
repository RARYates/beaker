#Beaker Practicioner

[I don't want to know the pieces of beaker, just take me to an example](example)

The general purpose of Beaker Practictioner is to set up a basic beaker 
environment for a module. This will include setting up the following files:

* spec directory
* spec_helper_acceptance.rb
* spec/acceptance directory
* nodesets
* *_spec.rb (Number Ordering)

## Install Beaker

Information on installing Beaker can be found [here](tutorials/installation.md)

## A beaker run

A Beaker run performs the following steps:

1) Spin up node(s) in the nodesets directory
2) Apply all code in spec_helper_acceptance.rb to the nodes
3) Run all tests in spec/acceptance
4) Shut down node(s) in nodesets directory.

### Default Rake Task

require 'puppetlabs_spec_helper/rake_tasks' in the default Rakefile attached to
the module gives a rake task that applies all tests available for the module.
This command can be run via `rake beaker`.

This rake task is available by default when running `puppet module generate`

### Beaker Command Line

See beaker --help on the command line for all beaker options.

## Files 

A beaker test is per module, and can be declared and described in every module
uniquely.

* module/spec: Automated Tests
* module/spec/spec_helper_acceptance.rb: Global Config and Prerequisite setup
* module/spec/acceptance: Beaker Test Directory 
* module/spec/acceptance/*_spec.rb: Beaker Tests
* module/spec/acceptance/nodesets: Virtual Machines used for Testing

Beaker Acceptance tests are included on a per-module basis, and fall into
the standard Puppet autoload format. Beaker tests autoload from the spec
directory, shared with general puppet rspec tests. The Puppet autoload format
helps puppet determine where to find files, manifests, libraries, and tests on a
per-module basis.

### spec folder

All beaker related files will be found in <module>/spec.

### spec/spec_helper_acceptance.rb

<module>/spec/spec_helper_acceptance.rb is used to set your pre-test global
configuration and actions required before testing. Some examples include:

* Configuring RSpec settings for tests
* Installing Puppet
* Configuring Puppet Settings
* Installing and configuring dependency modules and packages

### spec/acceptance

This directory contains all Beaker tests and node configurations

### spec/acceptance/*_spec.rb

Files ending in \_spec.rb are the individual unit acceptance tests that Beaker
will perform. These will be performed in alphabetical order, and remnants of the
tests will remain on the configured host. These are applied to each host in the
nodesets.

A best practice is to name your tests beginning with numbers. For example,
00_test_spec.rb would come before 01_anothertest_spec.rb.

The order of tests is important, because artifacts and objects created by a test
remain in place for any follow on test. If 00_test_spec.rb creates /tmp/file,
then 01_anothertest_spec.rb can reference that file.

### nodesets

Nodesets are YAML files designed to describe the Virtual Machine you intend to
perform the tests on. Consider this the configuration file for the host you
intend on perform the tests on. By default, the configuration default.yaml will 
be used to select the node(s) the tests are performed on.

## Example

### Build our spec_helper_acceptance.rb

Here is an example placed at <module>/spec/spec_helper_acceptance.rb:

```ruby
require 'beaker-rspec/spec_helper'
require 'beaker-rspec/helpers/serverspec'
require 'beaker/puppet_install_helper'

run_puppet_install_helper


RSpec.configure do |c|
  proj_root = File.expand_path(File.join(File.dirname(__FILE__), '..'))

  c.formatter = :documentation

  c.before :suite do
    hosts.each do |host|

      environmentpath = host.puppet['environmentpath']
      destdir = "#{environmentpath}/production/modules"
      on host, 'git clone -b master https://git.net/org/example #{destdir}/example

      copy_module_to(host, :source => proj_root, :module_name => 'example')

    end
  end
end
```
 
#### Ruby Dependencies (require)

Like any ruby file, rubygems can be include with the require statement. This
allows use of the gem in the code.

In this example, the following are used:

* beaker-rspec/spec_helper: Beaker specific
* beaker-rspec/helpers/serverspec: Allows the use of serverspec
* beaker/puppet_install_helper: Installs puppet agent

The run_puppet_install_helper command uses the code provided by
beaker/puppet_install_helper

```ruby
require 'beaker-rspec/spec_helper'
require 'beaker-rspec/helpers/serverspec'
require 'beaker/puppet_install_helper'

run_puppet_install_helper
```

#### RSpec Configuration

* `RSpec.configure do |c|`: Start the configuration of RSpec, name it 'c'
* `proj_root`: 

```ruby
RSpec.configure do |c|
  proj_root = File.expand_path(File.join(File.dirname(__FILE__), '..'))

  c.formatter = :documentation

  c.before :suite do
    hosts.each do |host|

      environmentpath = host.puppet['environmentpath']
      destdir = "#{environmentpath}/production/modules"
      on host, 'git clone -b master https://git.net/org/example #{destdir}/example

      copy_module_to(host, :source => proj_root, :module_name => 'example')

    end
  end
end
```

### Apply Puppet

### Check for values

