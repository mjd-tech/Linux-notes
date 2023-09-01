# msmtp - cron emails

NOTE: 8/2022 Gmail forces Oauth2 now. gmail instructions probably won't work.

- install **msmtp-mta** package
- msmtp simulates the traditional UNIX "sendmail" command.
- msmtp only runs when it is actually sending mail.
- Activity is logged in /var/log/syslog. Lines contain "msmtp:"
- msmtp is an alternative to "ssmtp" which is no longer maintained.

## To local mail server

/etc/msmtprc

    account default
    host myserver
    port 2500
    auto_from off
    from fred@bedrock
    aliases /etc/aliases
    syslog LOG_MAIL

/etc/aliases

    default: localmail@myserver
    fred: localmail@myserver

### Testing

```
# send mail to root, receive mail at mailbox specified in /etc/aliases
echo -e "Subject: Hello\n\nHello, world" | sendmail root

# create a test cron job as root.
sudo crontab -e

# add this line, and a blank line below it
* * * * *  echo Test from root cron

# email should be sent at start of next minute.
```

Be sure to **deactivate** the cron job when finished testing!

## GMail

NOTE: 8/2022 Gmail forces Oauth2 now. gmail instructions probably won't work.

- If you have two-factor authentication (2FA) enabled, you will need to
  create an app password. <https://myaccount.google.com/apppasswords>
- Google will generate a strong password for you.
- If you do NOT have 2FA enabled, you will need to allow access to
  unsecured apps from the Less Secure Apps page.
  <https://myaccount.google.com/lesssecureapps>

### Create configuration file

    sudo vim /etc/msmtprc

Copy and paste the following.

- Put the relevant info where it says **changeme** or
  **change.me@gmail.com**
- If your gmail account is something like **user@domain.com**, **use
  that** instead of user@gmail.com

<!-- -->

    # A system wide configuration file is optional.
    # If it exists, it usually defines a default account.
    # This allows msmtp to be used like /usr/sbin/sendmail.
    account default

    # The place where the mail goes. The actual machine name is required
    # no MX records are consulted. Both port 465 or 587 should be acceptable
    # See also https://support.google.com/mail/answer/78799
    host smtp.gmail.com
    port 587

    # Construct envelope-from addresses of the form "user@oursite.example".
    # auto_from on
    # maildomain itix.fr
    auto_from off
    from change.me@gmail.com

    # Dispatch mails according to /etc/aliases
    aliases /etc/aliases

    # Use TLS.
    tls on
    tls_starttls on
    tls_trust_file /etc/ssl/certs/ca-certificates.crt

    # Syslog logging with facility LOG_MAIL instead of the default LOG_USER.
    syslog LOG_MAIL

    # Authenticate to GMail
    #
    # If your Gmail account is secured with two-factor authentication, you need
    # to generate a unique App Password to use in ssmtp.conf.
    # See https://support.google.com/mail/answer/185833
    # App Passwords can be generated on https://myaccount.google.com/apppasswords
    #
    # Use you Gmail username (not the App Name) in the user line and use the
    # generated 16-character password in the password line, spaces in the password
    # can be omitted.
    #
    # If you do not use two-factor authentication, you need to allow access to
    # unsecure apps. You can do so on your Less Secure Apps page.
    # See https://support.google.com/accounts/answer/6010255
    # You can do so on https://myaccount.google.com/lesssecureapps

    auth on
    user change.me@gmail.com
    password changeme

## Create the /etc/aliases file

This ensures that every mail sent to a local user (root, ftp, nobody,
etc.) is in fact sent to the designated email address.

    sudo vim /etc/aliases

Copy and paste the following, replace change.me@gmail.com with same
address used in /etc/msmtprc

    default: change.me@gmail.com

If you know for sure that all the mails will be sent by root or daemons
running as root, you can tighten up the file permissions. Otherwise, let
it as-is.

    sudo chmod 600 /etc/msmtprc

