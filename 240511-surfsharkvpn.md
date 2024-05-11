# Make ![Surfshark-VPN icon](https://surfshark.com/website/_next/public/global/favicon-32.png)Surfshark-VPN to connect to a specific server without responding to prompts on Linux(legacy)
✨ Created on 2024/5/11

### 1. Install Surfshark-VPN of Linux:  
https://support.surfshark.com/hc/en-us/articles/360017418334-How-to-set-up-Surfshark-VPN-on-Linux-Legacy-version

### 2. Start Surfshark-VPN to connect to a specific server in the usual way:
```sh
sudo surfshark-vpn
```

### 3. Get the command of openvpn:
```sh
ps -eo command | grep ovpn
```
> [!NOTE]
> The following result will be displayed. And the first line is the command of openvpn.
```sh
/usr/sbin/openvpn --config /root/.surfshark/configs/jp-tok.prod.surfshark.comsurfshark_openvpn_udp.ovpn --auth-user-pass /root/.surfshark/credentials/authpass.txt --log-append /var/log/surfshark-vpn.log --status /var/lib/surfshark.status 1
grep --color=auto ovpn
```

### 4. Create a shell to start Surfshark-VPN:
> [!IMPORTANT]
> Add `sudo` to the beginning of the openvpn command  
> And add `--daemon --writepid /root/.surfshark/run/go-shark.pid` to the end of the openvpn command
```sh
#!/bin/sh
sudo /usr/sbin/openvpn --config /root/.surfshark/configs/jp-tok.prod.surfshark.comsurfshark_openvpn_udp.ovpn --auth-user-pass /root/.surfshark/credentials/authpass.txt --log-append /var/log/surfshark-vpn.log --status /var/lib/surfshark.status 1 --daemon --writepid /root/.surfshark/run/go-shark.pid
if [ $? -ne 0 ]
then
  exit 10
fi

# Wait until the VPN is ready.
str=""
while [ -z "${str}" ]
do
  /usr/bin/sleep 0.5
  str=$(tail -1 /var/log/surfshark-vpn.log | grep "Initialization Sequence Completed")
done
```
> [!TIP]
> Now you can check status or disconnect Surfshark-VPN in the usual way.
```sh
sudo surfshark-vpn status
sudo surfshark-vpn down
```

[🚗BACK](/)
