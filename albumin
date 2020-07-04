#!/bin/sh

# command line parameters
image=$1

# ignore null regex matches
shopt -s nullglob


if [[ "$image" != "-times" ]]
then
	# render video
	: > __tracks
	for song in *.{flac,wav,ogg,mp3}
	do
		processedFilename=$(echo "__processed_$song.wav" | tr ' ' '_')
		ffmpeg -i "$song" "$processedFilename"
		echo "file '$processedFilename'" >> __tracks
	done
	ffmpeg -f concat -i __tracks -c copy __combined.wav
	ffmpeg -loop 1 -r 1 \
		-i $image -i __combined.wav \
		-c:v libx264 -crf 32 -c:a aac -b:a 192k -shortest \
		album.mp4
	rm __combined.wav
	rm __processed*
	rm __tracks
fi

# show timestamps with song names (for pasting in description or comment)
echo ''
echo ''
echo 'TIMESTAMPS'
currentMinutes=0
currentSeconds=0
currentTime=00:00
for song in *.{flac,wav,ogg,mp3}
do
	# get all of the audio file information
	# extract duration
	# format will be "Duration xx:xx:xx.xx,", ignore "Duration"
	# remove comma at end
	# replace timestamp colons with spaces
	songTime=$(ffmpeg -i "$song" 2>&1  \
	| grep Duration \
	| awk '{ print $2 }' \
	| sed 's/,//g' \
	| tr ':' ' ' )

	# ignore empty output (this will happen if one of the filetypes above isn't matched)
	[[ -z $songTime ]] && continue

	# output current song and its timestamp
	echo $song - $currentTime 

	# calculate timestamp for next song 
	songMinutes=$(echo $songTime | cut -f2 -d ' ' | cut -c1-2 | sed 's/^0*//')
	songSeconds=$(echo $songTime | cut -f3 -d ' ' | cut -c1-2 | sed 's/^0*//')

	newSeconds=$((currentSeconds + songSeconds))
	if [ $newSeconds -gt 59 ]; 
	then
		((newSeconds%=60))
		((currentMinutes++))
	fi
	currentSeconds=$newSeconds
	currentMinutes=$((currentMinutes + songMinutes))
	currentTime=$(printf "%02d:%02d\n" $currentMinutes $currentSeconds)
done
