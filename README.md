# acme_controller
This role prepares a host to act as a central controller for the generation and validation of ACME (such as Let's Encrypt) based certificates.
## Staging URL is the default!!
Note that to prevent unintended consequences, the Let's Encrypt staging URL is configured. It is strongly recommended that you do not edit the default var and instead explicitly set `ac_acme_api_uri` in your playbook or host var files.
## Required variables
In addition to the target URL, it is recommended that you set the following variables:
- `ac_le_account_name` There will be one account for all domains.
- `ac_contacts` This list will be the emails contacted for notices relating to your certificates.

## Other playbooks
The `acme_run.yml` and `acme_dns-01*.yml` playbooks are provided to allow you to use your controller to use the dns-01 challenge to valid your certificates.
The playbook conducts the following steps for each domain you list in `ac_domains`:
1. Generate a CSR for all your domains. Optionally, per domain, you can force a CSR to be generated even if one exists by setting `new_key="True"` within the list item of `ac_domains `.
2. If no certificate is present, then an ACME challenge is generated.
3. The challenge is added to a DNS server. The variables: `ac_dnsupdate_host`, `ac_dns_tsig_name`, `ac_dns_tsig_key` and `ac_dns_tsig_algo` need to be set in order for the playbook to successfully submit an nsupdate.
4. The playbook will test an external DNS server at random (from the list `recursive_DNS`) to check if the new challenge is visible externally.
   a. This accounts for situations where DNS updates are made to a hidden primary server and propogated at a slow pace to the hosts which actually respond to DNS queries.
5. Once the challenge record is visible externally, the playbook will fetch the certificate.
