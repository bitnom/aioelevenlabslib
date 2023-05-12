# Usage examples

## Play back, save, rate and delete generated audio

```python
from elevenlabslib.helpers import *
from elevenlabslib import *

api_key = "INSERT KEY HERE"
user = ElevenLabsUser(api_key)
premadeVoice = user.get_voices_by_name("Rachel")[0]

#Generate the audio and get the bytes and historyID. Setting stability here only overrides it for this generation.
generationData = premadeVoice.generate_play_audio("This is a test.", stability=0.4, playInBackground=False)

#Save them to disk, in ogg format (can be any format supported by SoundFile)
save_audio_bytes(generationData[0], "testAudio.ogg", outputFormat="ogg")

#Fetch the corresponding historyItem
historyItem = user.get_history_item(generationData[1])

#Rate it
historyItem.edit_feedback(thumbsUp=True,feedbackText="This text to speech service works very well!")

#Delete it
historyItem.delete()
```

## Create a voice using Voice Design
```python
from elevenlabslib.helpers import *
from elevenlabslib import *

api_key = "INSERT KEY HERE"
newVoiceName = "newVoice"
user = ElevenLabsUser(api_key)

try:
    #Generate the audio and get the temporary voiceID.
    temporaryVoiceID, generatedAudio = user.design_voice(gender="female", accent="american", age="young", accent_strength=1.0)
    
    #Play back the generated audio.
    play_audio_bytes(generatedAudio, playInBackground=False)
    
    #Add the voice to the account.
    newGeneratedVoice = user.save_designed_voice(temporaryVoiceID, newVoiceName)
    
except requests.exceptions.RequestException:
    print("Couldn't design voice, likely out of tokens or slots.")

```


## Create and edit a cloned voice

```python
from elevenlabslib import *

api_key = "INSERT KEY HERE"
newVoiceName = "newVoice"
user = ElevenLabsUser(api_key)

try:
    existingVoice = user.get_voices_by_name(newVoiceName)[0]
except IndexError:
    print("Voice doesn't exist, let's create it")
    if not user.get_voice_clone_available():
        print("Sorry, your subscription doesn't allow you to use voice cloning.")
    else:
        #Load a sample from a filepath and use it to create the new voice.
        firstSample = r"X:\sample1.mp3"
        newClonedVoice = user.clone_voice_by_path(newVoiceName, firstSample)
        print("New voice:")
        print(newClonedVoice.get_name())
        
        #Add a sample by loading it as bytes.
        secondSample = open(r"X:\sample2.mp3", "rb").read()
        newClonedVoice.add_samples_bytes({
            "sample2.mp3": secondSample
        })
```

## Play back and delete a history item

```python
from elevenlabslib import *
from elevenlabslib.helpers import *

api_key = "INSERT KEY HERE"
user = ElevenLabsUser(api_key)

#Generate two items to be deleted later
premadeVoice = user.get_voices_by_name("Rachel")[0]
premadeVoice.generate_audio("Test.")
premadeVoice.generate_audio("Test.")

#Find them, download them then delete them.
testItems = list()
for historyItem in user.get_history_items():
    if historyItem.text == "Test.":
        testItems.append(historyItem)

#Download them
downloadedItems = user.download_history_items(testItems)

#Play them back
for historyID, audioData in downloadedItems.items():
    play_audio_bytes(audioData, playInBackground=False)

#Delete them
for item in testItems:
    item.delete()
```

## Use the multilingual TTS model

```python
from elevenlabslib import *

api_key = "INSERT KEY HERE"
user = ElevenLabsUser(api_key)
premadeVoice = user.get_voices_by_name("Rachel")[0]

#Print the models available to the user (you'll need the model_id for the multilingual one)
print(user.get_available_models())

premadeVoice.generate_and_play_audio("Questa è una prova!", playInBackground=False, model_id="eleven_multilingual_v1")
```