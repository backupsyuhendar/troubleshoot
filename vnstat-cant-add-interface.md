# Vnstat can't add interface 

```bash
❯ vnstat --add -i wlp0s29f7u1
Adding interface "wlp0s29f7u1" for monitoring to database...
Error: Exec step failed (8: attempt to write a readonly database): "insert into interface (name, active, created, updated,
rxcounter, txcounter, rxtotal, txtotal) values ('wlp0s29f7u1', 1, datetime('now', 'localtime'), datetime('now', 'localtime'), 0, 0, 0, 0)"
Error: Adding interface "wlp0s29f7u1" to database failed.
```

```bash
❯ sudo vnstat --add -i wlp0s29f7u1
[sudo] password for dar729:
Adding interface "wlp0s29f7u1" for monitoring to database...

Restart the vnStat daemon if it is currently running in order to start monitoring "wlp0s29f7u1".
```

```bash
❯ vnstat
...
 wlp0s29f7u1: Not enough data available yet.
...

❯ sudo systemctl restart vnstat.service

❯ vnstat
...
 wlp0s29f7u1:
       2021-08           0 B  /         0 B  /         0 B  /     --
         today           0 B  /         0 B  /         0 B  /     --
...

```
