# Compile And Install FFmpeg

([The original link](http://www.osxexperts.net/ffmpeg/ffmpegexperts.html)﻿)

## STEP 1 - Preparations

### Install `XCode`(include `git`)

## STEP 2 - Downloading all necessary source codes

Download all file into a folder named `FFmpeg` on the your disk:

### 1、FFmpeg source code

- Open the Terminal
- write the following in the Terminal : `git clone git://git.videolan.org/ffmpeg.git ffmpeg`
Because `FFmpeg` needs several extra codecs you need some other source codes too. 

### 2、MP3lame source code

- Go to `http://lame.sourceforge.net/download.php`
- Download lame source

### 3、Ogg, Vorbis and Theora source code

- Go to `http://xiph.org/downloads/`
- Download the following sources: `libogg` source , `libvorbis` source and `libtheora` source.

### 4、X264 source code (for H264 support)

- Open the Terminal
- write the following in the Terminal : `git clone git://git.videolan.org/x264.git`

### 5、VP8 source code

- Open the terminal
- write the following in the Terminal : `git clone git://review.webmproject.org/`libvpx.git

### 6、Xvid source code

- Go to `http://www.xvid.org/Downloads.43.0.html`
- Download `Xvid` source code


> All source code has been downloaded [here](https://github.com/ljf1239848066/FFmpeg/tree/master/zip).

## STEP 3 - Compiling all source codes

Open the Terminal and copy paste every line below marked **BOLD**.

	DISK_ID=$(hdid -nomount ram://26214400) && newfs_hfs -v tempdisk ${DISK_ID} && diskutil mount ${DISK_ID} && SOURCE="/Volumes/tempdisk/sw" && COMPILED="/Volumes/tempdisk/compile" && mkdir ${SOURCE} && mkdir ${COMPILED} && export PATH=${SOURCE}/bin:$PATH


### Compiling YASM:

	cd ${COMPILED} || exit 1
	cd yasm-1.2.0
	./configure --prefix=${SOURCE} && make -j 4 && make install

### Compiling LIBVPX

	cd ${COMPILED}
	cd libvpx
	./configure --prefix=${SOURCE} --disable-shared && make -j 4 && make install

### Compiling LAME

	cd ${COMPILED}
	cd lame-3.99
	./configure --prefix=${SOURCE} --disable-shared --enable-static && make -j 4 && make install

### Compiling XVIDCORE

	cd ${COMPILED}
	cd xvidcore
	cd build/generic
	./configure --prefix=${SOURCE} --disable-shared --enable-static --disable-assembly && make -j 4 && make install
	rm ${SOURCE}/lib/libxvidcore.4.dylib

### Compiling X264

	cd ${COMPILED}
	cd x264
	./configure --prefix=${SOURCE} --disable-shared --enable-static && make -j 4 && make install && make install-lib-static

### Compiling LIBOGG

	cd ${COMPILED}
	cd libogg-1.3.0
	./configure --prefix=${SOURCE} --disable-shared --enable-static && make -j 4 && make install

### Compiling LIBVORBIS

	cd ${COMPILED}
	cd libvorbis-1.3.2
	./configure --prefix=${SOURCE} --with-ogg-libraries=${SOURCE}/lib --with-ogg-includes=/Volumes/tempdisk/sw/include/ --enable-static --disable-shared && make -j 4 && make install

### Compiling LIBTHEORA

	cd ${COMPILED}
	cd libtheora-1.1.1
	./configure --prefix=${SOURCE} --with-ogg-libraries=${SOURCE}/lib --with-ogg-includes=${SOURCE}/include/ --with-vorbis-libraries=${SOURCE}/lib --with-vorbis-includes=${SOURCE}/include/ --enable-static --disable-shared && make -j 4 && make install


~~Compiling ZLIB~~

	cd ${COMPILED}
	cd zlib
	./configure --prefix=${SOURCE} && make -j 4 && make install
	rm ${SOURCE}/lib/libz*dylib
	rm ${SOURCE}/lib/libz.so*

### Compiling FFMPEG

	cd ${COMPILED}
	cd ffmpeg
	export LDFLAGS="-L${SOURCE}/lib"
	export CFLAGS="-I${SOURCE}/include"
	./configure --prefix=${SOURCE} --enable-gpl --enable-pthreads --disable-ffplay --disable-ffserver --enable-libvpx --disable-decoder=libvpx --enable-libmp3lame --enable-libtheora --enable-libvorbis --enable-libx264 --enable-libxvid --enable-avfilter  --enable-filters --arch=x86 --enable-runtime-cpudetect && make -j 4 && make install 

> After doing all the hardcore compiling you are awarded a FFmpeg binary in the `sw/BIN` folder. [Here](https://github.com/ljf1239848066/FFmpeg/tree/master/bin) is my binary.



## Test：

	$> mkdir frames
	$> ./ffmpeg -i input.mp4 -r 10 frames/frame%03d.png
	$> ./ffmpeg -t 25 -ss 00:00:01 -i loading.mp4 loading.gif

>  Copy the `ffmpeg` binary to `/usr/local/bin` folder so that you can use `ffmpeg` command anywhere in your terminal directly.



### Convert mp4 to gif

	ffmpeg -i loading.mp4 -s loading.gif
	ffmpeg -t 25 -ss 00:00:01 -i loading.mp4 -s 540x960 loading.gif
-s：adjust gif size



	ffmpeg -t 25 -ss 00:00:01 -r 15 -i loading1.mp4 -s 540x960 loading2.gif
-r：adjust frame rate



	ffmpeg -i loading.mp4 -i logo.png -filter_complex 'overlay=10:main_h-overlay_h-10' loading1.mp4
Add a logo to the lower left corner

More ffmpeg param at [FFmpeg Filters Documentation](https://ffmpeg.org/ffmpeg-filters.html)


## Three Steps For Vedio Clip

### 1、Convert mp4 to png

	ffmpeg -i input.mp4 -r 10 frames/frame%03d.png
	
### 2、Remove unnecessary files，rename files according to the serial number.

	i=0;dir=$(eval pwd); for name in $(ls $dir);do i=$(($i+1));p=$(eval printf "%03d" $i);mv $name frame$p.png; done
	
### 3、Convert png to mp4

	ffmpeg -r 10 -f image2 -i frame%03d.png -pix_fmt yuv420p loading.mp4

