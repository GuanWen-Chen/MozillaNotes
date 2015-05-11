#ScreenReader
##Outline
* Introduction
* Screen Reader Architecture
  * Core Modules and Components
* Accessibility
* Speech Synthesis

##Introduction
###What is screen reader
**Web Accessibility Initiative-Accessible Reich Internet Application(WAI-ARIA)** is a w3c recommendation that can gain 
the accessibility to the disabilities. And screen reader is a text-to-voice software accessibility tool for mozilla.
###How to enable screen reader on B2G?

Open "Settings" >> choose "Developers" >> check "Show screen reader settings" under "debug"<br/>
Back to "Settings" mune >> choose "Accessibility" >> Open "Screen Reader"

##Screen Reader Architecture
  There are two parts of screen reader, the first part is AccessFu.jsm which is imported by shell.html. It will listen to mozBroser-opened message and load content-script.js into the browser elements when are opened. In the other hands, it is also in charge of handling the guesture, drawing highlight box and trigger the TTS(text-to-speech) engine. The other part is for the browser element which contains content-script.js, ContentControl.jsm and EventManager.jsm, the detail of these file will be mentioned after. Every browser element has an accessibility tree and an accessible pivot, the tree is a subtree of a DOM tree and the pivot will traverse in the tree to indicate the current focused element in screen reader.<br>
###Move the cursor
The following is the flow when an user action occur, this action can be input like gesture, mouse scroll, key event:<br/><br/>
  ![Code flow](./img/codeFlow.png)<br/>
  Most of user actions like gesture and frame load will start from AccessFu.jsm. It will send the action message to the target browser element. And ContentControl.jsm will access the accessibility tree via nsAccessiblePivot which will send an AccEvent for pivot updating to EventManager.jsm. At last EventManager.jsm will wrap the update information and send it back to AccessFu.jsm for TTS and highlightbox updating.<br>
###Out-of-Process movement
  When cursor move across browser element, it will connect with the other browser element with frame message manager. ContentControl.jsm will manager this part. The following figure demostrate the flow:<br>
  ![Code flow2](./img/codeFlow2.png)<br>
  When ContentControl.jsm first receive action message from its parent Browser element, it will check if the current accessible element its pivot points to embeds another Browser element, if so pass it recursively to the deepest child. Then check the accessiblility tree like previous mentioned if it is moveable. If it is moveable, still, check if the pointing element a Browser element, if so do the check we previously did. if it is not movable, pass it back to the parent and check it is moveable like previous check, if it can move clear the previous cursor.

###Core Modules and Components
####AccessFu.jsm
  AccessFu.jsm is imported by settings.js in shell.html. It is the mainly controller of screen reader. When a frame is loaded to the system, AccessFu.jsm will injects it with a script "content-script.js ", which will initializes ContentControl.jsm and EventManager.jsm. Also, AccessFu.jsm captures user actions including gestures, key inputs and transfers them to action commands then sends it to target frame for changing the visual corsur. Moreover, it listens to visual cursor updating message from content and rerenders the visual cursor and send the chrome message to system app for the voice reading.
####ContentControl.jsm
  ContentControl.jsm handles the action commands which will change nsIAccessiblePivot's target element, gets the virtual pivot interface nsIAccessiblePivot via Utils.jsm and makes it executes corresponding action.
####Utils.jsm
  Utils.jsm supports screen reader javascrupt modules the entry points to the commponent interfaces like Accessibility elements, Message Managers and Window elements.
####nsIAccessiblePivot
  Every nsIDocAccessible contains a nsIAccessiblePivot pointing to an Accessible element which indicates targeted element in this Accessibility tree. nsIAccessiblePivot can traverse Accessibility tree by its member functions(e.g, moveNext/movePrev/moveToPoint). After its traversal functions, it will fire an AccEvent for announcing status updating.
####EventManager.jsm
  EventManager.jsm listens to any event that will update the visual cursor's position like AccEvent for nsIAccessiblePivot position updating or the veiwport change event like scroll, wheel and resize.
####Presentation.jsm
  Presentation.jsm is an interface for all presenter classes. A presenter could be, for example, a speech output module, or a visual cursor indicator.

##Accessibility
  Accessibility is to providing people an easier way to access web. Check details in [Accessibility](https://developer.mozilla.org/en-US/docs/Web/Accessibility).
##Speech Synthesis
  

