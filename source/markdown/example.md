
# Basic usage instructions 

In this exercise, we will configure the collector, collector client, and query modules to be able to encrypt data using the access policy and then later retrieve the encrypted data. We can encrypt with diverse encryption policies, and later we can query the backend service for encrypted data. The query module automatically decrypts decryptable data (as per CPABE access policy) into an output file.

1. Read the project description and installation procedures
2. Download the project source
3. Install docker and docker-compose (project dependencies)
4. Build the base image (takes about 30 minutes)(once per machine)
5. Build the collector and query modules. 
6. Create a virtual environment for the collector-client submodule
7. Configure each module (configuration file)
  - Add KMS credentials
  - Configure access policy(collector)
  - Configure resources URLs
8. Share some data
9. Query some data via the project

Resource links available below.

We can test out this sample deployment by changing the KMS access token in our configuration file to one of the many available preregistered accounts in the system. At any moment, feel free to pick another user below. In terms of access policy, users can encrypt using any policy that fits their encryption needs(in the collector, a KMS token is only used for retrieving public encryption key, so any token below works the same). Restrictions apply at decryption time, where CPABE cryptographically enforces(using private decryption keys) who can decrypt the data. 

## Resources
- [Project information, install procedues & manuals](https://cybexp-priv.ignaciochg.com/manual.html)
- [Project download](https://cybexp-priv.ignaciochg.com/dl/project.zip), full project download available `TODO here`   
- Sample data
  - [1](https://cybexp-priv.ignaciochg.com/dl/data1.json)
  - [2](https://cybexp-priv.ignaciochg.com/dl/data2.json)
  - [3](https://cybexp-priv.ignaciochg.com/dl/data3.json).   

Feel free to use your own JSON data (one record per line).   
Note it will be able to encrypt nested JSON, but one will not be able to correctly query it accurately because of the order of the nested data(not deterministic).

## Configuring

Each module has a configuration file; please modify this file with the information below. Configuration files are self-documented. More information is also available on the resources above.  Refer to the information below for information on this specific deployment and the instruction manual for more information on the project. 

### Decryption/access policy (collector)

Policy matching(encryption) is done according to the configuration file for the collector. One can specify an exact match or regular expression match and a rule for the name of the field. This will determine what access policy to apply to each field. Please add matching expressions to match the names of the fields. If a field does not match a rule, the collector will be encrypted said field with the default policy, also specified in the same file. All this is done on a per field, per record basis. A field is allowed to match an index rule and a CPABE rule at the same time but can not match two CPABE rules simultaneously. If multiple rules match a field, then the first rule to match will be used. The order of matching is the following: exact match, regex match, top to bottom.

Each record must have at least one field match an index rule; else, it will not be searchable and will not be sent to the backend database.    

Please use the `-a` on the collector wrapper scrip to figure out available attributes(this should provide you with the same information that I have provided below).   

To create a decryption/access policy, you must specify what attributes can decrypt the encrypted data. The following operators are allowed to create a more complex policy: `and`, `or`, `()`

Example policy: `"DEFAULT and RESEARCH" `
To decrypt, the user querying must have the attributes ` DEFAULT` and `RESEARCH` at the same time to be able to decrypt the data(attributes are part of the user's secret key).

### Server information (both)
Apply the following in the configuration

kms: `"https://cybexp-priv.ignaciochg.com:5002"`    
backend_server: `"https://cybexp-priv.ignaciochg.com:5001"`   

### Basic authentication, for unlocking access to servers
user: `johndoe`   
pass: `AKkmyqnJvR5SsRzNn8eF`   

### organization/team names (query)
In this deployment, the following KMS tokens grants access to each of the following user's secret keys with the attributes specified below. In reality, each user would keep their secrets to themselves. Pick one token when configuring the query module and query. Then pick another token and query again. Each user below has different attributes, so each should be able to decrypt different data(as per CPABE encryption policy). 

```
name: "UNRCSE"
name: "UNRRC"
name: "UNRCISO"
name: "Public"
```

#### Attributes 
```json
{
   "name": "UNRCSE",
   "attributes": ["DEFAULT",  "UNR", "CICIAFFILIATE", "RESEARCH"]
}

{
   "name": "UNRRC",
   "attributes": ["DEFAULT", "UNR", "ITOPS", "RESEARCH" ]
    
}

{
   "name": "UNRCISO",
   "attributes": ["DEFAULT", "UNR", "SECENG", "ITOPS", "RESEARCH", "CICIAFFILIATE"]
}

{  
   "name": "Public",
   "attributes": ["DEFAULT"]
}
```

# Test servers status
if you believe that the servers might be down, you can test with the following commands:
```bash
curl https://johndoe:AKkmyqnJvR5SsRzNn8eF@cybexp-priv.ignaciochg.com:5002
curl https://johndoe:AKkmyqnJvR5SsRzNn8eF@cybexp-priv.ignaciochg.com:5001
```
You should get an `up` response from each server. If you get "bad gateway" or "connection timeout," the system may be temporarily down. Please contact the system administrator.