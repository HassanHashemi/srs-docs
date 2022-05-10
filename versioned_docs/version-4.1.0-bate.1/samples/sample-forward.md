---
title: Forward Deploy
sidebar_label: Forward Deploy
hide_title: false
hide_table_of_contents: false
---

# Forward deploy example

SRS can forward stream to other RTMP server.

**Suppose the server ip is 192.168.1.170**

Forward will copy streams to other RTMP server:
* Master: Encoder publish stream to master, which will forward to slave.
* Slave: Slave forward stream to slave.

We use master to listen at 1935, and slave listen at 19350.

## Step 1, get SRS

For detail, read [GIT](https://github.com/ossrs/srs/wiki/v4_EN_Git)

```bash
git clone https://github.com/ossrs/srs
cd srs/trunk
```

Or update the exists code:

```bash
git pull
```

## Step 2, build SRS

For detail, read [Build](https://github.com/ossrs/srs/wiki/v4_EN_Build)

```bash
./configure && make
```

## Step 3, config master SRS

For detail, read [Forward](https://github.com/ossrs/srs/wiki/v4_EN_Forward)

Save bellow as config, or use `conf/forward.master.conf`:

```bash
# conf/forward.master.conf
listen              1935;
max_connections     1000;
pid                 ./objs/srs.master.pid;
srs_log_tank        file;
srs_log_file        ./objs/srs.master.log;
vhost __defaultVhost__ {
    forward {
        enabled on;
        destination 127.0.0.1:19350;
    }
}
```

## Step 4, start master SRS

For detail, read [Forward](https://github.com/ossrs/srs/wiki/v4_EN_Forward)

```bash
./objs/srs -c conf/forward.master.conf
```

## Step 5, config slave SRS

For detail, read [Forward](https://github.com/ossrs/srs/wiki/v4_EN_Forward)

Save bellow as config, or use `conf/forward.slave.conf`:

```bash
# conf/forward.slave.conf
listen              19350;
pid                 ./objs/srs.slave.pid;
srs_log_tank        file;
srs_log_file        ./objs/srs.slave.log;
vhost __defaultVhost__ {
}
```

## Step 6, start slave SRS

For detail, read [Forward](https://github.com/ossrs/srs/wiki/v4_EN_Forward)

```bash
./objs/srs -c conf/forward.slave.conf
```

Note: Ensure the master and slave is ok, no error in log.

```bash
[winlin@dev6 srs]$ sudo netstat -anp|grep srs
tcp        0      0 0.0.0.0:1935                0.0.0.0:*                   LISTEN      7826/srs            
tcp        0      0 0.0.0.0:19350               0.0.0.0:*                   LISTEN      7834/srs
```

## Step 7, start Encoder

For detail, read [Forward](https://github.com/ossrs/srs/wiki/v4_EN_Forward)

Use FFMPEG to publish stream:

```bash
    for((;;)); do \
        ./objs/ffmpeg/bin/ffmpeg -re -i ./doc/source.flv \
        -c copy \
        -f flv rtmp://192.168.1.170/live/livestream; \
        sleep 1; \
    done
```

Or use FMLE to publish:

```bash
FMS URL: rtmp://192.168.1.170/live
Stream: livestream
```

The stream in SRS:
* Stream publish by encoder: rtmp://192.168.1.170:1935/live/livestream
* The stream forward by master SRS: rtmp://192.168.1.170:19350/live/livestream
* Play stream on master: rtmp://192.168.1.170/live/livestream
* Play strema on slave: rtmp://192.168.1.170:19350/live/livestream

## Step 8, play the stream on master

For detail, read [Forward](https://github.com/ossrs/srs/wiki/v4_EN_Forward)

RTMP url is: `rtmp://192.168.1.170:1935/live/livestream`

User can use vlc to play the RTMP stream.

Or, use online SRS player: [srs-player][srs-player]

Note: Please replace all ip 192.168.1.170 to your server ip.

## Step 9, play the stream on slave

For detail, read [Forward](https://github.com/ossrs/srs/wiki/v4_EN_Forward)

RTMP url is: `rtmp://192.168.1.170:19350/live/livestream`

User can use vlc to play the RTMP stream.

Or, use online SRS player: [srs-player-19350][srs-player-19350]

Note: Please replace all ip 192.168.1.170 to your server ip.

Winlin 2014.11

[nginx]: http://192.168.1.170:8080/nginx.html
[srs-player]: http://ossrs.net/srs.release/trunk/research/players/srs_player.html?vhost=__defaultVhost__&autostart=true&server=192.168.1.170&app=live&stream=livestream&port=1935
[srs-player-19350]: http://ossrs.net/srs.release/trunk/research/players/srs_player.html?vhost=__defaultVhost__&autostart=true&server=192.168.1.170&app=live&stream=livestream&port=19350
[srs-player-ff]: http://ossrs.net/srs.release/trunk/research/players/srs_player.html?vhost=__defaultVhost__&autostart=true&server=192.168.1.170&app=live&stream=livestream_ff
[jwplayer]: http://ossrs.net/srs.release/trunk/research/players/srs_player.html?app=live&stream=livestream.m3u8&server=192.168.1.170&port=8080&autostart=true&vhost=192.168.1.170&schema=http&hls_autostart=true&hls_port=8080
[jwplayer-ff]: http://ossrs.net/srs.release/trunk/research/players/srs_player.html?app=live&stream=livestream_ff.m3u8&server=192.168.1.170&port=8080&autostart=true&vhost=192.168.1.170&schema=http&hls_autostart=true&hls_port=8080
