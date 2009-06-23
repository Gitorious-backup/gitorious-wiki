Dear all

#Short description how I successfully set up daemontools#
 to handle:

1. activemq
2. poller
3. git-daemon

I did it on Ubuntu Jaunty server edition at work and also Kubuntu Jaunty on my laptop.

Maybe I will try to handle the UltraSphinx daemon too, but for some reason I haven't done that yet.

Craig Websters words from 2008-12-13 13:34 helped me a lot:
http://barkingiguana.com/2008/11/28/running-daemontools-under-ubuntu-810
   
Should be said that I don't know the proper size of softlimits and stuff... 
Just copied it from http://barkingiguana.com/2008/11/28/running-daemontools-under-ubuntu-810

Daemontools (and don't confuse it with some virtual CD stuff for win... ) is very good 
because it supervises the daemons and restarts them in case they die. 
You have to set up the daemon and also the log for it so you can see if anything goes wrong...


**************************************
************** activemq **************
**************************************

      sudo adduser --system activemq
      sudo chown -R activemq /usr/local/apache-activemq-5.2.0/data
      sudo mkdir -p /usr/local/apache-activemq-5.2.0/service/activemq/{,log,log/main}

      sudo nano /usr/local/apache-activemq-5.2.0/service/activemq/run

`#!/bin/sh` 

exec 2>&1

USER=activemq

exec softlimit -m 1073741824 \
setuidgid $USER \
/usr/local/apache-activemq-5.2.0/bin/activemq


******************************************
************** activemq log **************
******************************************

      sudo nano /usr/local/apache-activemq-5.2.0/service/activemq/log/run

`#!/bin/sh`

USER=activemq
exec setuidgid $USER multilog t s1000000 n10 ./main

***********************************************************************
************** activemq rights, ownership and soft-links **************
***********************************************************************

      sudo sh -c "find /usr/local/apache-activemq-5.2.0/service/activemq -name 'run' |xargs chmod +x,go-wr"
      sudo chown activemq /usr/local/apache-activemq-5.2.0/service/activemq/log/main
      sudo ln -s /usr/local/apache-activemq-5.2.0/service/activemq /etc/service/activemq

      sudo svc -u /etc/service/activemq

      Tail the logs to make sure everything is happening as you'd expect.

      sudo tail -F /etc/service/activemq/log/main/current


************************************
************** poller **************
************************************

      sudo mkdir -p /home/git/service/poller/{,log,log/main}

      sudo nano    /home/git/service/poller/run

`#!/bin/sh`

exec 2>&1

USER=git

`#exec softlimit -m 1073741824 \`
`#setuidgid $USER \`
`#/var/www/gitorious/script/poller run`
`#to get the HOME env variable to work I changed to `
exec setuidgid $USER envdir ./env /var/www/gitorious/script/poller run


****************************************
************** poller log **************
****************************************

sudo nano      /home/git/service/poller/log/run

`#!/bin/sh`

USER=git
exec setuidgid $USER multilog t s1000000 n10 ./main


*********************************************************************
************** poller rights, ownership and soft-links **************
*********************************************************************


      sudo sh -c "find /home/git/service/poller -name 'run' |xargs chmod +x,go-wr"
      sudo chown git /home/git/service/poller/log/main
      sudo ln -s /home/git/service/poller /etc/service/poller

      sudo svc -u /etc/service/poller

      Tail the logs to make sure everything is happening as you'd expect.

      sudo tail -F /etc/service/poller/log/main/current


****************************************
************** git-daemon **************
****************************************

     sudo mkdir -p /home/git/service/git-daemon/{,log,log/main}

  sudo nano    /home/git/service/git-daemon/run

`#!/bin/sh`

exec 2>&1

USER=git

exec softlimit -m 1073741824 \
setuidgid $USER \
/var/www/gitorious/script/git-daemon run


********************************************
************** git-daemon log **************
********************************************

sudo nano      /home/git/service/git-daemon/log/run

`#!/bin/sh`

USER=git
exec setuidgid $USER multilog t s1000000 n10 ./main

*************************************************************************
************** git-daemon rights, ownership and soft-links **************
*************************************************************************

      sudo sh -c "find /home/git/service/git-daemon -name 'run' |xargs chmod +x,go-wr"
      sudo chown git /home/git/service/git-daemon/log/main
      sudo ln -s /home/git/service/git-daemon /etc/service/git-daemon

      sudo svc -u /etc/service/git-daemon

      Tail the logs to make sure everything is happening as you'd expect.

      sudo tail -F /etc/service/git-daemon/log/main/current
***************************************************************
Beginning to see a pattern here? 

Should be said that I don't know the proper size of the softlimits and stuff... 
Just copied it from http://barkingiguana.com/2008/12/13/deploying-activemq-on-ubuntu-810


I have also put a 
svc -t /etc/service/poller
in the crontab once per hour... 
It kills the poller. Daemontools restarts it immediately again.

My server is not used much and I think the poller looses the connection to the 
mysql server after a while with no activity... not sure but it usually stops working 
after long time... and this line in the crontab cures it... 
Maybe it is not needed anymore. I think there are some correspondence about it?
 
`Thanks and best regards`
`Martin`
