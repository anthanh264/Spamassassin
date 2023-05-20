# Spamassassin
## Install
-  Install 
```
sudo apt-get update
sudo apt-get install spamassassin spamc -y
```
- Add a SpamAssassin user and disable the login.
```
sudo adduser spamd --disabled-login
```
## Config
### spamassassin file
- Edit file config
```
sudo nano /etc/default/spamassassin
```
- Add line ENABLED=0
- Find the line: `OPTIONS="--create-prefs .....` 
Change it to 
```
OPTIONS="--create-prefs --max-children 5 --username spamd --helper-home-dir /home/spamd/ -s /home/spamd/spamd.log"
```
- Find the line `CRON=0` change the value from 0 to 1 `CRON=1`

### SpamAssassin local configuration file.
- Backup and create new file 
```
sudo mv /etc/spamassassin/local.cf  /etc/spamassassin/local.cf.bk
sudo nano /etc/spamassassin/local.cf 
```
- Add content into file
```
rewrite_header Subject ***** SPAM _SCORE_ *****

report_safe             0

required_score          5.0

use_bayes               1

use_bayes_rules         1

bayes_auto_learn        1

skip_rbl_checks         0

use_razor2              0

use_dcc                 0

use_pyzor               0

ifplugin Mail::SpamAssassin::Plugin::Shortcircuit

endif
```
## Configure Postfix
### master file
```
sudo nano /etc/postfix/master.cf
```
- Locate these entries.
`smtp      inet  n       -       y       -       -       smtpd`
- Add this line below this
`-o content_filter=spamassassin`
![](https://hackmd.io/_uploads/rkbKyABB2.png)
This line is indented by one space compared to the smtp line, o parameter is displayed in blue  
- And also add at the end of the file
```
spamassassin unix -     n       n       -       -       pipe

user=spamd argv=/usr/bin/spamc -f -e  

/usr/sbin/sendmail -oi -f ${sender} ${recipient}

```
![](https://hackmd.io/_uploads/Hkc7eAHrh.png)

- Restart postfix and enable Spamassassin
```
sudo systemctl restart postfix.service
postfix reload
sudo systemctl enable spamassassin.service
sudo systemctl start spamassassin.service
```
## Test Spamassassin
- Send an email with content:
```
XJS*C4JDBQADN1.NSBN3*2IDNEN*GTUBE-STANDARD-ANTI-UBE-TEST-EMAIL*C.34X
```
- if error => WORKED 

- Spamasssassin add header to mail
![](https://hackmd.io/_uploads/rJ_fzABS3.png)
