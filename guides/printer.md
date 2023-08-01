# Printer installation

References:

1. (Arch Wiki) [Arch linux install network printer](https://kb.adamsdesk.com/operating_system/arch_linux_install_network_printer/)
2. (Arch Wiki) [Avahi Hostname resolution](https://wiki.archlinux.org/title/Avahi#Hostname_resolution)

Install packages
```
# pacman -S cups cups-pdf
```

Enable CUPS service with systemd
```
# systemctl enable cups.socket
```

Install printer driver
```
$ paru -S brother-dcp-t710w-lpr-bin 
```

Link printer files
```
# ln -s /opt/brother/Printers/dcpt710w/cupswrapper/brother_lpdwrapper_dcpt710w /usr/lib/cups/filter/brother_lpdwrapper_dcpt710w
# ln -s /opt/brother/Printers/dcpt710w/cupswrapper/cupswrapperdcpt710w /usr/lib/cups/filter/cupswrapperdcpt710w 
```

Open ports on firewall
```
# ufw allow 5353/udp
```

Hostname resolution for Avahi
```
# pacman -S nss-mdns
```

Enable Avahi service with systemd
```
# systemctl enable --now avahi-daemon.service
```

Add hostname resolution to a file
```
/etc/nsswitch.conf
.....
-- hosts: mymachines resolve [!UNAVAIL=return] files myhostname dns
++ hosts: mymachines mdns_minimal [NOTFOUND=return] resolve [!UNAVAIL=return] files myhostname dns
```

Restart CUPS service with systemd
```
# systemctl restart cups.service
```