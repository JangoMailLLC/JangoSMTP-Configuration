# Configuring JangoSMTP with Postfix

_Tested using a fresh install of CentOS 7.4 x64 at DigitalOcean with no prior configuration/setup._

## Installing Prerequisites

In order to use Postfix with JangoSMTP, you'll need to make sure that Postfix can use PLAIN authentication.  Since you'll be sending this traffic over SSL, this is **not** a security risk.

```
yum install cyrus-sasl-plain
```

## Configuring authentication

First, create a file for Postfix to authenticate with.  Be sure to replace `USERNAME` and `PASSWORD` with your JangoSMTP Username and Password!

```
echo 'express-relay.jangosmtp.net USERNAME:PASSWORD' > /etc/postfix/sasl_passwd
```

Next, you'll need to change the ownership and permission level of the file you just created:

```
chown root:root /etc/postfix/sasl_passwd
chmod 600 /etc/postfix/sasl_passwd
```

Finally, convert this file into a format that Postfix can read:

```
postmap /etc/postfix/sasl_passwd
```

## Configuring Postfix

Make the following changes to `/etc/postfix/main.cf`:

```
relayhost = express-relay.jangosmtp.net:587
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_sasl_security_options = noanonymous
smtp_sasl_auth_enable = yes
smtp_use_tls = yes
```

Once completed, run the following command to reload Postfix:

```
postfix reload
```

## Testing

Now that you've made the changes, it's time to test them!

First, create a file at `/tmp/email.txt` with the following content:

```
Subject: Terminal Email Send

Email Content line 1
Email Content line 2
```

Then, run the following command to send the test.  Be sure to replace `user@example.com` with your own email address!

```
sendmail user@example.com  < /tmp/email.txt
```