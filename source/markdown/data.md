# TLS/SSL
TLS/SSL provides us an extra layer of transport security security. HTTPS is not enabled by default for this project. Please add a reverse proxy in front of the KMS and the Backend servers to enable HTTPS. All modules support the use of HTTPS.

To enable HTTPS and enable automatic certificate renewal please visit your certificate authority for instructions on obtaining certificates. 

In a nutshell, to enable HTTPS we change the server bindings to a local only interface and then we configure the reverse proxy with HTTPS to point to our local interface. 

To enable HTTPS (follow these instructions for the KMS and backend servers):
1. add reverse reverse proxy to server, Nginx or Apache
2. Make KMS and Backend bind to local interface (for safety). That is so the HTTP services are only exposed and available to the proxy.
    - edit `docker-compose.yml` file in root of respective module
    - for the KMS and backend look under `ports:` (not under the MongoDB ports service) the line that look like  from `- "<INTERFACE_IP>:50XX:50YY"` to `- "127.0.0.1:50XX:50YY"`, where in `- "<INTERFACE_IP>:<bind_port>:<docker_port>"` interface is the bind location for the service, bind port is the port that it will be made available, and docker port is the internal docker service the service originates from, the docker port should remain unchanged. Important to note the interface can not be the hostname `localhost`(it must the the IP of the interface localhost), for example `127.0.0.1` is valid. For more information look at the [Docker Port Documentation](https://docs.docker.com/config/containers/container-networking/#published-ports) and [Docker compose documentation](https://docs.docker.com/compose/compose-file/compose-file-v3/#ports).
    - For example:

        ```
         # Change this in docker-compose.yaml (not under the DB section):
         ports:
           - "192.168.1.101:5002:5000"

         # To this:   
        ports:
          - "127.0.0.1:5012:5000"

        # Use 5012 on the next step
        ```

3. point the reverse proxy servers to each respective service in teh NGINX configuration:   


    ```
    server {
       location / {
           proxy_pass http://127.0.0.1:50XX; #<-- change port to match bind port in docker-compose.yml
       }
    }
    ```

4. modify reverse proxy configuration to use certificates
   - If using let's encrypt: run the certbot tool. Link below
5. configure autorenewal of certificates
   - if using let's encrypt and certbot, a `systemd` timer has been added to the system for autorenewal. Please follow the instructions on cerbot's  instructions page

More information:
- [Nginx](https://docs.nginx.com/nginx/admin-guide/)
  - [reverse proxy](https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/)
- [let's encrypt](https://letsencrypt.org/)
- [cerbot](https://certbot.eff.org/)


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

```{admonition} KMS and Backend basic authentication consistency
:class: caution

When configuring basic authentication in KMS and Backend make sure that system user has the same username and password across both KMS and Backend and the user will setup basic authentication on their configuration file where the software will use the same credentials for KMS and backend. i.e. basic authentication credentials for a user should be consistent across servers.
```


```{admonition} Must configure modules after enabling
:class: important

After enabling basic authentication (step 1), everyone using the system must follow steps 2 and 3, else modules will exit because they can not connect to servers.
```

# Check server accessibility

To check if KMS or backend are properly configured (reverse proxy, HTTPS, basic auth, and bind interface/port) one can do a GET request to their root (`/`) endpoint. You should get an `up` response from each server. If you get "bad gateway" or "connection timeout," the module may be down or the configuration or binding locations may be miss configured. We can use `curl` to test them (or anu other API test utility). The syntax os is a follows:   
```
# without basic auth
curl <protocol>://<host>:<port>
# with basic auth
curl <protocol>://<user>:<password>@<host>:<port>
```   

for example:
```bash
curl https://johndoe:AKk2345rtfxde5@cybexp-priv.example.com:5001
```



