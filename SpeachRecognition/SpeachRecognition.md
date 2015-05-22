#SpeechRecognition
##What is Speech Recognition?
  Speech recogntion is a functionality that let system can recognize what user says and transfer it to text which is known as Automated Speech Recognition (ASR). <br> The specification is proposed at (https://dvcs.w3.org/hg/speech-api/raw-file/tip/speechapi.html#dfn-continuous)

##Sample code
The following is a simple example of SpeachRecogniton usage
```
<script type="text/javascript">
  var recognition = new SpeechRecognition();
  recognition.onresult = function(event){
    textInput.value = event.result[0][0].transcript
  }
</script>

<input id="textInput">
<input type="button" value="Speak" onclick="recognition.start()">
```
##Code Architecture
###SpeechRecognition
SpeachRecognition receives audio data from media stream and maintain a state machine for desiding how to deal with incoming and sending response message to content App.<br>
Create a nsDOMUserMediaStream -> OnSuccess -> add SpeechStreamListener -> NotifyQueuedTrackChanges <br>
####Audio data
When starting SpeechRecognition, it will get a media stream via MediaManager::GetUserMedia. A nsDOMUserMediaStream will be passed back through Speechrecognition::GetUserMediaSuceesCallBack::OnSuccess. SpeechRecognition will then add SpeechStreamListener to the stream. After that we will get audio data via NotifyQueuedChunkChanged and notify the SpeechRecognition event about the data.
####State machine
The following is the state machine flow, there are two factors to decide the transition between the states, one is the SpeechEvent, the other is state of Endpointer. <br>
![state machine](./img/stateMachine.png)
The state machine has six states, IDLE, STARTING, ESTIMATING, WAITING_FOR_SPEECH, RECOGNITION and WAITING_FOR_RESULT. The state usually stay in IDLE, when the start method is called the state will set to STARTING and it is the time SpeechRecognition ready to receive audio data. When the first audio data arrive, the state will transfer to ESTIMATING, in this state, system collect audio_data as background noise and change to WAIT_FOR_SPEECH when get enough data. WAITING_FOR_SPEECH detect if user start to talk depending on the incoming audio data, once the system detect the user start to speech, it turns into RECOGNIZING and continues to receive audio data until this speech is end and go to the final state, WAITING_FOR_RESULT to wait the speech recognition service return final result. This is for a non-continuous case, continuous one may go through another flow but not implemented in gecko currently.<br>
###SpeechEvent
There are some event types to trigger state machine flows, EVENT_START, EVENT_STOP and EVENT_ABORT are fired when calling SpeechRecognition's web API and EVENT_AUDIO_DATA, EVENT_AUDIO_ERROR, EVENT_RECOGNITIONSERVICE_INTERMEDIATE_RESULT, EVENT_RECOGNITIONSERVICE_FINAL_RESULT and EVENT_RECOGNITIONSERVICE_ERROR are fired by SpeechRecognitionService.
###Endpointer, eneergy-endpointer
Endpointer and energy_endpointer is in charge of recognize time points of a user speech; energy_endpointer use RMS of audio data to define the stages of user speech and Endpointer defines the start and end point via stage of energy_endpointer supplies.<br>
There are two modes in energy_endpointer, EnvironmentEstimationMode and SetUserInputMode, in EnvironmentEsitmationMode, the incoming audio data will be consider as background noise and calculate the noise level via the signal's RMS. When it is changed to UserInputMode, the audio data's RMS should bigger than the noise level to be considered the user is speaking. There are four stages for user to complete a speech, energy_pointer will calculate the time of valid audio data in some time window to deside whether to go to next stage.<br>
![energy_endpointer states](./img/energy_endpointer.png)
When processing audio data Endpointer will check the state of energy_pointer. Two main time points are recorded for SpeechRecognition state machine transition. have_received_speech is set true when energy_endpointer'state go from EP_PRE_SPEECH to EP_POSSIBLE_ONSET and speech_input_complete is set true when after requested_silence_length after speech_end_time_us.<br>
