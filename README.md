# 0. server's functions

| function | VM | ipaddress | internal | container's port |
| --- | --- | --- | --- | --- |
| nginx | VM#1 | 192.168.0.1:8080 | 172.17.0.1:8080 | 80 | 
| mongo | VM#2 | 192.168.0.1:27017 | 172.17.0.1:27017 | 27017 | 
| emp0 | VM#2 | 192.168.0.2:5001 | 172.17.0.1:5001 | 5001 |
| emp1 | VM#2 | 192.168.0.2:5011 | 172.17.0.1:5011 | 5001 |
| emp2 | VM#2 | 192.168.0.2:5021 | 172.17.0.1:5021 | 5001 |
| emp3 | VM#2 | 192.168.0.2:5031 | 172.17.0.1:5031 | 5001 |
| emp4 | VM#2 | 192.168.0.2:5041 | 172.17.0.1:5041 | 5001 |
| emp5 | VM#2 | 192.168.0.2:5051 | 172.17.0.1:5051 | 5001 |
| emp6 | VM#2 | 192.168.0.2:5061 | 172.17.0.1:5061 | 5001 |
| emp7 | VM#2 | 192.168.0.2:5071 | 172.17.0.1:5071 | 5001 |


# 1. VM's IP address settings
```
select Adapter2 and select "intnet"
```

# 2. asign ip address on "enp0s8" in each VM
```
$ cat /etc/netplan/99_config.yaml 
# This is the network config written by 'subiquity'
network:
  ethernets:
    enp0s8:
      dhcp4: false
      addresses:
         - 192.168.0.1/24 
  version: 2

$ sudo reboot
```

# 3. Port Forwarding 
See also https://qiita.com/tukiyo3/items/94eda73b951d23b214c3.
But this step might not be necessary. Please check the result of iptables so that you can find the port forwarding.

# 4. VM#1@192.168.0.1 (nginx config file and run mongodb)
```
$ sudo docker volume create nginx_conf2
$ sudo docker inspect nginx_conf2
[
    {
        "CreatedAt": "2021-03-08T07:31:19Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/snap/docker/common/var-lib-docker/volumes/nginx_conf2/_data",
        "Name": "nginx_conf2",
        "Options": {},
        "Scope": "local"
    }
]
$ sudo cat /var/snap/docker/common/var-lib-docker/volumes/nginx_conf2/_data/default.conf
upstream proxy.com {
	server 192.168.0.2:5001;
	server 192.168.0.2:5011;
	server 192.168.0.2:5021;
	server 192.168.0.2:5031;
	server 192.168.0.2:5041;
	server 192.168.0.2:5051;
	server 192.168.0.2:5061;
	server 192.168.0.2:5071;
}

server {
	listen 80;
	server_name localhost;
	location / {
		root /usr/share/nginx/html;
		index index.html index.htm;
		proxy_pass https://proxy.com;
	}
}

$ sudo docker run -itd -p 8080:80 --rm -v nginx_conf2:/etc/nginx/conf.d --name="nginx" nginx:latest
$ sudo docker run -itd -p 27017:27017 --rm --name="mongodb" mongo:latest
(or sudo docker run -itd -p 27017:27017 --rm --name="mongodb" -v sample_mongo:/data/db mongo:latest)
```

# 5. VM#2@192.168.0.2 (run Employee Application)
```
sudo docker run -itd -p 5001:5001 -p 5000:5000 --env MONGO="192.168.0.1" --name="emp0" --rm employee:latest /usr/local/dotnet/publish/Employee
sudo docker run -itd -p 5011:5001 -p 5010:5000 --env MONGO="192.168.0.1" --name="emp1" --rm employee:latest /usr/local/dotnet/publish/Employee
sudo docker run -itd -p 5021:5001 -p 5020:5000 --env MONGO="192.168.0.1" --name="emp2" --rm employee:latest /usr/local/dotnet/publish/Employee
sudo docker run -itd -p 5031:5001 -p 5030:5000 --env MONGO="192.168.0.1" --name="emp3" --rm employee:latest /usr/local/dotnet/publish/Employee
sudo docker run -itd -p 5041:5001 -p 5040:5000 --env MONGO="192.168.0.1" --name="emp4" --rm employee:latest /usr/local/dotnet/publish/Employee
sudo docker run -itd -p 5051:5001 -p 5050:5000 --env MONGO="192.168.0.1" --name="emp5" --rm employee:latest /usr/local/dotnet/publish/Employee
sudo docker run -itd -p 5061:5001 -p 5060:5000 --env MONGO="192.168.0.1" --name="emp6" --rm employee:latest /usr/local/dotnet/publish/Employee
sudo docker run -itd -p 5071:5001 -p 5070:5000 --env MONGO="192.168.0.1" --name="emp7" --rm employee:latest /usr/local/dotnet/publish/Employee
```
