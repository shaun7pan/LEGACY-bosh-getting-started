These instructions are a combination of "Boot an AWS instance" and "Install BOSH via its chef_deployer"

h2. Setup

Install fog, \~/.fog credentials (for AWS), and \~/.ssh/id_rsa(.pub) keys

Install fog

```
gem install fog
```

Example \~/.fog credentials:

```
 :default:
  :aws_access_key_id:     PERSONAL_ACCESS_KEY
  :aws_secret_access_key: PERSONAL_SECRET
```
To create id_rsa keys:

```
$ ssh-keygen ```

h2. Boot instance

From Wesley's [fog blog post|http://www.engineyard.com/blog/2011/spinning-up-cloud-compute-instances/], boot a vanilla Ubunutu 64-bit image:

```
$ fog
  Welcome to fog interactive!
  :default provides AWS and VirtualBox
connection = Fog::Compute.new({:provider => 'AWS'})
server = connection.servers.bootstrap({
  :public_key_path => '~/.ssh/id_rsa.pub',
  :private_key_path => '~/.ssh/id_rsa',
  :username => 'ubuntu'
})
```

Or a big instance type to make everything go faster:

```
server = connection.servers.bootstrap({
  :public_key_path => '~/.ssh/id_rsa.pub',
  :private_key_path => '~/.ssh/id_rsa',
  :flavor_id => 'c1.xlarge', # 64 bit, high CPU
  :username => 'ubuntu'
})
```

Check that SSH key credentials are setup. The following should return "ubuntu", and shouldn't timeout.

```
server.ssh "whoami"
```

Now create an elastic IP and associate it with the instance. (I did this via the console).

```
address = connection.addresses.create
address.server = server
server.reload
address.public_ip
"107.21.120.243"
```

h2. Firewall/Security Group

FIXME/CHECK

You need to open ports 80 and 9022 to Internet. For AWS your Security Group will look like:

!https://img.skitch.com/20111212-nj6grrj6utrh9rx6qgcede75pp.png!

h2. Install Cloud Foundry

These commands below can take a long time. If it terminates early, re-run it until completion.

Alternately, run it inside screen or tmux so you don't have to fear early termination:

```
$ ssh ubuntu@107.21.120.243
# sudo apt-get install screen -y
# screen
sudo apt-get update
sudo apt-get install git-core build-essential libsqlite3-dev curl \
libmysqlclient-dev libxml2-dev libxslt-dev libpq-dev -y
curl -L get.rvm.io | bash -s stable
source ~/.bash_profile
rvm install 1.9.3 # though I think they prefer 1.8.7
git clone https://github.com/cloudfoundry/bosh.git