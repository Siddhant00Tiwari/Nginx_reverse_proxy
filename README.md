# Nginx_reverse_proxy
We will perform this project in ***RHEL 9*** instances, where one instance will host a ***HTTP server*** and other our ***NGINX proxy server.***

### First, we will start with installing http server in our first RHEL instance.

1. Install the package using `sudo dnf install httpd` , then make an `index.html` file with your desired code. For my case I used this code
    
     
    
    ![html_file.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/77ead240-a866-4aa1-883b-179fa59f3972/f25ee419-b6d4-4ddd-bef1-c81bf3e7a018/html_file.png)
    
2. Then we’ll enable it using `sudo systemctl enable --now httpd`

> **!**  If you are using a local virtual machine you’ll need to add httpd service in firewall with `firewall-cmd --add-service=http` , for my case I’m using an AWS instance so I would need to add the inbound rule in *security groups.*
> 

### Then, we will install nginx on our second RHEL instance.

1. Make a repo file `/etc/yum.repo.d/nginx.repo` 
    
    ```bash
    [nginx-stable]
    name=nginx stable repo
    baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
    gpgcheck=1
    enabled=1
    gpgkey=https://nginx.org/keys/nginx_signing.key
    module_hotfixes=true
    
    [nginx-mainline]
    name=nginx mainline repo
    baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
    gpgcheck=1
    enabled=0
    gpgkey=https://nginx.org/keys/nginx_signing.key
    module_hotfixes=true
    ```
    
    - nginx-mainline is disabled by default
    - module_hotfixes are the patches applied to them by themselves or third-party
    
2. Then install nginx with `sudo yum install nginx` 
3. You would need to enable the nginx service with `sudo systemctl enable —now nginx`
4. Then make sure the server is working with `curl -I 127.0.0.1`
    
    ```bash
    HTTP/1.1 200 OK
    Server: nginx/1.24.0
    Date: Wed, 14 Feb 2024 12:52:40 GMT
    Content-Type: text/html
    Content-Length: 615
    Last-Modified: Tue, 11 Apr 2023 17:23:43 GMT
    Connection: keep-alive
    ETag: "6435979f-267"
    Accept-Ranges: bytes
    ```
    
    If you get a response like this, nginx is working.
    

### Then, we will write our nginx configuration.

We will remove the `/etc/nginx/conf.d/default.conf` file that is create by defualt, and create a `test.conf` file in it’s place with configuration 

 

![nginx_configuration.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/77ead240-a866-4aa1-883b-179fa59f3972/f6b4f6d0-3e61-4ba5-8543-513cba4e2d91/nginx_configuration.png)

- here we will write this simple configuration under server block.
- first listen is for IPv4
- second listen is for IPv6
- server_name is our domain name for our nginx server
- in location block we’ll specify from where nginx is suppose get our document for hosting
    - proxy_pass is used to specify nginx that it’ll be used as proxy and the documents are at http://{ip_address}:{port_number};

After this we’ll enable nginx service using `sudo systemctl enable --now nginx`

> If you try to launch the nginx server and hope to find our http server, it’ll show an error page. Because in RHEL OS there’s a security system call SELINUX which inhibits us from using this proxy service. So, for that we’ll need to turn the boolean on for `httpd_can_network_connect` using `semanage boolean -m --on httpd_can_network_connect` on both of our instances.
> 

### Now we will launch our nginx website.

While searching for  nginx IP, we’ll be greeted with our http website.

![website.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/77ead240-a866-4aa1-883b-179fa59f3972/e25c70fe-0074-4c98-974c-047b4663d341/website.png)
