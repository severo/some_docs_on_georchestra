# Test mails

Some geOrchestra modules send mails: console (password change, user creation), extractorapp (job finished). To test them i na simple manner, the docker installation provides local smtp and imap servers (see the [docker instructions](https://github.com/georchestra/docker/blob/master/README.md) and the [local mail server documentation](https://github.com/camptocamp/docker_smtp/blob/master/README.md)).

## Webmail

The docker image provides a [webmail](https://georchestra.mydomain.org/webmail/). But I couldn't make it work, because it asks for a "Server" field, and I could not find the right value for it.

## Local mail client

Another way to see the mails sent by the local geOrchestra instance is to configure a local mail client, say Thunderbird. It requires to first expose a port to the docker imap service:

- edit the `docker-compose.override.yml` file:

    ```yml
      courier-imap:
        (...)
        ports:
          - "143:143"
    ```
- reload the imap service with

    ```bash
    docker-compose up -d courier-imap
    ```

- check that the port is exposed:

    ```bash
    sudo ss -platn | grep -e 143
    ```

Then configure a new account on the local mail client, with the following parameters:

- name: "local geOrchestra mails"
- imap server: "localhost"
- imap port: "143"
- security: "none"
- identification: "password"
- user: "smtp"
- password: "smtp"

With this configuration, all the mails sent by geOrchestra may be read from the local client. Note that it does not allow to send mails (this would require to expose the SMTP port, but it's not useful to send mails for development purposes).
