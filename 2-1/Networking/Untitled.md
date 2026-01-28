```bash
which named

sudo apt install bind9

ss -tuln

sudo lsof -i -n -P

dig yahoo.com

dig @127.0.0.1(my loopback) www.yahoo.com

(check query time after multiple attempts, there are some timeout limits 5 min for google, 1 minute for yahoo, dont know what is this timeout actually)

Explain the  answer section, CNAME, etc

dig @127.0.0.1(my loopback) -t ns s00.zhjlab.bd


dig s01.zhjlab.bd



```



personal sub--domains:

```

	dns1.ac.bd handles the following sub-domains of zhjlab.bd

s{RR}.ZH

```


## GLUE RECORD OF dns1.du.ac.bd

| s01            | IN  | NS  | NS1.S01            |
| -------------- | --- | --- | ------------------ |
| NS1.S01        | IN  | A   | 10.17.100.1        |
| S02.ZHJLAB.BD. | IN  | NS  | NS1.S02.ZHJLAB.BD. |
| S03.           | IN  | A   | 10.17.100.2        |


To dig out all the records of the domain

```bash
dig +trace www.yahoo.com
dig +trace s01.zhjlab.bd

```

```bash
<<>> DiG 9.18.39-0ubuntu0.24.04.2-Ubuntu <<>> +trace s01.zhjlab.bd
;; global options: +cmd
.			2714	IN	NS	e.root-servers.net.
.			2714	IN	NS	f.root-servers.net.
.			2714	IN	NS	i.root-servers.net.
.			2714	IN	NS	b.root-servers.net.
.			2714	IN	NS	g.root-servers.net.
.			2714	IN	NS	a.root-servers.net.
.			2714	IN	NS	c.root-servers.net.
.			2714	IN	NS	j.root-servers.net.
.			2714	IN	NS	m.root-servers.net.
.			2714	IN	NS	d.root-servers.net.
.			2714	IN	NS	l.root-servers.net.
.			2714	IN	NS	k.root-servers.net.
.			2714	IN	NS	h.root-servers.net.
;; Received 239 bytes from 127.0.0.53#53(127.0.0.53) in 0 ms

bd.			172800	IN	NS	dns.bd.
bd.			172800	IN	NS	bd-ns.anycast.pch.net.
bd.			172800	IN	NS	surma.btcl.net.bd.
bd.			172800	IN	NS	jamuna.btcl.net.bd.
bd.			86400	IN	DS	7807 8 2 FEE7D4265C78882701E52EC56D56316A985047D0FC24F095BA90F772 458CB5EC
bd.			86400	IN	RRSIG	DS 8 1 86400 20260209170000 20260127160000 21831 . WgUGYKFekg2moX3MndudAm5xHPRj+S/WYiIVQZTaCFHHsBDUwszTKpEm nNevPT4tQS2H4xYMvqM+/OSEBGln+aKC2IeGiVEnLLLuhlStIw2Mvix9 9gFgU+4Amq9qnOA2gaYPEqNSsrC5rP1Fo6twg0azg1fUFeWdgGwjqDG3 0SxzipDC3sGtZMbO0hOh1+EC5tIviAc/0OWv6btSJreZhCwtEKkDx4Qk oFbg4hq1+6qu6qWaw+QIKRyiiso9JulzdjXI9KTrP+misq3XapoocGNX OKJpPbqkgd3yPGqC5g0P1LOgmuuCXHleKUrYiGPXLs5DUc00mOu3u2NR jO4+cg==
;; Received 656 bytes from 199.7.83.42#53(l.root-servers.net) in 152 ms

zhjlab.bd.		86400	IN	NS	dns1.du.ac.bd.
zhjlab.bd.		86400	IN	NS	dns2.du.ac.bd.
zhjlab.bd.		86400	IN	NSEC	zia.bd. NS RRSIG NSEC
zhjlab.bd.		86400	IN	RRSIG	NSEC 8 2 86400 20260218123212 20260119113212 15720 bd. kdrY30X0HXmExCcaCkoUa1GzxTIbh3YBtjfwrr8PR8rAkb2k9Bj3L65I /kfa7uLrV/KB/lMC8XH2plWS9GBgCruC16Di7nIyiUlTJf6N+zi38/IK f0N6jO9exFYFaa6DrDTq+RBxz6BYkcFHhdtg9dgFdpoLkvOSFsqkJrT/ oEg=
;; Received 276 bytes from 2407:5000:88:4::232#53(surma.btcl.net.bd) in 48 ms

s01.zhjlab.bd.		86400	IN	NS	ns1.s01.zhjlab.bd.
couldn't get address for 'ns1.s01.zhjlab.bd': failure
dig: couldn't get address for 'ns1.s01.zhjlab.bd': no more

```



---
DNS Topics:
	1. Dig
	2. Host
	3. Bind9
	