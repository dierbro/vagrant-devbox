# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  #config.vm.box = "saucy32"
  #config.vm.box = "base"
  config.vm.box = "precise32"

  # config.vm.network :forwarded_port, guest: 80, host: 8080

  config.vm.network :private_network, ip: "192.168.3.10"
  config.vm.hostname = "devbox"
  config.hostsupdater.aliases = ["api.devbox"]
  config.ssh.forward_agent = true

  config.vm.provider :virtualbox do |vb|
     vb.customize ["modifyvm", :id, "--memory", "2048"]
     vb.customize ["modifyvm", :id, "--cpus", "4"]   
     vb.customize ["modifyvm", :id, "--ioapic", "on"]
  end

  config.omnibus.chef_version = :latest

  config.vm.provision :chef_solo do |chef|
    chef.add_recipe 'nginx'
    chef.add_recipe 'nginx::apps'
    chef.add_recipe 'nginx::status'

    chef.add_recipe 'git'
    
    chef.add_recipe 'nodejs::install_from_package'

    chef.add_recipe "ruby_build"
    chef.add_recipe "rbenv::vagrant"
    chef.add_recipe "rbenv::user"

    chef.json = {
      rbenv: {
        user_installs: [
          {
            user: 'vagrant',
            rubies: ['2.0.0-p247'],
            global: '2.0.0-p247',
            gems: {
              '2.0.0-p247' => [
                { name: 'bundler' },
                { name: 'rails-api' }
              ]
            }
          }
        ]
      },
      nginx: {
        distribution: 'precise',
        components: ['main'],
        apps:{
        backend_devbox: {
          listen: [80],
          server_name: "api.devbox",
          #try_file: [
          #  "$uri @backend_devbox"
          #],
          locations: [
            {
              path: "/",
              directives: [
                "proxy_set_header X-Forwarded-Proto $scheme;",
                "proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;",
                "proxy_set_header X-Real-IP $remote_addr;",
                "proxy_set_header Host $host;",
                "proxy_redirect off;",
                "proxy_http_version 1.1;",
                "proxy_set_header Connection '';",
                "proxy_pass http://backend_devbox;"
              ]
            }
          ],
          upstreams: [
            {
              name: "backend_devbox", # defaults to your apps name (eg. myapp_ssl)
              servers: [
                "127.0.0.1:3000",
              ]
            }
          ]
        },
        frontend_devbox: {
          listen: [80],
          server_name: "devbox",
          #try_file: [
          #  "$uri @frontend_devbox"
          #],
          locations: [
            {
              path: "/",
              directives: [
                "proxy_set_header X-Forwarded-Proto $scheme;",
                "proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;",
                "proxy_set_header X-Real-IP $remote_addr;",
                "proxy_set_header Host $host;",
                "proxy_redirect off;",
                "proxy_http_version 1.1;",
                "proxy_set_header Connection '';",
                "proxy_pass http://frontend_devbox;"
              ]
            }
          ],
          upstreams: [
            {
              name: "frontend_devbox", # defaults to your apps name (eg. myapp_ssl)
              servers: [
                "127.0.0.1:8000",
              ]
            }
          ]
        }
        }
      }
    }
 
  end

end
