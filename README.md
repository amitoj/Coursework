# Coursework

## Running
With docker installed:
```bash
docker-compose up --build
```

With the containers running, to run a command inside the attacker container:
```bash
docker exec coursework_attacker $command
```

To run an interactive shell:
```bash 
docker exec -it coursework_attacker /bin/sh
```

Then to run the attack script from inside the attacker just run:
```bash
python attack.py
```

Or avoiding the shell altogether:
```bash 
docker exec -ti coursework_attacker sh -c "python attack.py"
```

To run multiple attackers potentially using multiple DNS servers:
```bash
./attacker/multiple_attacks.sh <dns or web> <number_processes_per_attacker> <last byte of ip address of DNS for first attacker > <last byte of ip address of DNS for second attacker>...
```
For example, to run an attack against the web victim with 4 attackers, each with 2 processes against DNSes running on 172.16.238.8 and 172.16.238.9, each being hit by two attackers: 
```bash
./attacker/multiple_attacks.sh web 2 8 8 9 9
```

To sniff DNS packets incoming to the victim:
```bash 
docker exec -it coursework_victim_web .venv/bin/python sniff.py
```

To see Flask app: navigate to http://localhost

To see DNS admin panel: navigate to https://localhost:10000/ and log in with `root`/`highentropy`
## Wireshark
If Wireshark is not installed, run:
```bash
sudo apt install wireshark-qt
```
To access Wireshark, cd to where the tcpdump folder is and run:
```bash
tail -c +1 -f tcpdump/tcpdump.pcap | wireshark -k -i -
```
To see the dns packets for our network, in the filter, type ip.addr == 172.16.238.100

If you get access denied:
```bash
sudo dpkg-reconfigure wireshark-common
```
Select yes, then:
```bash
sudo chmod +x /usr/bin/dumpcap
```

## DNS config
The DNS server is set up to be the authoritative NS for example.com.

There are multiple configs:
- base config: `dns/db.example.com`, which the signed config is generated from
- signed config: `dns/db.example.com.signed`
- simple config: `dns/db.example.com.simple`, which does not mention the keys, and should
 be kept in line with the base config.

After changing the base config, to regenerate the signed config run `./gen-signed.sh` in `/data/bind/etc` on the DNS container.

If the `gen-signed.sh` script takes more than a few seconds, you might not have enough entropy and the process will take
a very long time. Stop the script and install haveged (`apt-get install haveged`) on the host machine, and try again.

To change which config is used modify this line `file "/etc/bind/db.example.com.signed";`
in `named.conf.local`.

To copy the signed config back out run:
```bash
docker cp coursework_dns:/data/bind/etc/db.example.com.signed ./dns/db.example.com.signed
```
