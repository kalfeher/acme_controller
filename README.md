# acme_controller
This role prepares a host to act as a central controller for the generation and validation of ACME (such as Let's Encrypt) based certificates.
## Staging URL is the default!!
Note that to prevent unintended consequences, the Let's Encrypt staging URL is configured. It is strongly recommended that you do not edit the default var and instead explicitly set `ac_acme_api_uri` in your playbook or host var files.
## Required variables
In addition to the target URL, it is recommended that you set the following variables:
- `ac_le_account_name` There will be one account for all domains.
- `ac_contacts` This list will be the emails contacted for notices relating to your certificates.
- `ac_domains` Is a list of domains dictionaries. The default var can be used as an example of the required data:
~~~
ac_domains:
  - { name: example.cloud, new_key: "false", org: "Example LTD", country: "au" }
~~~
- `dns_update_details` contains the details for updating DNS. The `default` dictionary will be used if an exact match can't be found for the domain being updated.

## What the role does
The role will initially create an account for use with your ACME provider, generating a new private key if none is found in `ac_priv_key_folder`.

The role will then conduct the following steps for each domain you list in `ac_domains`:
1. Generate a CSR for all your domains. Optionally, per domain, you can force a CSR to be generated even if one exists by setting `new_key="True"` within the list item of `ac_domains `.
2. If no certificate is present, then an ACME challenge is generated.
3. The challenge is added to a DNS server. The variable `dns_update_details` is a dictionary of labels for which nsupdate details are provided. If a label is not listed, the `'default'` dictionary will be used. If all your domains are hosted by the same primary server, then you only need to define the values for `'default'`.
4. The playbook will test an external DNS server at random (from the list `recursive_DNS`) to check if the new challenge is visible externally.
   a. This accounts for situations where DNS updates are made to a hidden primary server and propagated at a slow pace to the hosts which actually respond to DNS queries.
5. Once the challenge record is visible externally, the playbook will fetch the certificate.

### Known issues
The _lookup_ plugin _dig_ does not support the IPv6 addresses in the `recursive_DNS` list if using the '@' notation style. However specifying the _dnspython_ variable `name_server` will work for both IPv4 and IPv6, therefore this remains the current work around.
