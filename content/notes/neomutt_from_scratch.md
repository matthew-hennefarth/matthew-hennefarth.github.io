---
title: "NeoMutt From Scratch"
date: 2023-08-03
draft: false
author: "Matthew R. Hennefarth"
---

# Prerequisite Software

We will be using `isync` to perform offline-imap storage. Then we will use `notmuch` to index our emails. We will use `neomutt` to actually view and edit our emails. Finally, we will use `msmtp` to actually send the emails through our ports. So at a minimum, lets install the following using your system package manager (or Homebrew for MacOS).

- `isync`
- `neomutt`
- `notmuch`
- `msmtp`

In addition, for security reasons, you should have a GPG key set up for encryption in order to securely store your passwords/tokens.

# IMAP Offline

The first step is going to be to setup `isync` so that we can download all of
our emails. For the majority of people, they will be using Gmail or Outlook. If
it is possible for either of these to use an app-password, that should be done.
Otherwise, the new default is to use XOAUTH2 as this is supposedly more secure. 

## App Password

If you are able to use an app password, then one should also install `pass` in
order to securely store the app password. `pass` is a great password manager
and it only requires a GPG key. To set it up just run 

```sh 
$ pass init "<GPG key>"
```

where `<GPG key>` is your GPG key that has been setup.

### Google App Password

If you are trying to setup an app password with Google, you need to go to
`Manage your Google Account`, `Security`, `2-Step Verification`, and finally
`App Passwords`. Proceed through creating a new app password (preferably for an
application called `neomutt`) and copy the password to your clipboad. Don't
close the page until you have stored the password into the `pass` database.

We can now add a new password to `pass` by 

```sh 
$ pass edit <email>
```

where `<email>` is your email address. Then paste in the app password in the
resulting file and save. Now if you run

```sh 
$ pass <email>
```

the app password should be returned (though this in general is not recommended
for security reasons)! Typically it is best to use the `-c` option to only keep
the password on your clipboard for a set amount of time (30 seconds) before
being essentially shredded. 

We will deal with adding an email account that requires 2FA for Gmail to
`neomutt`. Firstly, we need to get a client id and client secret using Google
Cloud. Make sure that we register the desktop version! Make sure to then
authorize the emails that you want with this application and that you allow less
secure applications for each of the email accounts.

### Outlook 

I am not sure how to generate app passwords for Outlook but allegedly they are
possible. If a password is generated, then also put it into `pass` as is done
in the [Google section](#google-app-password).

## Generating 2FA Tokens

If instead we need to generate 2FA tokens (because this is an organization or
school account), then we need to use the following script:
[mutt_oauth2.py](https://github.com/muttmua/mutt/blob/master/contrib/mutt_oauth2.py).
We need to fill in the client id and client secret in this file with that from
Google (or the Outlook one). I will not discuss how to generate the client id
and client secret as that is quite extensive, but a quick search should yield
instructions on how to do that. Additionally, replace the `YOUR_GPG_IDENTITY`
in the `ENCRPYTION` with the correct GPG Id to use. Place this script in
`~/.local/bin`. Next we can call it as (but first create the directory where we
will cache the tokens) 

```sh
$ mkdir -p ~/.cache/mbsync/
$ mutt_oauth2.py ~/.cache/mbsync/<email>.tokens --verbose --authorize
```

I then recommend using `authcode` but you may have to try all of the various
authorizations. Once this finishes, then there should be a file
`<email>.tokens`. 

## Setting the MBSYNC Config Files

We are now ready to just setup the `.mbsyncrc` and `msmtp/config` files. For
the `.mbsyncrc` file, we need to add the following lines, replacing `<email>`
with the email address of course:

```
IMAPStore <email>-remote
Host imap.gmail.com
Port 993
User <email>
PassCmd "<pass-cmd>"
AuthMechs <auth-mech>
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
Patterns * 
Create Both
SyncState *
MaxMessages 0
ExpireUnread no
# End profile
```

While we are at it, we can also setup the `.config/msmtp/config` as

```
account <email>
host smtp.gmail.com
port 465
from <email>
user <email>
passwordeval "<pass-cmd>"
auth <auth>
tls on
tls_trust_file	/etc/ssl/cert.pem
logfile ~/.cache/msmtp/msmtp.log
tls_starttls off
```

The above configuration is specific to a Gmail account (given the Hosts and Ports). For Outlook, the IMAP host should be `outlook.office365.com` with port `993`, and the SMTP host should be `smtp.office365.com` and port `587`. (Though this might be specific to exchange and not personal Outlook accounts, I would double check this).

Note that `<pass-cmd>`, `<auth-mech>`, and `<auth>` depend on if you have an [app password stored in `pass`](#app-password) or [2FA tokens](#generating-2fa-tokens).  If you are using an app password, then

- `<pass-cmd> = pass <email>`
- `<auth-mech> = LOGIN`
- `<auth> = on`

If you are using 2FA, then 

- `<pass-cmd> = mutt_oauth2.py ~/.cache/mbsync/<email>.tokens`
- `<auth-mech> = XOAUTH2`
- `<auth> = xoauth2`

Make sure to then create the mail directory as

```sh
$ mkdir ~/.local/share/mail/<email>
```

We can now run `mbsync <email>`. If everything has been set up properly, then
it should have created all of your email directories in
`~/.local/share/mail/<email>`.

Note, that you can modify the `Patterns *` option in the `.mbsyncrc` file to exclude directories that you do not want synced. For example, for Gmail, I don't like to use the `All Mail`, `Starred`, or `Important` folders, so to omit them from syncing I would use 

```
Patterns * !"[Gmail]/All Mail" !"[Gmail]/Starred" !"[Gmail]/Important"
```

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

# Notmuch

A good config for `notmuch` (which should be stored in `~/.config/notmuch/default/config`) is 

```
[database]
path=~/.local/share/mail
[user]
name=matthew.hennefarth
primary_email=<email>
[new]
tags=unread;inbox;
ignore=.mbsyncstate;.uidvalidity
[search]
exclude_tags=deleted;spam;
[maildir]
synchronize_flags=true
[crypto]
gpg_path=gpg
```

After running `mbsync -a` or `mbsync <email>` run `notmuch new` to index the files! 

# Automatic Syncing

The mutt-wizard `mailsync` script is a useful script to run periodically which will first download any new mail, notify if so, and run `notmuch` automatically. I would suggest downloading it and placing it somewhere in your `PATH` variable (such as `~/.local/bin`). 

You can then have `mailsync` run every so often using a `cron` job! This can be easily accomplished by editing your `cron` jobs using

```sh 
$ crontab -e
```

You can then add the following line:

```
*/1 * * * * <abs-path>/mailsync > /dev/null 2>&1
```

This sets the frequency to every 1 minute. Changing the 1 to a 5 will have it
run every 5 minutes instead. However, note that `cron` jobs are run as the root user and you need to provide the absolute path to `mailsync`. Also, it might be necessary to source some of your shell files (like `~/.zshenv`) so that the necessary environment variables are available. For example, I have

```
*/1 * * * * source /Users/mhennefarth/.zshenv && source /Users/mhennefarth/.config/zsh/.zprofile && /Users/mhennefarth/.local/bin/mailsync > /dev/null 2>&1
```

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
