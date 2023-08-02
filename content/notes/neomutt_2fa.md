We will deal with adding an email account that requires 2FA for gmail to
neomutt. Firstly, we need to get a client id and client secret using Google
Cloud. Make sure that we register the desktop version! Make sure to then
authorize the emails that you want with this application and that you allow less
secure applications.

## Generating 2FA Tokens

Next, we need the following script: [mutt_oauth2.py](https://github.com/muttmua/mutt/blob/master/contrib/mutt_oauth2.py). We need to fill in the client id and client secret in this file with that from Google. Additionally, replace the `YOUR_GPG_IDENTITY` in the `ENCRPYTION` with the correct GPG id to use. Place this script in `~/.local/bin`. Next we can call it as 
```
mutt_oauth2.py <email>.tokens --verbose --authorize
```
I then recommend using `authcode` but you may have to try all of the various
authorizations. Once this finishes, then there should be a file
`<email>.tokens`. I recommend placing this into the `~/.local/bin` directory
(though it probably belongs in the `.cache` or `.config` directory). 

## Setting the Config Files

We are now ready to just setup the `.mbsyncrc`
and `msmtp/config` files. For the `.mbsyncrc` file, we need to add the following
lines, replacing `<email>` with the email address of course:

```
IMAPStore <email>-remote
Host imap.gmail.com
Port 993
User <email>
PassCmd "mutt_oauth2.py ~/.local/bin/<email>tokens"
AuthMechs XOAUTH2
SSLType IMAPS
CertificateFile /etc/ssl/cert.pem

MaildirStore <email>-local
Subfolders Verbatim
Path ~/.local/share/mail/<email>/
Inbox ~/.local/share/mail/<email>/INBOX

Channel <email>
Expunge Both
Far :<email>-remote:
Near :<email>-local:
Patterns * !"[Gmail]/All Mail"
Create Both
SyncState *
MaxMessages 0
ExpireUnread no
# End profile
```

We can now update the `.config/msmtp/config` file similarly as

```
account <email>
host smtp.gmail.com
port 465
from <email>
user <email>
passwordeval "mutt_oauth2.py ~/.local/bin/<email>.tokens"
auth oauthbearer
tls on
tls_trust_file	/etc/ssl/cert.pem
logfile ~/.cache/msmtp/msmtp.log
tls_starttls off
```
Make sure to then create the mail directory as

```
mkdir ~/.local/share/mail/<email>
```

We can now run `mbsync <email>`! 

## Error Performing SASL Authentication Step

If you are on MacOS, you likely will get an error of 
```
Error performing SASL authentication step: SASL(-1): generic failure: Unable to find a callback: 18948
```

This has to do with an issue of the [OS X SASL setup](https://github.com/moriyoshi/cyrus-sasl-xoauth2/issues/9#issuecomment-888149239). To circumvent this, we need to modify the default `isync` that comes with Homebrew. To do this, we can do the following:

```
brew edit isync
```

Now, replace the `HEAD` with `https://github.com/xukai92/isync.git`. Then, we
need to remove all lines starting with `url` and `sha256`. Then, we run the
following command

```
HOMEBREW_NO_INSTALL_FROM_API=1 brew reinstall isync --build
```
Unfortunately, doing this will cause `openssl$3--3.0.7` to be the default
version of `openssl` (or some lower version in general), which can cause issues with ssh and git. To get around
this, we just need to unlink and link:

```
brew unlink openssl@3
brew link openssl@3
```

Now, everything should be good!
