# rsyslog4splunk

Splunk's best practice is to send your network device's syslog to a syslog server to be written to file/disk.  From there either the Splunk Universal or Heavy Forwarder will pick it up and send it to your Splunk Indexer(s).

This allows you to cache your syslog data stream to rsyslog while doing Splunk upgrades, restarts or if Splunk is down.

**Assumptions and Prerequisites**
Tested on Ubuntu Server 20.04 LTS, 21.04
Install up to date versions of:
- rsyslog
- logrotate
- zstd (optional, but highly recommended to save on syslog file storage)
- zfs/btrfs (optional)



**Hardware Recommendations:**
<=50GB/day of syslog ingest:
CPU Cores: >=6
RAM: >=8GB, more if using zstd for compression or add more if needed
Disk Space: >=150GB

>50GB/day of syslog ingest
CPU Cores: >=12
RAM: >=8GB, more if using zstd for compression or add more if needed
Disk Space: (Daily Syslog ingest GB) x 2 x 10%

Bar Napkin Math to calculate storage requirements with **NO** logrotate.conf compression: 
*Note: factor in storaged used for the OS after install and updates or mount a 2nd disk to /var/log/
  *If using compression, I recommend zstd.  zstd has much better compression that logrotate's default, gzip but at the expense of slightly higher CPU usage and 30 to 40% more RAM usage. As with any compression algorithm, higher compression settings have diminishing returns and isn't worth it at the expense of speed, CPU and RAM usage and storage savings.

###############################################################

Low-volume ingest example: 
- 42GB/day according to Splunk licensing dashboard
- 2 days of syslog data retention/rotation
- 10% disk overprovisioning

42GB x 2 x 10% = 92.4GB or just round it up to 95GB or 100GB

###############################################################

Medium-volume ingest example: 
- 126GB/day according to Splunk licensing dashboard
- 2 days of syslog data retention/rotation
- 10% disk overprovisioning

126GB x 2 x 10% = 277.2GB or just round it up to 280GB or 300GB

###############################################################

High-volume ingest example: 
- 395GB/day according to Splunk licensing dashboard
- 2 days of syslog data retention/rotation
- 10% disk overprovisioning

395GB x 2 x 10% = 869GB or just round it up to 870GB or 1TB


rsyslog folder hierarchy example:

/var/log/send2splunk/<sourcetype>/%HOSTNAME%/%$YEAR%-%$MONTH%-%$DAY%.log

  Replace <sourcetype> with the sourcetype you are using in Splunk so that it makes it far easier to stay organized when editing rsyslog.conf for the OS and inputs.conf for the Splunk Universal or Heavy Forwarder 
  For example, Cisco ASA:
  /var/log/send2splunk/cisco_asa/%HOSTNAME%/%$YEAR%-%$MONTH%-%$DAY%.log


Since rsyslog is receiving and storing logs on disk, it makes sense to keep at a minimum 48 hours or 2 days worth of logs in the event there is some kind problem where the Splunk Forwarder is not working, down momentarily while Splunk is being updated/upgraded or the Splunk Indexer(s) are not available to receive data.
  Firewall syslog can easily fill up a disk, so be sure to align your syslog server's storage to the daily ingest seen in Splunk License dashboard, add >=10% over provisioning for disk storage and enable either rsyslog compression (zstd, for best compression/performance ratio) for rotated files or use file system-level compression such as ZFS or BTRFS.  
  
Recommended rsyslog settings - logrotate.conf
  
compress
compresscmd /bin/zstd
compressext .zst
compressoptions -18 -T0 --rm
uncompresscmd /bin/unzstd
rotate 2
daily
nodelaycompress
nomail
notifempty
size 10M
include /etc/logrotate.d
