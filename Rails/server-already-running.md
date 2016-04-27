`sudo lsof -iTCP -sTCP:LISTEN -P | grep :3000`

ruby      19236 timrobertson   10u  IPv6 0x4c9aa06069a6e93b      0t0  TCP localhost:3000 (LISTEN)

`kill -9 19236`

`rm server.pid`
