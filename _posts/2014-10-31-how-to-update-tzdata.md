---
title: "How to manually update tzdata"
categories:
  - howto
tags:
  - tzdata
  - linux
  - java
  - mysql
---

This can be useful if your distributive has outdated information about time zones (time shifts especially) and you need to update it manually.

## Linux
1. Check for current time shifts (just to be sure do you have required settings or not):
```
zdump -v /etc/localtime | grep `date +"%Y"`
```
1. Go to ftp://ftp.iana.org/tz/releases/ and download required (latest) tzdata file;
1. Compile zone file:
```
zic -d {temp_dir} {file_to_compile}
```
1. Copy required zone definitions into `/usr/share/zoneinfo` (alternatively it can be built with `make`);
1. Update `/etc/localtime` to point to correct zone:
```
rm /etc/localtime 2>/dev/null; ln -s /usr/share/zoneinfo/Europe/London /etc/localtime
```
1. Check current settings:
```
zdump -v /etc/localtime | grep `date +"%Y"`
ntpdate -q pool.ntp.org
```

## Java
Download `Timezone Updater Tool` from Oracle site and then for each JRE/JDK installations run:
```
${JAVA_HOME}/bin/java -jar tzupdater.jar 
```

## MySQL
1. Check for date/time settings:
```
SELECT @@global.time_zone, @@session.time_zone;
```
If both options have value `SYSTEM` most probably don't need to do anything.
1. Update TZ information:
```
mysql_tzinfo_to_sql /usr/share/zoneinfo/Europe/London 'Europe/London' | mysql -u root mysql
```
3. Check for result:
```
select now(), sysdate() from dual;
```


