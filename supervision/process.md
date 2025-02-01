[Install nagios deps](https://support.nagios.com/kb/article/nagios-core-installing-nagios-core-from-source-96.html#Ubuntu)
[]()


# Nagios installation on server  ( ubuntu )


```bash
sudo mkdir :/usr/local/nagios/etc/servers
# Modify
sudo vi /usr/local/nagios/etc/objects/localhost.cfg

# in define host , add the line below
max_check_attempts 5
```


# Nagios installation on client  ( alpine )

## Step 2: after nrpe installation  
```bash
# Open the file which already exists
sudo vi /etc/<nrpe.cfg | nagios/nrpe.cfg>

# Put on line 51 or 61 ( after the comment server_address=127.0.0.1)
server_address=192.168.56.16
# Change allowed host to add the new server address and delete ::1
allowed_hosts=127.0.0.1,192.168.56.16

# restart nrpe service
sudo rc-service nrpe restart
```

