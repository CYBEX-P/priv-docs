
<!--
# Fetching images TODO

When fetching the images, the base image is unnecessary as the images are bundled with the base image.

## Option 1 Export import save
[tutorial](https://stackoverflow.com/a/23938978/12044480)   
[docker export](https://docs.docker.com/engine/reference/commandline/export/)   

## Option 2 Docker registry
[docker registry](https://docs.docker.com/registry/deploying/)   
[How to use your own Registry tutorial](https://www.docker.com/blog/how-to-use-your-own-registry/) -->


Before running a module, make sure that the base and the module images are built.

```{admonition} Python and pip version
:class: note

This project is built using Python3, therefore any python command specified in this guide refers to python3. If in doubt whether the `python` command is executing the right version it is better to explicitelly specify the version the following way `python3` inteast of using `python`. Same applies with the python package magaer pip, use `pip3` as teh command.
```

# Configuration file
Before you can use each module, you must create a configuration file. A sample configuration file is provided with each module. Each sample configuration file has comments describing what fields do. Here we will talk about the more important fields.

After creating and configuring the configuration file, one is ready to run the module.
When executing the wrapper scripts, pass the configuration file as an argument. For more help, see each wrapper script's help output for complete and more accurate usage information. 

## Server URLS
The format is a follows:
```
procotol://host:port
```

```{admonition} Note
:class: warning

There is no trailing `/`
```

The protocol can be either `http` or `https`. The server administrator will let you know. 
By default, the protocol will be `http`; one can configure `https` by adding a reverse proxy like Nginx in front of each server. More information found [here](TODO).



## Setting up the encryption policy
<!-- timestamp match
default match 
exeact match
regex match -->

Policy matching is done according to the Collector's configuration file.  One Specifies rules to match fields in each record. Each rule has an associated CPABE policy that will be applied to the field if matching. If no rule matches, the default policy is applied. One can use an exact match or regex match for a field. Encryption is done field-wise. Aside from encryption policy, the policy administrator must also specify what field will be indexed for encrypted searching. This can be done in a similar fashion as encryption rules.
    
Please use the `-a` on the collector wrapper scrip to figure out available attributes. Then one can use `and`, `or`, `parenthesis` to create the policy.
Example of policy: `"DEFAULT and RESEARCH"`. With this example policy, the person querying must be `"DEFAULT and RESEARCH"`to decrypt that field.

You should provide the configuration file in the YAML language. Please use the provided template to avoid any misconfiguration. Here is a link to a [YAML tutorial](https://docs.ansible.com/ansible/latest/reference_appendices/YAMLSyntax.html).



```{admonition} Warning
:class: warning

A syntactically correct configuration with wrong semantics can yield incorrect policy parsing, resulting in incorrectly encrypted data. We advised to run on Collector with the `debug`, `process_only_one`, and `show_enc_policy` flags with test data to confirm correct policy matching, after which you can disable.
```


## More on Encryption policy

To avoid confusion, here is a layout with data types for the policy portion of the configuration (not in official YAML format):
TODO FIX INDENTING!!
```
policy:
   default:
      # if no other policy matches below, this will be used
      # if an index rules matches but no exact or regex rules match
      #  then this will be used as CPABE policy
      abe_pol: String
   
   index:
   # this will exact match fields to create searchable encrypted fields
      # this will match exact strings with fields
      exact: # list of "match: string"
         - match: String
         - match: String
   
   # this will match fields with regex to create searchable encrypted fields
   # this is a list of "match: string", where the string is the regex
      regex: 
         - match: string
         - match: string

   # these rules are for exact matching of each field
   exact:   # list of exact matching strings and their CPABE policy (or remove flag)
      - match: string
        abe_pol: string
      - match: string
        abe_pol: string

   # these rules are for regex matching of each field
   regex: # list of regex strings and their CPABE policy (or remove flag)
      - match: string
        abe_pol: string
      - match: string
        remove: bool # set to True else you will need to specify "abe_pol"

```

<!-- ##### Collector -->

# Run
Execute the respective wrapper script at the root of the module. Each wrapper script supports the `-h` and the `--help` flag, which will output their respective help/usage information.

for example, the query help information looks as follows:   
```
$ ./query.sh -h
Create and run a docker container to query the encrypted backend. By default the container is left behind,
vacuum recommended. Build at least once or when code has changed.

Usage:
  ./query.sh [Options] config-file query output-file
  ./query.sh --build --build-only

Positional arguments:
  config-file           configuration file yaml
  query                 string representing any query value
  output-file           file where the unencrypted information is placed

Options arguments:
  -b, --build           build docker image
  --build-only          exit after building
  -s, --shell           run shell, ignores most flags
  -f, --from-time EPOCH integer epoch used as > filter for the query
  -t, --to-time EPOCH   integer epoch used as < filter for the query
  -l, --left-inclusive  modify the --from-time to be inclusive >=
  -r, --right-inclusive modify the --to-time to be inclusive <=
  -c, --vacuum          remove container upon exit. If more than one container
                        of this type exists, it will remove all
  -h, --help            print this help

```

To run the collector:
```bash
./collector.sh config.yaml
```

when querying one can run as follows:
```bash
./query.sh config.yaml '123.123.123.123' output_file
# view output 
cat output_file
```
It is recommended to run Collector, KMS, and backend in a detached manner, for example, using Tmux.

The KMS and backend can be run as follows:
```bash
./kms.sh config.yaml
```
```bash
./backend.sh config.yaml
```
The Collector and Query use a simple Docker run procedure (not in daemon mode), so `ctrl+c` or a signal can terminate the process, just like any other. This makes it easy to incorporate into scripts and systemd timers and services. 

```{admonition} Permission denied
:class: note

If bash scripts are not running because of permissions, please make them executable with `sudo chmod +x <script-name.sh>`. Alternatively run them using `bash` or `sh`, like so `sh <script-name.sh>`.
```

```{admonition} Key Files Problems
:class: caution

If for some reason the module is stopped (either by a signal or error) while key files are bing fetched, it will produce bad key files. Next time the code runs it will try to read from said key files before trying to get them from the KMS server. This condition results in corrupted key files, which will lead to either the program crashing or to data that is encrypted incorrectly. To fix this dispose of the keys from `secrets` folder and run again (will fetch from KMS server). To prevent this, do not early terminate module.
```

# Running KMS and Backend as daemons

The KMS and Backend modules use docker-compose. Docker-compose handles automatic start on boot (as specified in the docker-compose.yaml file) as daemons.  To achieve this, we can run each module (KMS, backend), wait for the script to deploy the containers and initialize them correctly and then reboot the system.

Alternative if we are not going to reboot the system for the time being,  after issuing  `./kms.sh config.yaml` or `./backend.sh config.yaml` one can simply `CTRL+c` (after each script is done initializing the containers) to temporarily stop the services and free the terminal. Since the Docker service runs at boot, our services will automatically start at boot next time that happens. Since we stopped out services and are not rebooting for the time being, we can start them back up manually and in the background with `sudo docker-compose start`  (this has to be executed from within each module's root directory).

## Retrieving logs of KMS and backend modules
if they are running in the foreground, it is as easy as looking in the terminal. 

If they are running in the background, one can use the `sudo docker logs <container-id>` command to see its logs. One can find the container id of a particular container/module with `sudo docker ps --all` command. 

# Checking the status of the containers
To check the status of a module, one can issue the following command in the terminal:
```bash
sudo docker ps --all
```
This command will show all containers in the system, including ones that are currently not running. This command is helpful to determine if a container/module is running or not and if it has suddenly stopped/exited (and how long that happened).


# More on deploying Collector-client & Collector

The Collector is a multi-part module. The Collector itself is the encryption module; the gatherers ship the information to the Collector encryption module to be encrypted and sent over to the backend.  To run the Collector correctly, one has to set up the encryption policy. 

One has to start the Collector using `collector.sh`, then one can run multiple collector clients. 
Another good reason for separating the encryption portion from the data collection process is that it can allow multiple sources to be encrypted under the same policy.  Collectors-clients are meant to be easy to code to fit the user/organization's liking. The only requirement that client collectors have is that they must connect to the Collector via a TCP socket and send one JSON record per connection. We have provided a sample Collector client inside the Collector's root directory. This client grabs JSON records from a file and sends them to the Collector/encryption module.

The dependencies for this client are are `python3`, `python3-venv`, and python modules specified in `requirements.txt` (install as specified below).

Install collector dependencies(for Debian based, e.g., Ubuntu):
```bash
sudo apt install -y python3 python3-venv
```
To run the collector client:
```bash
cd collector-client

# make python environment (do once)
python3 -m venv env 

# then source environment, do when wanting to run client
#  or to install dependencies
source env/bin/activate
# note (env) at the left-hand side of the terminal prompt

# install dependencies, do once
pip install -r requirements.txt 

# then run client
python3 collector-client.py --host <collector-host> --port <collector-port> <json-file>
# when running a collector on the same machine this script will default by using default ports and hostname
python3 collector-client.py <json-file>

```
Note: `collector-client.py` has a flag `--json-key`, it make it so it only sends that spesific key-data pair. For example:
```json
{"_id": {"$oid": "5d659c2d34ecea6c769ec9ff"}, "itype": "raw", "data": {"session": "17e8b9e02971", "geoip": {"ip": "165.227.91.185", "latitude": 40.7214, "longitude": -74.0052, "location": {"lat": 40.7214, "lon": -74.0052}, "continent_code": "NA", "timezone": "America/New_York", "country_code3": "US", "country_code2": "US", "postal_code": "10013", "country_name": "United States", "city_name": "New York", "dma_code": 501, "region_name": "New York", "region_code": "NY"}, "source": "/home/cowrie/cowrie/var/log/cowrie/cowrie.json", "@timestamp": "2019-08-27T20:51:18.993Z", "tags": ["beats_input_codec_plain_applied", "geoip", "beats_input_codec_json_applied"], "timestamp": "2019-08-27T20:51:16.595926Z", "host": {"name": "ssh-peavine"}, "duration": 120.02226901054382, "eventid": "cowrie.session.closed", "msg": "Connection lost after 120 seconds", "beat": {"hostname": "ssh-peavine", "version": "6.5.4", "name": "ssh-peavine"}, "offset": 3165054, "@version": "1", "src_ip": "165.227.91.185", "prospector": {"type": "log"}, "sensor": "ssh-peavine"}, "orgid": "identity--f27df111-ca31-4700-99d4-2635b6c37851", "timezone": "US/Pacific", "sub_type": "x-unr-honeypot", "_hash": "ce1125ef568028d65ad444dd9a6dacf6a860b7a3d0a43ebabc0b03fb258c80db", "uuid": "raw--8d30f29e-9285-4790-ac57-bbba5aa56fbb", "filters": ["filter--ad8c8d0c-0b25-4100-855e-06350a59750c"]}
```
The data that we are after is inside the `data` key. Therefore we would use the tool as follows:
```bash
python3 collector-client.py --json-key data <json-file>
```
this will result in only sending the following:
```json
{"session": "17e8b9e02971", "geoip": {"ip": "165.227.91.185", "latitude": 40.7214, "longitude": -74.0052, "location": {"lat": 40.7214, "lon": -74.0052}, "continent_code": "NA", "timezone": "America/New_York", "country_code3": "US", "country_code2": "US", "postal_code": "10013", "country_name": "United States", "city_name": "New York", "dma_code": 501, "region_name": "New York", "region_code": "NY"}, "source": "/home/cowrie/cowrie/var/log/cowrie/cowrie.json", "@timestamp": "2019-08-27T20:51:18.993Z", "tags": ["beats_input_codec_plain_applied", "geoip", "beats_input_codec_json_applied"], "timestamp": "2019-08-27T20:51:16.595926Z", "host": {"name": "ssh-peavine"}, "duration": 120.02226901054382, "eventid": "cowrie.session.closed", "msg": "Connection lost after 120 seconds", "beat": {"hostname": "ssh-peavine", "version": "6.5.4", "name": "ssh-peavine"}, "offset": 3165054, "@version": "1", "src_ip": "165.227.91.185", "prospector": {"type": "log"}, "sensor": "ssh-peavine"}

```
```{admonition} Collector
:class: warning

The Collector index creation works well on the first level of the JSON structure(make sure that indices are created only on the first level). It can encrypt on more levels on the tree, but index retrieval/querying of the data might not be deterministic(due to sorting of the JSON structure) if the index was multilevel; therefore, it may result in unsearchable data.
```

# Force key retrieval

If for some reason keys are not working it may be necessary for keys to be fetched again from the KMS server (this applies to `collector` and `query`), one can:   

1. Stop the module
2. Delete the contents from `secrets` folder for that module
3. Restart the module

```{admonition} Keys folder
:class: caution

Do not delete the actual folder as the folder structure is important.
```


```{admonition} KMS keys
:class: caution

Do not delete keys from KMS `secret` folder. If system keys are deleted and there happens to be encrypted data in the backend and new data is submitted with different set of keys (KMS regenerates new keys upon not finding them) it will create incompatible data.
```

# Collector & collector-client status indicators

In the least verbose mode `collector` and `collector-client` print letters indicating their current state. It is important to note that both of these programs are mutithreaded so output will not be in any guaranteed order. The indicators are a follows:

collector:   

| Indicator | Description                                                             |
|:---------:|-------------------------------------------------------------------------|
|     N     | A worked thread established a new connection with a collector-client    |
|     R     | A worked thread read data from client successfully                      |
|     E     | A worked thread encrypted the data successfully                         |
|     P     | A worker thread posted encrypted data to backend successfully           |
|     F     | A worker thread failed to posted encrypted data to backend, will retry  |
|     L     | Worker thread gave up on posting encrypted data, too many fail attempts |

collector-client:   

| Indicator | Description                                                       |
|:---------:|-------------------------------------------------------------------|
|     i     | A new piece of data has been added to the work queue              |
|     F     | All the data has been added to the work queue                     |
|     .     | A piece of data was successfully sent to the collector            |
|     D     | A worked thread has exited (usually after finishing all its work) |


# Date format

Date format can be any of the major date formats. This works when submitting data for encryption and when quering data using date ranges, not restricted to just UNIX epoch. This includes things like timezones as well. For more information on supported date format visit the [dateutil parser documentation](https://dateutil.readthedocs.io/en/stable/parser.html).

# Troubleshooting
- If an error regarding `unix:///var/run/docker.sock` or permission denied, please make sure docker daemon/service is running. 
- If a module can not connect to another, make sure that the following configurations are correct:
  - the bind interface of the server
  - the address of the server in the client configuration
  - for example, if the server is bound to its IP, it can not be accessed via localhost, even if they are the same machine. It is recommended to bind the servers to a local interface like "`127.0.0.1`:`<port>` so that they are only accessible locally and the use of proxypass using a reverse proxy (line Nginx) bound to the public interface, this would allow you to enable HTTPS. Please see [here](TODO) for more information on proxy setup.
    - the address format (for clients) is as follows:
    - protocol://hostname:port
    - no slash at the end






<!--

# Important
Regardless of whether you are building or fetching the images, a few things are required to run each image. Here is a table:

|Module     |docker            | Building | Configuration File| DB|
|-----------|:----------------:|:--------:|:-----------------:|:-:|
|priv-libs  |:heavy_check_mark:|-         |      :x:          |-  |
|collector  |:heavy_check_mark:|priv-lib  |:heavy_check_mark: |:x:|
|query      |:heavy_check_mark:|priv-lib  |:heavy_check_mark: |:x:|
|KMS        |:heavy_check_mark:|priv-lib  |:heavy_check_mark: |:heavy_check_mark:|
|backend API|:heavy_check_mark:|priv-lib  |:heavy_check_mark: |:heavy_check_mark:|

 :heavy_check_mark:
:x: -->


