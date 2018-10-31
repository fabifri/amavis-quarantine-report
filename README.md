# amavis-quarantine-report

## About

This script generates automated email reports for quarantined items in /var/lib/amavis/virusmails/ on a per-mailbox basis and allows users to release items from their quarantine via email. Inspired by @le1ca's @le1ca/spam-report.

Releasing quarantined items works via an alias (eg. spammgr@yourdomain.com) by forwarding all incoming mails to the python script with the `--release` parameter. The generated email reports include a release link in the form of `mailto:spammgr@yourdomain.com?subject=x-amavis-release:xxx` where xxx is the ID of the item within amavis quarantine. No additional open ports or http server on your MDA/MTA required, since releasing of quarantined items is done solely via email commands.

Tested with amavisd-new-2.10.x and postfix 3.x but currently *not* production ready.


## Install

1) Clone the repo via git and adjust the vars in config.ini

2) Create a new postfix alias maps file like the following

		echo "spammgr@yourdomain.com spammgr" >/etc/postfix/amavis_qmgr_transport
		postmap /etc/postfix/amavis_qmgr_transport

3) Add the transport file to `virtual_alias_maps` in /etc/postfix/main.cf, eg.

		..
		virtual_alias_maps = hash:${config_directory}/amavis_qmgr_transport
		..
 
4) Add a new entry in /etc/aliases for message releasing via email, eg.

		..
		# amavis-quarantine-report
		spammgr: "|/usr/bin/python3 /opt/amavis-quarantine-report/amavis-quarantine-report.py --release"
		..

5) Run the `newaliases` command

6) Add a cronjob for the automated report generation, eg.

		5 0 * * * /usr/bin/python3 /opt/amavis-quarantine-report/amavis-quarantine-report.py --send-reports >/opt/amavis-quarantine-report/report.log 


## Todo

- Fix encoding and locale issues
- Add a better/safer way for releasing quarantined items
- Add multi-language email templates
- Clean-up
