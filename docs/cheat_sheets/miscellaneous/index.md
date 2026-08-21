# Miscellaneous

## ffmpeg

```
ffmpeg -framerate 25 -i $PWD/%d.png -r 30  -b 5000k klein.mp4
ffmpeg -i klein.mp4 -loop 0 -vf scale=400:240 klein.gif
```