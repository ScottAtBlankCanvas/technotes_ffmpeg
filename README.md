# technotes_ffmpeg

# ffmpeg and whisper

# Background

Found the whisper ffmpeg info via LinkedIn Post LI post: https://www.linkedin.com/posts/ericfontaine13_ffmpeg-openai-whisper-activity-7366499799753281537-yYNk

The Medium Article that I followed to build: https://medium.com/@vpalmisano/run-whisper-audio-transcriptions-with-one-ffmpeg-command-c6ecda51901f

# Difficulties I had with Medium Build Steps

I was building on a MacBook Pro 2019: 2.6 GHz 6-Core Intel Core i7

1. libpulse was not installed on my machine
2. ffmpeg scripts attempt to modify /usr/ instead of /usr/local/ (Mac SiP prevents this)
3. whisper lirary was not laoding with ffmpeg


Should find thgem in /usr/local/lib based on rpath setting (using otool) but had to force it via:

export DYLD_LIBRARY_PATH=/usr/local/lib:$DYLD_LIBRARY_PATH

9. ffmpeg --help filter=whisper

works!

## Summary of Modified Build steps:

```nix
# pulsa library was not found, so install it
brew install pulseaudio

# Build whisper (no changes from Medium article)
cd ~/code
git clone https://github.com/ggml-org/whisper.cpp
cd whisper.cpp
sh ./models/download-ggml-model.sh base.en
cmake -B build
cmake --build build --config Release
sudo make install -C build

# Build ffmpeg (changes denoted below)
cd ~/code
git clone https://code.ffmpeg.org/FFmpeg/FFmpeg.git
# I renamed the ffmpeg directory bc I have several
mv FFmpeg FFmpeg.whisper
cd FFmpeg.whisper

# Mac does not like to modify /usr so change prefix to /usr/local
./configure --prefix=/usr/local --enable-version3 --disable-shared --enable-gpl \
  --enable-nonfree --enable-static --enable-pthreads --enable-filters \
  --enable-openssl --enable-runtime-cpudetect --enable-libvpx --enable-libx264 \
  --enable-libx265 --enable-libspeex --enable-libfreetype --enable-fontconfig \
  --enable-libzimg --enable-libvorbis --enable-libwebp --enable-libfribidi \
  --enable-libharfbuzz --enable-libpulse --enable-libass --enable-whisper
make
sudo make install

# should not need this because whisper libraries were ijstalled to /usr/local/lib.  But only way ffmpeg executables wouod load whisper libs
export DYLD_LIBRARY_PATH=/usr/local/lib:$DYLD_LIBRARY_PATH

ffmpeg --help filter=whisper
```

# Running

ffprobe.  Need to turn off gpu and use a different file:

