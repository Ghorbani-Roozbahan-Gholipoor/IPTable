# IPTable
iptables Cookbook

Build Status Cookbook Version

Installs iptables and provides a custom resource for adding and removing iptables rules
Requirements
Platforms

    Ubuntu/Debian
    RHEL/CentOS and derivatives

Chef

    Chef 12.5+

Cookbooks

    none

Recipes
default

The default recipe will install iptables and provides a ruby script (installed in /usr/sbin/rebuild-iptables) to manage rebuilding firewall rules from files dropped off in /etc/iptables.d.
disabled

The disabled recipe will install iptables, disable the iptables service (on RHEL platforms), and delete the rules directory /etc/iptables.d.
Attributes

default['iptables']['iptables_sysconfig'] and default['iptables']['ip6tables_sysconfig'] are hashes that are used to template /etc/sysconfig/iptables-config and /etc/sysconfig/ip6tables-config. The keys must be upper case and any key / value pair included will be added to the config file.
Custom Resource
rule

The custom resource drops off a template in /etc/iptables.d after the name parameter. The rule will get added to the local system firewall through notifying the rebuild-iptables script. See Examples below.

NOTE: In the 1.0 release of this cookbook the iptables_rule definition was converted to a custom resource. This changes the behavior of disabling iptables rules. Previously a rule could be disabled by specifying enable false. You must now specify action :disable
Usage

Add recipe[iptables] to your runlist to ensure iptables is installed / running and to ensure that the rebuild-iptables script is on the system. Then create use iptables_rule to add individual rules. See Examples.

Since certain chains can be used with multiple tables (e.g., PREROUTING), you might have to include the name of the table explicitly (i.e., *nat, *mangle, etc.), so that the /usr/sbin/rebuild-iptables script can infer how to assemble final ruleset file that is going to be loaded. Please note, that unless specified otherwise, rules will be added under the filter table by default.
Examples

To enable port 80, e.g. in an my_httpd cookbook, create the following template:

# Port 80 for http
-A FWR -p tcp -m tcp --dport 80 -j ACCEPT

This template would be located at: my_httpd/templates/default/http.erb. Then within your recipe call:

iptables_rule 'http' do
  action :enable
end

To redirect port 80 to local port 8080, e.g., in the aforementioned my_httpd cookbook, create the following template:

*nat
# Redirect anything on eth0 coming to port 80 to local port 8080
-A PREROUTING -i eth0 -p tcp --dport 80 -j REDIRECT --to-port 8080

Please note, that we explicitly add name of the table (being *nat in this example above) where the rules should be added.

This would most likely go in the cookbook, my_httpd/templates/default/http_8080.erb. Then to use it in recipe[httpd]:

iptables_rule 'http_8080' do
  action :enable
end

To create a rule without using a template resource use the lines property:

iptables_rule 'http_8080' do
  lines '-A PREROUTING -i eth0 -p tcp --dport 80 -j REDIRECT --to-port 8080'
end

To get attribute-driven rules you can (for example) feed a hash of attributes into named iptables.d files like this:

node.default['iptables']['http_80'] = '-A FWR -p tcp -m tcp --dport 80 -j ACCEPT'
node.default['iptables']['http_443'] = [
  '# an example with multiple lines',
  '-A FWR -p tcp -m tcp --dport 443 -j ACCEPT',
]

node['iptables'].map do |rule_name, rule_body|
  iptables_rule rule_name do
    lines [ rule_body ].flatten.join("\n")
  end
end

Chefspec Matchers

    enable_iptables_rule(resource_name)
    disable_iptables_rule(resource_name)
