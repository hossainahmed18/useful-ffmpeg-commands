1.ffmpeg command to loop an mp4 file, and send output video-audio to rtmp push destination(like medialive input) 

```
ffmpeg -re -stream_loop -1 -i dolphin.mp4 \
-vf "scale=1920:1080,drawtext=text='%{gmtime}': fontcolor=white: fontsize=48: x=w-tw-10: y=10" -r 30 \
-c:v libx264 -preset veryfast -b:v 5000k -maxrate 5000k -bufsize 10000k \
-c:a aac -b:a 128k -ar 44100 -f flv rtmp://ip:port/{Application Name}/{Application instance} \
-loglevel debug

```

N.B. if application name is not there, then user Application instance twice.(like: rtmp://ip:port/{Application instance}/{Application instance). use ```localtime``` to send your  timezone time

2. Open a SRT stream in listener mode looping an mp4 file and play
```
ffmpeg -re -stream_loop -1 -i dolphin.mp4 -c:v libx264 -c:a aac -f mpegts "srt://127.0.0.1:40052?mode=listener" 
```
add latency param. like: latency=20000000

to play(in caller mode):
```
ffplay "srt://127.0.0.1:40052?mode=caller"
```

for remote connection user:

```
ffmpeg -re -stream_loop -1 -i input.mp4 -c:v libx264 -c:a aac -f mpegts "srt://{ip}:{port}?mode=listener&passphrase=pass"
```
to play:
```
ffplay "srt://{ip}:{port}?mode=caller&passphrase=pass"
```

3. To Send input to Mediaconnect

```
ffmpeg -re -stream_loop -1 -i dolphin.mp4 \
-vf "scale=1920:1080,drawtext=text='%{gmtime}':fontcolor=white:fontsize=48:x=w-tw-10:y=10" \
-c:v libx264 -c:a aac -r 30 -f mpegts \
"srt://{Mediaconnect-Source-Inbound-IP-address}:{Mediaconnect-source-Inbound-port}"
```

Configs in MediaConnect flow:
   Protocol: SRT Listener
   Allowlist CIDR block: 0.0.0.0/0 (To allow all IP, but it's not recommended as it goes to open for all)


4. Add a time code to video

```
ffmpeg -i input.mp4 -vf "drawtext=text='%{pts\\:hms}':fontcolor=white:fontsize=48:x=(w-text_w)/2:y=(h-text_h)/2:box=1:boxcolor=black@0.5" -c:a copy input_1.mp4
```

5. convert mp4 to mp3

```
ffmpeg -i input.mp4 -codec: copy -start_number 0 -hls_time 10 -hls_list_size 0 -f hls ffmpeg-outputs/input.m3u8
```

6. convert hls to mp3

```
ffmpeg -i "http://example.com/your_hls_stream.m3u8" -q:a 0 -map a output.mp3
```

7.  convert hls to mp4: 

```
ffmpeg -i http://example.com/your_hls_stream.m3u8 -acodec copy -vcodec copy out.mp4
```
