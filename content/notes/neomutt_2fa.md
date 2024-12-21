---
title: "Two Factor Authentication for GMail and (Neo)Mutt"
date: 2023-08-03
draft: false
author: "Matthew R. Hennefarth"
---
We will deal with adding an email account that requires 2FA for gmail to
neomutt. Firstly, we need to get a client id and client secret using Google
Cloud. Make sure that we register the desktop version! Make sure to then
authorize the emails that you want with this application and that you allow less
secure applications for each of the email accounts.

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

# Neomutt Accounts

For Gmail accounts, we can bind all of the folders by using the following command:
```sh 
dir=~/.local/share/mail/<email>; tree $dir/ -l -d -I "cur|new|tmp|certs|.notmuch|INBOX|\[Gmail\]" -afin --noreport | tail -n +2 | while read i; do echo \"=${i#$dir/}\"; done | tr '\n' ' '
```

The other things to include in the `<email>.muttrc` file is:

```
# vim: filetype=neomuttrc
# muttrc file for account <email> 
set real_name = "Matthew Hennefarth"
set from = "<email>"
set sendmail = "msmtp -a <email>"
alias me matthew.hennefarth <<email>>
set folder = "/Users/mhennefarth/.local/share/mail/<email>"
set header_cache = "/Users/mhennefarth/.cache/mutt-wizard/<email>_gmail.com/headers"
set message_cachedir = "/Users/mhennefarth/.cache/mutt-wizard/<email>_gmail.com/bodies"
set mbox_type = Maildir
set hostname = "gmail.com"
source /Users/mhennefarth/.local/share/mutt-wizard/switch.muttrc
set spool_file = "+INBOX"
set postponed = "+[Gmail]/Drafts"
set trash = "+[Gmail]/Trash"
unset record

macro index o "<shell-escape>mailsync <email><enter>" "sync <email>"

named-mailboxes "Gmail" $spool_file
mailboxes "=[Gmail]/Sent Mail" "=[Gmail]/Spam"
mailboxes $postponed $trash

# This is the magic right here...
mailboxes `dir=~/.local/share/mail/<email>; tree $dir/ -l -d -I "cur|new|tmp|certs|.notmuch|INBOX|\[Gmail\]" -afin --noreport | tail -n +2 | while read i; do echo \"=${i#$dir/}\"; done | tr '\n' ' '`

set pgp_default_key=<pgp-key>

macro index,pager gd "<change-folder>=[Gmail]/Drafts<enter>" "go to drafts"
macro index,pager Md ";<save-message>=[Gmail]/Drafts<enter>" "move mail to drafts"
macro index,pager Cd ";<copy-message>=[Gmail]/Drafts<enter>" "copy mail to drafts"
macro index,pager gj "<change-folder>=[Gmail]/Spam<enter>" "go to junk"
macro index,pager Mj ";<save-message>=[Gmail]/Spam<enter>" "move mail to junk"
macro index,pager Cj ";<copy-message>=[Gmail]/Spam<enter>" "copy mail to junk"
macro index,pager gt "<change-folder>=[Gmail]/Trash<enter>" "go to trash"
macro index,pager Mt ";<save-message>=[Gmail]/Trash<enter>" "move mail to trash"
macro index,pager Ct ";<copy-message>=[Gmail]/Trash<enter>" "copy mail to trash"
macro index,pager gs "<change-folder>=[Gmail]/Sent Mail<enter>" "go to sent"
macro index,pager Ms ";<save-message>=[Gmail]/Sent Mail<enter>" "move mail to sent"
macro index,pager Cs ";<copy-message>=[Gmail]/Sent Mail<enter>" "copy mail to sent"
```

Again, replace `<email>` with the email. Note that I also use `muttwizard` so
some of the configurations are in that directory. Ideally, I should migrate
away from that dependency and just move everything I can into dotfiles and
write some good documentation on how to set things up as I need.

Also, there is a `<pgp-key>` which should also be replaced with the correct value (if you want to sign your emails).
