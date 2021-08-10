
# Instructions

In this exercise, we will configure the collector, collector client, and query modules to be able to encrypt data using the access policy and then later retrieve the encrypted data. We can encrypt with diverse encryption policies, and later we can query the backend service for encrypted data. The query module automatically decrypts decryptable data (as per CPABE access policy) into an output file. In this demo we use the priv-libs image, collector, collector-client and the query modules.

1. Read the project description and installation procedures
2. Download the project source
3. Install docker and docker-compose (project dependencies)
4. Build the base image (takes about 30 minutes)(once per machine)(could optionally be exported to other machines)
5. Build the collector and query modules. 
6. Create a virtual environment for the collector-client submodule
7. Configure each module (configuration file)
  - Add KMS credentials
  - Configure access policy(collector)
  - Configure resources URLs
8. Share some data
9. Query some data via the project


We can test out this sample deployment by changing the KMS access token in our configuration file to one of the many available preregistered accounts in the system. At any moment, feel free to pick another user below. In terms of access policy, users can encrypt using any policy that fits their encryption needs(in the collector, a KMS token is only used for retrieving public encryption key, so any token below works the same). Restrictions apply at decryption time, where CPABE cryptographically enforces(using private decryption keys) who can decrypt the data. 


# Get demo keys and basic authentication credentials

Get demos key for the above accounts [here](https://cybex.cse.unr.edu/keys), login with cybex credentials. Each account has access to certain attributes. Basic authentication grants access to the the system as a whole. Please configure configuration files according to the manual.


# Sample data
  - [1](https://cybex.cse.unr.edu/docs/encrypted-analytics/_static/data1.json)
  - [2](https://cybex.cse.unr.edu/docs/encrypted-analytics/_static/data2.json)
  - [3](https://cybex.cse.unr.edu/docs/encrypted-analytics/_static/data3.json) 

Feel free to use your own JSON data, just remember, one JSON record per line. Nested json is not fully supported.

# Configuring

Each module has a configuration file; please modify this file with the information below. Configuration files are self-documented. More information is also available on the documentation.  Refer to the information below for information on this specific deployment and the instruction manual for more information on the project. 

# Decryption/access policy (collector)

Policy matching(encryption) is done according to the configuration file for the collector. One can specify an exact match or regular expression match and a rule for the name of the field. This will determine what access policy to apply to each field. Please add matching expressions to match the names of the fields. If a field does not match a rule, the collector will be encrypted said field with the default policy, also specified in the same file. All this is done on a per field, per record basis. A field is allowed to match an index rule and a CPABE rule at the same time but can not match two CPABE rules simultaneously. If multiple rules match a field, then the first rule to match will be used. The order of matching is the following: exact match, regex match, top to bottom.

Each record must have at least one field match an index rule; else, it will not be searchable and will not be sent to the backend database.    

Please use the `-a` on the collector wrapper scrip to figure out available attributes(this should provide you with the same information that I have provided below).   

To create a decryption/access policy, you must specify what attributes can decrypt the encrypted data. The following operators are allowed to create a more complex policy: `and`, `or`, `()`

Example policy: `"DEFAULT and RESEARCH" `
To decrypt, the user querying must have the attributes ` DEFAULT` and `RESEARCH` at the same time to be able to decrypt the data(attributes are part of the user's secret key).

# Server information (both)
Apply the following in the configuration

kms: `"https://cybex.cse.unr.edu:5002"`    
backend_server: `"https://cybex.cse.unr.edu:5001"`

note: no trailing slash (/).


# Sample accounts
In this deployment, the following accounts grant access to each of their respective attributes. In reality, each user would keep their secrets to themselves. Pick one token when configuring the query module and query. Then pick another token and query again. Each user below has different attributes, so each should be able to decrypt different data(as per CPABE encryption policy). 


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
To double check that the server are up, use the following commands (substitute credentials):
```bash
curl https://USERNAME:PASSWORD@cybex.cse.unr.edu:5002
curl https://USERNAME:PASSWORD@cybex.cse.unr.edu:5001
```
You should get an `up` response from each server. If you get "bad gateway" or "connection timeout," the system may be temporarily down. Please contact the system administrator.