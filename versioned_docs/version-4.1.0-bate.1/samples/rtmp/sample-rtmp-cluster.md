---
title: RTMP Cluster Deploy
sidebar_label: RTMP Cluster Deploy 
hide_title: false
hide_table_of_contents: false
---

# RTMP Edge Cluster Example

RTMP Edge cluster deploy example

RTMP Edge cluster is the kernel feature of SRS.

**Suppose the server ip is 192.168.1.170**

## Step 1, get SRS

For detail, read [GIT](../../doc/git.md)

```bash
git clone https://github.com/ossrs/srs
cd srs/trunk
```

Or update the exists code:

```bash
git pull
```

## Step 2, build SRS

For detail, read [Build](../../doc/build/install.md)

```bash
./configure && make
```

## Step 3, config origin SRS

For detail, read [RTMP](http://ossrs.net/srs.release/wiki/v4_EN_DeliveryRTMP) and [Edge](http://ossrs.net/srs.release/wiki/v4_EN_Edge)

Save bellow as config, or use `conf/origin.conf`:

```bash
# conf/origin.conf
listen              19350;
max_connections     1000;
pid                 objs/origin.pid;
srs_log_file        ./objs/origin.log;
vhost __defaultVhost__ {
}
```

## Step 4, config edge SRS

For detail, read [RTMP](http://ossrs.net/srs.release/wiki/v4_EN_DeliveryRTMP) and [Edge](http://ossrs.net/srs.release/wiki/v4_EN_Edge)

Save bellow as config, or use `conf/edge.conf`:

```bash
# conf/edge.conf
listen              1935;
max_connections     1000;
pid                 objs/edge.pid;
srs_log_file        ./objs/edge.log;
vhost __defaultVhost__ {
    cluster {
        mode            remote;
        origin          127.0.0.1:19350;
    }
}
```

## Step 5, start SRS

For detail, read [RTMP](http://ossrs.net/srs.release/wiki/v4_EN_DeliveryRTMP) and [Edge](http://ossrs.net/srs.release/wiki/v4_EN_Edge)

```bash
./objs/srs -c conf/origin.conf &
./objs/srs -c conf/edge.conf &
```

## Step 6, start Enocder

For detail, read [RTMP](http://ossrs.net/srs.release/wiki/v4_EN_DeliveryRTMP) and [Edge](http://ossrs.net/srs.release/wiki/v4_EN_Edge)

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

## Step 7, play RTMP

For detail, read [RTMP](http://ossrs.net/srs.release/wiki/v4_EN_DeliveryRTMP) and [Edge](http://ossrs.net/srs.release/wiki/v4_EN_Edge)

Origin RTMP url is: `rtmp://192.168.1.170:19350/live/livestream`, User can use vlc to play the RTMP stream. Or, use online SRS player: [srs-player-19350][srs-player-19350]

Edge RTMP url is: `rtmp://192.168.1.170:1935/live/livestream`, User can use vlc to play the RTMP stream. Or, use online SRS player: [srs-player][srs-player]

Note: Please replace all ip 192.168.1.170 to your server ip.

Winlin 2014.11