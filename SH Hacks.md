---
tags:
  - Tools/CMD
type: page
---

---

### Errors
This application failed to start because no Qt platform plugin could be initialized. Reinstalling the application may fix this problem:

```
export QT_QPA_PLATFORM=offscreen
```

### Disk space

```
ncdu
```

### TMUX

```
set -g mouse on
```

### CPU Temp

```
sudo apt install lm-sensors
sudo sensors-detect
sensors
```

### FFMPEG - Convert video to GIF
-ss second to start from
-t duration in seconds
```
ffmpeg -i input.mp4 -ss 0 -t 1 -vf "setpts=0.5*PTS,fps=10,scale=320:-1:flags=lanczos" -loop 0 output.gif
```