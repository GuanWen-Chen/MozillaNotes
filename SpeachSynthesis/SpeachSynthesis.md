#SpeachSynthesis
##Introduction
Speech synthesis is part of html5 speech API, it can do the text-to-speech work.
##WebAPI
The following a sample codes that we can use in web page.<br>
Example<br>
```javascript
    var txt = new SpeechSynthesisUtterance('Hello Mozilla');
    speechSynthesis.speak(txt);
```
To make speech synthesis API speeching the sentence or words you want it to, you have to wrap them as a SpeechSynthesisUtterance object, and pass it to speechSynthesis.speak() as parameter. There are more details in following sections.

###SpeechSynthesisUtterance
SpeechSynthesisUtterance is an interface that contain the information about how and what content the system should speech.<br>
```javascript
attribute DOMString text;  //The content system should speech
attribute DOMString lang; //The language system should choose
attribute SpeechSynthesisVoice? voice; //The voice system should use
attribute float volume; //The speech volume
attribute float rate;  //The speech rate
attribute float pitch;  //The speech pitch
```
###SpeechSynthesis
SpeechSynthesis is an interface of the speech system. developer can control it through following methods.<br>
```javascript
void speak(SpeechSynthesisUtterance utterance);  //Make the system speak the SpeechSynthesisUtterance you pass.
void cancel();  //Cancel the speak request and all the request in speak queue.
void pause();  //Pause the current speak task.
void resume();  //Resume the task from pause state.
```
And developer can use SpeechSynthesis.getVoices() to obtain available voices in system.<br>

##Components
There are four main components of speech synthesis in the gecko system: SpeechSynthesis, nsSynthVoiceRegistry and nsISpeechService.
###SpeechSynthesis
SpeechSynthesis is in charge of managing all the speech request. It puts receiving requests in a queue and pass the request to nsSynthVoiceRegistry when speech system available. Also it controls the requests execution with pause(), resume(), cancel() API from web API developers.
###nsSynthVoiceRegistry
Speech service will register their voice to nsSynthVoiceRegistry when they are started. And it will wrap request from SpeechSynthesis into a peech task and send it from child process to chrome process. It bind the task with mStream. It will find the best mVoice to task and send it to the corresponding speech service
###nsISpeechService
The actual component to do the text-to-speech, it will take initiate to register voice to nsSynthVoiceregistry. Two kinds of VoiceService <br> 
1. Indirect audio - the service is responsible for outputting audio. The service calls the nsISpeechTask.dispatch methods directly. Starting with dispatchStart() and ending with dispatchEnd or dispatchError().<br>
2. Direct audio - the service provides us with PCM-16 data, and we output it. The service does not call the dispatch task methods directly. Instead, audio information is provided at setup(), and audio data is sent with sendAudio(). The utterance is terminated with an empty sendAudio().
###How it works?
####Pico Service
When the Speech Service is inited, it will register its voice file to nsSynthVoiceRegistry, which store in mVoice<br>
GetInstanceForService[1]<br>
Create the "Pico Worker" thread for init Pico Service [2]<br>
Read the language file and register them to nsSynthVoiceRegistry [3] [4]<br>
<br>
[1] https://dxr.mozilla.org/mozilla-central/source/dom/media/webspeech/synth/pico/PicoModule.cpp#51<br>
[2] https://dxr.mozilla.org/mozilla-central/source/dom/media/webspeech/synth/pico/nsPicoService.cpp#464<br>
[3] https://dxr.mozilla.org/mozilla-central/source/dom/media/webspeech/synth/pico/nsPicoService.cpp#600<br>
[4] https://dxr.mozilla.org/mozilla-central/source/dom/media/webspeech/synth/pico/nsPicoService.cpp#501<br>
###nsSpeechTask
