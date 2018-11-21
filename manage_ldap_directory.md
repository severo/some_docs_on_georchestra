# Manage LDAP directory

The easiest tool to manage the geOrchestra LDAP directory is [Apache Directory Studio](https://directory.apache.org/studio/). See the most apropriate way to install it on your distribution.

## Connect to the docker volume

To connect to and manage the docker LDAP directory, first it needs to be exposed to the host network. To do so, in the docker directory, edit the `docker-compose.yml` file to add the following lines:

```yml
  ldap:
    (...)
    ports:
      - "389:389"
```

and restart the docker ldap volume:

```bash
docker-compose restart ldap
```

Verify that the port is exposed, by:

```bash
sudo ss -platn | grep -e 389
```

Then open Apache Directory Studio:

- in the "Connections" tab, add a new connection: "Hostname": `localhost`, "Port": `389`, no encryption, "Next"
- "Simple authentication", "Bind DN": `cn=admin,dc=georchestra,dc=org`, "Bind password": `secret` (see [the geOrchestra documentation on LDAP](https://github.com/georchestra/georchestra/blob/18.06/docs/setup/openldap.md))
- "Finish"


