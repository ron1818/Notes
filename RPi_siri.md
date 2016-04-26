RPi as Siri
===========
1. speech to text
2. query processing
3. text to speech
4. Putting Them Together

Speech to text 
--------------
###install ffmpeg###

> sudo apt-get install ffmpeg

###install goolge speech API###
*speech2text.sh*
```bash
#!/bin/bash

echo “Recording… Press Ctrl+C to Stop.”
arecord -D “plughw:1,0” -q -f cd -t wav | ffmpeg -loglevel panic -y -i – -ar 16000 -acodec flac file.flac > /dev/null 2>&1

echo “Processing…”
wget -q -U “Mozilla/5.0” –post-file file.flac –header “Content-Type: audio/x-flac; rate=16000” -O – “http://www.google.com/speech-api/v1/recognize?lang=en-us&client=chromium” | cut -d” -f12 >stt.txt

echo -n “You Said: ”
cat stt.txt

rm file.flac > /dev/null 2>&1
```

referred from: https://www.raspberrypi.org/forums/viewtopic.php?uid=19416&f=38&t=27290&start=0
```bash
#!/bin/bash
arecord -D "plughw:1,0" -q -f cd -t wav | ffmpeg -y -i - -ar 16000 -acodec flac file.flac
wget -q -U "Mozilla/5.0" --post-file file.flac --header "Content-Type: audio/x-flac; rate=16000" -O - "http://www.google.com/speech-api/v1/recognize?lang=en-us&client=chromium" | cut -d\" -f12
rm file.flac
```

###execute###
> chmod +x speech2text.sh
> ./speech2text.sh

Query Processing
----------------
###Install Wolfram Alpha###

python Wolfram Alpha library can do the job

> apt-get install python-setuptools easy_install pip
> sudo python setup.py build
> sudo python setup.py

must signup [online](http://products.wolframalpha.com/api/) for an ID

You should now be signed in to the Wolfram Alpha Developer Portal and, 
on the My Apps tab, click the “Get an AppID” button and fill out the "Get a New AppID" form. 
Use any Application name and description you like. Click the "Get AppID" button.

###Wolfram Alpha Python Interface###

*queryprocess.py*

```python
#!/usr/bin/python

import wolframalpha
import sys

# Get a free API key here http://products.wolframalpha.com/api/
# This is a fake ID, go and get your own, instructions on my blog.
app_id=’HYO4TL-A9QOUALOPX’

client = wolframalpha.Client(app_id)

query = ‘ ‘.join(sys.argv[1:])
res = client.query(query)

if len(res.pods) > 0:
  texts = “”
  pod = res.pods[1]
  if pod.text:
    texts = pod.text
  else:
    texts = “I have no answer for that”
  # to skip ascii character in case of error
  texts = texts.encode(‘ascii’, ‘ignore’)
  print texts
else:
  print “Sorry, I am not sure.”
```

> chmod +x ./queryprocess.py
e.g.: 
> ./queryprocess.py "what time is it"

Text to Speech
--------------
###audio playback###

> sudo apt-get install mplayer
> sudo nano /etc/mplayer/mplayer.conf
> nolirc=yes

###text to speech###
*text2speech.sh*
```bash
#!/bin/bash
say() { local IFS=+;/usr/bin/mplayer -ao alsa -really-quiet -noconsolecontrols “http://translate.google.com/translate_tts?tl=en&q=$*”; }
say $*
```

> chmod +x text2speech.sh

e.g.:
> ./text2speech.sh "My name is Oscar and I am testing the audio."

###Google Text To Speech Text Length Limitation###

Although it’s very kind of Google sharing this great service, there is a limit on the length of the message. 
I think it’s around 100 characters.

To work around this, here is an upgraded bash script that breaks up the text into multiple parts so 
each part is no longer than 100 characters, and each parts can be played successfully. 
I modified the original script is from here to fit into our application.

*text2speech_enhanced.sh*

```bash
#!/bin/bash

INPUT=$*
STRINGNUM=0
ary=($INPUT)
for key in “${!ary[@]}”
do
SHORTTMP[$STRINGNUM]=”${SHORTTMP[$STRINGNUM]} ${ary[$key]}”
LENGTH=$(echo ${#SHORTTMP[$STRINGNUM]})

if [[ “$LENGTH” -lt “100” ]]; then

SHORT[$STRINGNUM]=${SHORTTMP[$STRINGNUM]}
else
STRINGNUM=$(($STRINGNUM+1))
SHORTTMP[$STRINGNUM]=”${ary[$key]}”
SHORT[$STRINGNUM]=”${ary[$key]}”
fi
done
for key in “${!SHORT[@]}”
do
say() { local IFS=+;/usr/bin/mplayer -ao alsa -really-quiet -noconsolecontrols “http://translate.google.com/translate_tts?tl=en&q=${SHORT[$key]}”; }
say $*
done
```

Toplevel
--------

###toplevel script###
*main.sh*
```bash
#!/bin/bash

echo “Recording… Press Ctrl+C to Stop.”

./speech2text.sh

QUESTION=$(cat stt.txt)
echo “Me: “, $QUESTION

ANSWER=$(python queryprocess.py $QUESTION)
echo “Robot: “, $ANSWER

./text2speech.sh $ANSWER
```

Reference
---------
https://oscarliang.com/raspberry-pi-voice-recognition-works-like-siri/

https://www.raspberrypi.org/forums/viewtopic.php?uid=19416&f=38&t=27290&start=0
