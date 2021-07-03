# Submision

Submission of data starts at each organization. Each organization determines the importance of the data/data fields they are sharing. With that information, they create an encryption policy. The organizations write the encryption policy in terms of the fields and their respective encryption policy, i.e., fields and attributes that can decrypt the said fields. The decryption of information is enforced using CPABE encryption/decryption scheme.

For example the following comfiguration snippet:   

```yaml
policy:
  default:
    abe_pol: "default_public"
  index:
    exact:
      - match: "ipv4"
      - match: "ipv6"
    regex:
      - match: .*src.*

exact:
  - match: "event_id"
    abe_pol: "itops AND ciso"
  - match: "count"
    abe_pol: "default_public AND research"
regex:
  - match: ".*ssn.*"
    remove: True
  - match: ".*timestamp.*"
    abe_pol: "itops"
```

It will create an index for a record over fields that match:
- `ipv4` and `ipv6` exactly 
- the regular expression `.*src.*`
- if neither of these rules matches at least one field on a  record, then that record is rejected as it will be unable to search over in the database if submitted without one
Then it will search for exact matches and do:
- encrypt the field `event_id` with the policy `itops AND ciso`
- encrypt the field `count` with the policy `default_public AND research`
- will remove the fields that match the regular expression `.*ssn.*` from being submitted
- encrypt the fields that match the regular expression `.*timestamp.*` with the policy `itops`

Any remaining fields for a particular record that have not matched a rule (excluding index rules) will be encrypted with the default policy, `default_public` in this case. A user is free to specify this edge condition. It is recomended to follow the system convention, so that data is share with everyone on this edge condition.

For example, using the above encryption policy and the following input record:   

```json
{
   "ipv4": "192.168.1.1",
   "user_ssn": "123-45-6789",
   "event_id": 9824398,
   "count": 92747634982,
   "event_timestamp": 1578246810,
   "error": "invalid ID",
   "color": "blue"
}
```
will yield the following encrypted record:   

```
{
   "index": [ DE("192.168.1.1", DE_PK) ],
   "cpabe_event_id": CPABE(9824398,CPABE_PK,"itops AND ciso"),
   "cpabe_count": CPABE(92747634982,CPABE_PK,"itops AND research"),
   "cpabe_event_timestamp": CPABE(1578246810,CPABE_PK,"itops"),
   "cpabe_error": CPABE( "invalid ID",CPABE_PK,"itops"),
   "cpabe_color": CPABE( "blue",CPABE_PK,"default_public")
}
```
`DE()` and `CPABE()` are used to show what actions will be applied to the data as part of this example. In reality, these will evaluate to encrypted values.

It is essential to note that any attribute can be specified in the CPABE encryption policies. If no organization holds that attribute, then no one will be able to decrypt that data. If a new organization signs up and receives that previously unused attribute, they will now decrypt that data. To prevent encrypting data with unused attributes (if that is something of interest), check the KMS for available attributes. To check available attributes, use the `-a` flag with the `collector.sh` script. For usage information use `collector.sh --help`.

# Query

The encryption flow is much simpler than the encryption flow. To query, one can use the `query.sh`. In a nutshell, this software encrypts the query `DE(QUERY, DE_PK)` and sends that over to the backend API. The backend then searches over the indexed fields for any match and returns the whole encrypted record. Then the software tries to decrypt the data using the user's secret CPABE key. Each key is assigned certain attributes at creation time by the KMS. Only the fields that were encrypted using a policy that correctly matches the user's secret key will be able to be decrypted. Then, only the decrypted records are returned to the user.