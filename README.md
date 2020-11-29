Audio Source
===

Extract audio resource ID
===
After upload and processing of audio files you should get UUID of them
```bash
curl  -H "Authorization: OAuth $DIALOG_TOKEN" \
   "https://dialogs.yandex.net/api/v1/skills/${SKILL_ID}/sounds" > sounds.json
cat sounds.json | jq '[ .sounds | sort_by(.originalName) | .[] | { (.originalName[20:22]) : .id} ]' | fgrep \"
``` 

Data Preparation
===
Marking data partially made via ffmpeg and service and [STT service](https://cloud.yandex.ru/docs/speechkit/stt/request).

Something like
```bash
# convert first 7 sec to OGG format
for f in *.mp3; 
do 
    ffmpeg -i $f -c:a libvorbis -q:a 4 -to 7  "${f/%mp3/ogg}"; 
done

# output file ID and result of TTS
for f in *.ogg; 
do 
    foo=${f##Brilliant_RS_1_1502_}; 
    echo ${foo%%.ogg} -- $(curl -X POST -s \
     -H "Authorization: Bearer ${IAM_TOKEN}" \
     -H "Transfer-Encoding: chunked"  \
     --data-binary "@${f}" \
     "https://stt.api.cloud.yandex.net/speech/v1/stt:recognize?topic=general&folderId=${FOLDER_ID}") ; 
done
```
