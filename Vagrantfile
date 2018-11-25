$provisionScript= <<SCRIPT
path_jenkins=/opt/jenkins
JENKINS_HOME=/opt/jenkins

sudo yum install -y net-tools vim java-1.8.0-openjdk-devel nginx
wget http://mirrors.jenkins.io/war-stable/latest/jenkins.war
useradd jenkins
mkdir -p  $path_jenkins
cp jenkins.war $path_jenkins
chown -R jenkins:jenkins $path_jenkins

#configure systemd startup script
sudo cat > /etc/systemd/system/jenkins.service <<EOF
[Unit]
Description=Jenkins
After=network.target
Requires=network.target
[Service]
User=jenkins
Group=jenkins
Environment=JENKINS_HOME=$JENKINS_HOME
ExecStart=/usr/bin/java -jar $path_jenkins/jenkins.war
Restart=always
RestartSec=10
[Install]
WantedBy=multi-user.target
EOF

# start a jenkins service
systemctl daemon-reload
systemctl start jenkins

#configure Nginx server for navigation to Jenkins without port
cat  > /etc/nginx/conf.d/jenkins.conf <<EOF
upstream jenkins {
                server 127.0.0.1:8080;
                }
server {
    listen      80;
    server_name jenkins;
#    access_log  $JENKINS_HOME/logs/access.log;
#    error_log   $JENKINS_HOME/logs/error.log;
    location / {
        proxy_pass  http://jenkins;
        proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
        proxy_redirect off;
        }
}
EOF

# start nginx service
systemctl start nginx
systemctl enable nginx
SCRIPT

Vagrant.configure("2") do |config|
  config.vm.box = "sbeliakou/centos"
  config.vm.box_version = "7.5"
  
  config.vm.define "jenkins_serv" do |jenkins|
    jenkins.vm.provider :"virtualbox" do |virt|
      virt.name = "jenkins"
      virt.memory = "4096"
    end
    jenkins.vm.hostname = "jenkins"
    jenkins.vm.network "private_network", ip: "192.168.100.2"
    jenkins.vm.provision :shell, inline: $provisionScript
    end
end
