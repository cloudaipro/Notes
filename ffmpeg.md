### CROP the video,  1920 x 1080 ==> 1920 x 886,
```
ffmpeg -i App_Preview.mp4 -vf "crop=1920:886:0:97" -c:a copy output_video.mp4
```

### merge video
```
ffmpeg -f concat -i conv.sh -c copy large.mp4
```
