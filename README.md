# Ansible dynamic chef inventory

Query chef API server to know about available servers. Can be useful if most of your automation is done in chef but you wish to run some scripts via ansible. See the [documentation on dynamic inventory](http://docs.ansible.com/ansible/intro_dynamic_inventory.html) for more details on the 'dynamic inventory' concept.

## Requirements:

* [Ansible](http://docs.ansible.com/ansible/) [this script was tested with ansible 2.2.0.0]
* PyChef (`pip install PyChef`)
* Openssl Dev Libs (`apt-get install libssl-dev`)

You can set the following environment variables for the dynamic ansible inventory to pull custom setting. Otherwise this script will pull the setting from a knife.rb in either ~/.chef/knife.rb or ./.chef/knife.rb.

```
export CHEF_USER=john
export CHEF_PEMFILE=/home/john/.chef/john.pem
export CHEF_SERVER_URL="https://my-chef-api"
export CHEF_SERVER_SSL_VERIFY=true
```

(alternatively you can put the settings into a chef.ini file living in the same directory as the chef_inventory.py file)

You might find the information under `~/.chef/knife.rb` or `~/.chef/knife_local.rb`

Copy chef_inventory.py to a location of your convenience, configuring your `ansible.cfg` to include

```
hostfile=chef_inventory.py
```

or point to it when running your scripts with

```
ansible-playbook -i /path/to/chef_inventory.py
```

Make sure to make the file executable. (`chmod +x chef_inventory.py`)

## Usage

try it with
```
./chef_inventory.py --list
```

### dynamic inventory details:

This python script `chef_inventory.py` first queries the chef API to find servers, and caches the results in your home directory at `~/.ansible-chef.cache`.
That file is kept for 60 minutes (feel free to change `self.cache_max_age = 3600` in the python code)
passing `--list` then extracts server's IPs in the format ansible expects, e.g.

```
{
"role_my-webserver-group": [
    "10.0.0.1",
    "10.0.0.2"
  ],
"chef_environment_aws": [
    "10.0.0.3",
    "10.0.0.4",
    "10.0.0.5"
  ]
}
```

Within ansible playbooks or roles, you can then refer to `hosts: my-webserver-group`, and name folders under `group_vars` also `my-webserver-group`, which will operate on those servers.

Additional query parameters can be used. This script can be used to search for chef_environments or roles or recipes either in the run-list or included in the roles or by tags.

#### Examples
```
ansible 'chef_environment_aws' --list-hosts

ansible 'role_webservers' --lists-hosts

ansible 'tag_tomcat' --list-hosts
```

force cache refresh: `./chef_inventory.py --refresh-cache`

## Thanks

Code based on https://gist.github.com/tjheeta/f3538c32965575e59bcd with some modifications to work with a recent version (2.2.0) of ansible and a custom chef setup, as well as to remove duplicate servers in the end result.
