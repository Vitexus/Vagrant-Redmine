# -*- mode: ruby -*-
# vi: set ft=ruby :

# 1) Create an directory
# 2) Put this Vagrantfile and your Redmine (EasyRedmine/EasyProject) zip Archive into 
# 3) vagrant up
# 4) wait for provision to be done
# 5) run command: sudo -H -u easy bash -c ..
# 6) reboot the vagrant virtual machine
# 7) see application in action on http://localhost:8080

Vagrant.configure("2") do |config|
  config.vm.box = "debian/stretch64"
  config.vm.synced_folder ".", "/vagrant"
  config.vm.network "forwarded_port", guest: 80, host: 8080
  config.vm.provision "shell", inline: <<-SHELL
    RUBYVERSION="2.3.3"
    
    export DEBIAN_FRONTEND="noninteractive"
    
    apt update
    apt -y install curl mysql-server default-libmysqlclient-dev g++ libmagickcore-6.q16-dev libmagickwand-6.q16-dev libmagickwand-6-headers nginx-full

    #database 
    
    mysql -e "create database redmine CHARACTER SET utf8 COLLATE utf8_unicode_ci"
    mysql -e "GRANT ALL PRIVILEGES ON redmine.* TO redmine@localhost IDENTIFIED BY 'redmine'"
    
    
    #custom ruby
    
    gpg2 --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
    curl -sSL https://get.rvm.io | bash -s stable
    source /etc/profile.d/rvm.sh
    rvm install $RUBYVERSION 

    gem install bundler
    gem install redmine-installer --pre
    gem install unicorn

    #Webserver config
    
    cat << EOF > /etc/nginx/conf.d/unicorn-upstream.conf
upstream unicorn {                                                                                                                                                                                                                                                             
  server     unix:/srv/redmine/run/unicorn.sock  fail_timeout=10s;                                                                                                                                                                                                         
}

EOF


    DOLAR='$'
        cat << EOF >  /etc/nginx/sites-available/redmine.conf
server {
  listen *:80;
  server_name           _;
  client_max_body_size 999M;
  index  index.html index.htm index.php;

  access_log            /var/log/nginx/redmine.access.log combined;
  error_log             /var/log/nginx/redmine.error.log;

  location / {

    proxy_pass            http://unicorn;
    proxy_read_timeout    90;
    proxy_connect_timeout 90;
    proxy_redirect        off;
    proxy_set_header      X-Forwarded-For ${DOLAR}proxy_add_x_forwarded_for;
    proxy_set_header      Host ${DOLAR}http_host;
  }
}

EOF
 
    ln -s  /etc/nginx/sites-available/redmine.conf /etc/nginx/sites-enabled/redmine.conf
    
    gem install unicorn
        cat << EOF >  /etc/systemd/system/unicorn.service
[Unit]                                                                                                                                                                                                                                                                         
Description=Redmine Unicorn                                                                                                                                                                                                                                                            
                                                                                                                                                                                                                                                                               
[Service]                                                                                                                                                                                                                                                                      
Type=simple                                                                                                                                                                                                                                                                    
User=easy                                                                                                                                                                                                                                                                      
WorkingDirectory=/srv/redmine/public_html                                                                                                                                                                                                                                  
Environment=RAILS_ENV=production                                                                                                                                                                                                                                               
PIDFile=/srv/redmine/run/unicorn.pid                                                                                                                                                                                                                                       
ExecStart=/bin/bash -lc 'bundle exec unicorn_rails -D -c /etc/unicorn/redmine.rb -E production'                                                                                                                                                                            
                                                                                                                                                                                                                                                                               
[Install]                                                                                                                                                                                                                                                                      
WantedBy=multi-user.target                                                                                                                                                                                                                                                         

EOF

    
    mkdir /etc/unicorn
        cat << EOF >  /etc/unicorn/redmine.rb
    
worker_processes 4
stderr_path '/srv/redmine/public_html/log/stderr.log'
stdout_path '/srv/redmine/public_html/log/stdout.log'

listen '/srv/redmine/run/unicorn.sock'
pid "/srv/redmine/run/unicorn.pid"

timeout 300
preload_app true

before_fork do |server, worker|
    Signal.trap 'TERM' do
        puts 'Unicorn master intercepting TERM and sending myself QUIT instead'
    Process.kill 'QUIT', Process.pid
end

defined?(ActiveRecord::Base) and
    ActiveRecord::Base.connection.disconnect!
end

after_fork do |server, worker|
    Signal.trap 'TERM' do
        puts 'Unicorn worker intercepting TERM and doing nothing. Wait for master to send QUIT'
end

defined?(ActiveRecord::Base) and
    ActiveRecord::Base.establish_connection
end
   
EOF
 
        cat << EOF >  /tmp/Gemfile.local
gem 'unicorn'        
EOF
 
 
    rm -f /etc/nginx/sites-enabled/default
    systemctl enable nginx
    systemctl restart nginx
    systemctl enable unicorn

    
    
    
    adduser --disabled-password easy
    mkdir -p /srv/redmine/run/ /srv/redmine/log
    chown easy:easy /srv/redmine/ /usr/local/rvm/gems/ruby-2.3.3/wrappers  -R
    echo "easy ALL=NOPASSWD:ALL" > /etc/sudoers.d/easy

    echo "\n\nnow run: vagrant ssh"
    echo "and then\n\n"
    
    echo 'sudo -H -u easy bash -c "source /etc/profile.d/rvm.sh ; redmine install `ls /vagrant/*.zip`;cp /tmp/Gemfile.local /srv/redmine/public_html/"'
    
    echo "\n\nto finish (Easy)Redmine/Project installation enter:\n\n"
    echo "Path to redmine root: /srv/redmine/public_html"
    echo "mysql user:     redmine"
    echo "mysql password: redmine"
    echo "mysql database: redmine"
    echo "mysql port:     [enter]"
    echo "mysql encodng:  [enter]"
    
    echo "\nFinally start service unicorn and nginx or reboot vagrant machine"
    echo "\n\nopen http://localhost:8080/ to see web interface "
   
    
  SHELL
end

