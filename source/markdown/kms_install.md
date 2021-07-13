# KMS Administration account

The KMS is managed by administratior accounts. KMS administrator account(s) will allow the creation of users.

One can create an administrator account in the KMS container with the following steps:
1. cd into <kms-project-root>
2. run `./add_admin.sh`
3. enter administrator username to create
4. enter password
  - Administrator can also login and retrieve its token via KSM API in the even that it is misplaces after creation
5. If succesfull you will be presented with account username and token, and level of permissions (in this case admin)



```{admonition} Note
:class: note

When quering the API enpoints for keys, the administrator account will only be able to retrieve public keys as no private key is generated for the user.
```

```{admonition} Warning
:class: warning

Physical access to the `secrets` folder will provide full access to all the keys. keep this folder secret. If possible mount an encrypted filesystem prior to running the KMS as this will provide protection of keys at rest.
```

```{admonition} Warning
:class: warning

Do not use the an administrator API token for anything other than user creation.
```

# KMS JWT Secret

This random value is used for Bearer token creation. This value is to be kept secret as it can be used to generate access to the system. Please generate this randomly when configuring KMS for the first time using proven cryptographically secure random number generator. One can do so as follows:   

```bash
python -c 'import os; print(os.urandom(16).hex())'
```
place output of above command in `FLASK_JWT_SECRET_KEY` field in the KMS configuration.

```{admonition} KMS Secret Key
:class: note

Changing the value of `FLASK_JWT_SECRET_KEY` in the configuration effectively revokes all active tokens. To get new tokens use the login endpoint in the KMS.
```


# KMS User Creation 

User creation must be done via the KMS API (Please refer to KMS API section for endpoint details). This will also automatically create user keys with their respective attributes. 

1. Refer to API reference
2. Use administrator API key in `X-Authorization` header
3. Call user creation endpoint 
4. Hand API keys to User, else discard as they can login and get API keys


```{admonition} Server authentication and authorization
:class: caution

The backend API and KMS API use two types of authentication. That is, Basic Athentication and Bearer Token Authentication. Basic Authentication is used to prevent anauthenticated access to any resource on the systems. Bearer token is used for authentication and authorization for key distribution. Basic authentication uses header `Authorization` and Bearer token uses `X-Authorization` header. 
```

```{admonition} Registered Username
:class: caution

Usernames registered with KMS may differ from provided username, please check response. This is because the KMS normalizes username before using them for compatibility purposes. The registered username is the one provided in the response to the registration API call.
```

# KMS System keys

KMS automatically checks for keys in the mounted `secrets` folder, if not found it will generated them.

