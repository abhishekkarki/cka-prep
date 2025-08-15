# DNS

 In linux you can put the host name with the maping of it's ip like:

  ```bash
    # cat >> /etc/hosts
    192.168.1.11  db
    192.168.1.11  www.google.com
  ```
Now if you ping from system A to B which was db `ping db` it will work. 

Here the system will not verify it though. Translating host name from the IP is called **Name Resolution**

This is ok for couple of hosts, what if the hosts will increase then offcourse this list will also increase and it is hard to maintiain it.

As a solution we migrate this whole list to a dedicated server called **DNS**. And now we configure each host to refer this DNS in order to resolve the address.

### Configuring your host for a particular DNS

For refering to a particular DNS you have follow this:

```bash
  # cat /ect/resolve.conf
  nameserver     192.168.1.100
```

Even though you have the entry on the DNS server you can still have the same into your `/etc/hosts` and in this case when the entry is in both place by default it will first
check it locally and then the DNS.

But this order can be change like

```bash
  # cat /etc/nsswitch.conf
  hosts:     files dns
```

 You can also add public name server in your `/etc/resolv.conf` file for example
 ```bash
  # google nameserver
  nameserver    8.8.8.8 
```

## Domain Names
Root 
  |
Top level Domain


For example if you want to have inside the company they just use `web` and it should be resolve to `web.mycompany.com` to do this you can add this:
```bash
  # cat >> /etc/resolv.conf
  search     mycompany.com   prod.mycompany.com
```



## Record Types




**nslookup**: You can use it to query the address

`nslookup www.google.com`

Note: If you add an entry locally in the `/etc/hosts` this nslookup will not work there. It only lookup the DNS.

The another option is `dig`.

`dig www.google.com`







