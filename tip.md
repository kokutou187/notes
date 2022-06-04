#### get video then obtain audio from it
```
// get the list of avaiable format codes for particular video
1. youtube-dl -F URL
// select format, or don't use -f option, default to use the highest quality
2. youtube-dl -f ID URL 
3. ffmpeg -i src dst.mp3