# TLS/SSL
TLS/SSL provides us an extra layer of transport security security. HTTPS is not enabled by default for this project. Please add a reverse proxy in front of the KMS and the Backend servers to enable HTTPS. All modules support the use of HTTPS.

To enable HTTPS and enable automatic certificate renewal please visit your certificate authority for instructions on obtaining certificates. 

To enable HTTPS (follow these instructionf for the KMS and backend servers):
1. add reverse reverse proxy to server, Nginx or Apache
2. Make KMS and Backend bind to local interface (for safety). That is so the HTTP services are only exposed and available to the proxy.
   - edit `docker-compose.yml` file in root of respective module
   - for the KMS and backend look under `ports:` (not under the MongoDB ports service) the line that look like  from `- "<INTERFACE_IP>:50XX:50YY"` to `- "127.0.0.1:50XX:50YY"`, where in `- "<INTERFACE_IP>:<bind_port>:<docker_port>"` interface is the bind location for the service, bind port is the port that it will be made available, and docker port is the internal docker service the service originates from, the docker port should remain unchanged. Important to note the interface can not be the hostname `localhost`(it must the the IP of the interface localhost), for example `127.0.0.1` is valid. For more information look at the [Docker Port Documentation](https://docs.docker.com/config/containers/container-networking/#published-ports) and [Docker compose documentation](https://docs.docker.com/compose/compose-file/compose-file-v3/#ports).
3. point the reverse proxy servers to each respective service:
```
TODO
PUT CONFIG HERE 
...
location / {
    proxy_pass http://127.0.0.1:50XX; #<-- change port to match bind port in docker-compose.yml
}
...
```
```
TODO
PUT CONFIG HERE 
```
4. modify reverse proxy configuration to use the SSL certificates
   - If using let's encrypt: run the certbot tool
5. configure autorenewal of certificates
   - if using let's encrypt and certbot, a `systemd` timer has been added to the system for autorenewal. Please follow the instructions on cerbot's  instructions page

More information:
- [Nginx](https://docs.nginx.com/nginx/admin-guide/)
  - [reverse proxy](https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/)
- [let's encrypt](https://letsencrypt.org/)
- [cerbot](https://certbot.eff.org/)

```



```

# Securing Services from unauthorized access

Each module supports basic authentication as an extra layer of security. This will act as a service gatekeeper.

To enable:
1. In each server (KMS and backend), modify the reverse proxy configuration to enable the use of basic authentication. More info [here](https://docs.nginx.com/nginx/admin-guide/security-controls/configuring-http-basic-authentication/)
   - add a user to the `.htaccess` file
   - add the exact same user with the same password to both KMS and Backend
2. Configure basic authentication credentials on all the modules(except KMS and backend) by editing the configuration file. The basic authentication stanzas should be located in the example configuration file. Add these to your current configurations and uncomment them. Fill out with the credentials added from the previous step
   
3. If a module is already running, send SIGINT signal and wait for the service to gracefully exit(to avoid loss data). Then start again using the new configuration




```{admonition} Warning
:class: warning

The backend server API has no authentication, therefore it is very important to add this layer of security(unless deploying for testing/development purposes).
```

```{admonition} Must configure modules after enabling
:class: important

important note: that after enabling basic authentication(step 1), everyone using the system must follow this step(step 2 and 3), else modules will exit because they can not connect to servers
```

