# guide-tailscale-on-edgerouter
A Guide on how to setup Tailscale on the EdgeRouter.

> [!CAUTION]
> This Was Tested for EdgeOS Firmware 3.0.1 (E100). This Guide is Designed for The EdgeRouter Lite 3 and the mips64 Architecture.

> [!CAUTION]
> This is not actively maintained and I am not responsible if this script breaks any configuration or environment. Use at your own risk. It is not possible for me to test every use case and therefore I cannot assume responsibility when things go wrong.

> [!WARNING]
>  This Install will not survive Firmware Upgrades so it will need to be re done every time the router is updated. 

# Guide

SSH Into Your Edge Router and login with an admin account, By default thats ubnt.
~~~ bash
ssh <admin>@<router_ip>
~~~
Then Change into root via the command
~~~ bash
sudo -i
~~~
Although EdgeOS is based of Debain 9 For Security Reasons the Package Manager has no sources and is near completley disabled so we will have to download and install the binary for tailscale manually. CD into a Temporary Directory and Download the latest version with the following command.
~~~ bash
cd /tmp
curl -fsSL https://pkgs.tailscale.com/stable/tailscale_1.94.2_mips64.tgz -o tailscale.tgz
~~~
Next we will need to extract the two binarys from the archive.
~~~ bash
tar xzf tailscale.tgz
cd tailscale_*
~~~
Copy Them to there respective binary folders.
~~~ bash
cp tailscaled /usr/sbin/
cp tailscale /usr/bin/
~~~
After that we will need to make them executeable with the chmod command.
~~~ bash
chmod +x /usr/sbin/tailscaled
chmod +x /usr/bin/tailscale
~~~
Congratualtions, You should now have installed tailscale to your edge router. But the daemon wont be running so we will need to create a service and install it with.

> [!WARNING]
> The Tailscale Daemon wont start on reboot so a init script must becreated following the steps bellow. 
~~~ bash
cat > /etc/init.d/tailscaled <<'EOF'
#!/bin/sh
### BEGIN INIT INFO
# Provides:          tailscaled
# Required-Start:    $network $local_fs
# Required-Stop:     $network $local_fs
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Tailscale VPN daemon
### END INIT INFO
 
DAEMON=/usr/sbin/tailscaled
PIDFILE=/var/run/tailscaled.pid
 
case "$1" in
  start)
    echo "Starting tailscaled..."
    start-stop-daemon --start --pidfile $PIDFILE --make-pidfile --background --exec $DAEMON -- --state=/var/lib/tailscale/tailscaled.state --tun=userspace-networking
    ;;
  stop)
    echo "Stopping tailscaled..."
    start-stop-daemon --stop --pidfile $PIDFILE
    rm -f $PIDFILE
    ;;
  restart)
    $0 stop
    sleep 2
    $0 start
    ;;
  status)
    if [ -f $PIDFILE ]; then
      echo "tailscaled is running (PID $(cat $PIDFILE))"
    else
      echo "tailscaled is not running"
    fi
    ;;
  *)
    echo "Usage: /etc/init.d/tailscaled {start|stop|restart|status}"
    exit 1
    ;;
esac
 
exit 0
EOF
~~~
Then Make it executable and Update OpenRC with this command
~~~ bash
chmod +x /etc/init.d/tailscaled
update-rc.d tailscaled defaults

