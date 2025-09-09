---
title: ffmpeg 编译
date: 2025-09-09 10:53:00
tags: [MinGW, C, msys2]
---

1. 安装 Msys2
2. 打开 MSYS2 MINGW64
3. 安装依赖
   ``` shell
   pacman -Su
   pacman -S make pkg-config yasm nasm
   pacman -S mingw-w64-x86_64-toolchain
   ```
4. 下载 ffmpeg库
5. 配置 编译
   ``` shell
   ./configure \
   --prefix=../ffmpeg \
   --disable-programs \
   --disable-doc \
   --disable-avdevice \
   --disable-swscale \
   --disable-postproc \
   --disable-avfilter \
   --disable-everything \
   --enable-swresample \
   --enable-protocol=file \
   --enable-demuxer=wav \
   --enable-demuxer=mp3 \
   --enable-parser=mpegaudio \  
   --enable-parser=gsm \
   --enable-decoder=pcm_s16le \
   --enable-decoder=pcm_s24le \
   --enable-decoder=pcm_f32le \
   --enable-decoder=gsm_ms \   
   --enable-decoder=mp3 \ 
   --enable-shared \
   --disable-static \
   --extra-ldflags="-static -lpthread -liconv"

   make -j$(nproc) && make install
   ```