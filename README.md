# docker-mailserver

[![Build Status](https://travis-ci.org/tomav/docker-mailserver.svg?branch=master)](https://travis-ci.org/tomav/docker-mailserver)

A fullstack but simple mail server (smtp, imap, antispam, antivirus...).  
Only configuration files, no SQL database. Keep it simple and versioned.  
Easy to deploy and upgrade.  

Includes:

- postfix with smtp auth
- courier-imap with ssl support
- amavis
- spamassasin supporting custom rules
- clamav with automatic updates
- opendkim
- opendmarc 
- fail2ban
- [LetsEncrypt](https://letsencrypt.org/) and self-signed certificates
- optional pop3 server
- [integration tests](https://travis-ci.org/tomav/docker-mailserver) 
- [automated builds on docker hub](https://hub.docker.com/r/tvial/docker-mailserver/)

Why I created this image: [Simple mail server with Docker](http://tvi.al/simple-mail-server-with-docker/)

Before you open an issue, please have a look this `README`, the [FAQ](https://github.com/tomav/docker-mailserver/wiki/FAQ) and Postfix documentation. 

## Usage

    # get latest image
  	docker pull tvial/docker-mailserver

    # create a "docker-compose.yml" file containing:  
    mail:
      image: tvial/docker-mailserver
      hostname: mail
      domainname: domain.com
      # your FQDN will be 'mail.domain.com'
      ports:
      - "25:25"
      - "143:143"
      - "587:587"
      - "993:993"
      volumes:
      - ./spamassassin:/tmp/spamassassin/
      - ./postfix:/tmp/postfix/

    # start he container
  	docker-compose up -d mail

## Managing users and aliases

### Users

Users are managed in `postfix/accounts.cf` with the helper script `generate-user-databases`. 
Just add the full email address and its password separated by a pipe.  

Example:

    user1@domain.tld|mypassword
    user2@otherdomain.tld|myotherpassword

Then the user databases for courier and cyrus sasl with encrypted passwords must be generated with the following:

    docker run -ti --rm -v "$(pwd)"/postfix:/tmp/postfix -h mail.domain.com -t tvial/docker-mailserver generate-user-databases

The needed DBs will then be found inside `postfix/accounts-db/` folder.

For **security reason** clear text passwords are no longer allowed on running instances of the image. For that reason the file `postfix/accounts.cf` MUST be removed before starting up the container (it will not start up if that file is still there).

### Aliases

Please first read [Postfix documentation on virtual aliases](http://www.postfix.org/VIRTUAL_README.html#virtual_alias).

Aliases are managed in `postfix/virtual`.  
An alias is a full email address that will be:
* delivered to an existing account in `postfix/accounts.cf`
* redirected to one or more other email adresses

Alias and target are space separated. 

Example:

    # Alias to existing account
    alias1@domain.tld user1@domain.tld

    # Forward to external email address
    alias2@domain.tld external@gmail.com

## Environment variables

* DMS_SSL
  * *empty* (default) => SSL disabled
  * letsencrypt => Enables Let's Encrypt certificates
  * self-signed => Enables self-signed certificates
* ENABLE_POP3
  * *empty* (default) => POP3 service disabled
  * 1 => Enables POP3 service
* SMTP_ONLY
  * *empty* (default) => courier daemons might start
  * *1 => do not launch any courier daemons (imap, pop3)
* SA_TAG
  * *2.0* (default) => add spam info headers if at, or above that level
* SA_TAG2
  * *6.31* (default) => add 'spam detected' headers at that level
* SA_KILL
  * *6.31* (default) => triggers spam evasive actions
* SASL_PASSWORD
  * *empty* (default) => No sasl_passwd will be created
  * *string* => A /etc/postfix/sasl_passwd will be created with that content and postmap will be run on it
* ENABLE_FAIL2BAN
  * *empty* (default) => fail2ban service disabled
  * 1 => Enables fail2ban service

Please read [how the container starts](https://github.com/tomav/docker-mailserver/blob/master/start-mailserver.sh) to understand what's expected.  

## SSL

Please read [the SSL page in the wiki](https://github.com/tomav/docker-mailserver/wiki/SSL) for more information.

## Todo

Things to do or to improve are stored on [Github](https://github.com/tomav/docker-mailserver/issues), some open by myself.
Feel free to improve this docker image.

## Contribute

- Fork
- Improve
- Add integration tests in `test/tests.bats`
- Build image and run tests using `make`  
- Document your improvements
- Commit, push and make a pull-request
