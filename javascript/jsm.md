#jsm
##What is jsm
see : https://developer.mozilla.org/en-US/docs/Mozilla/JavaScript_code_modules/Using
##Cu.import
###XPCOMUtils:LazyModuleGetter
Load the module when needed. The advantage compare with Cu.import is that if you use Cu.import in the head of the code, it will cost a significant initial time. And LazyModuleGetter separate the loading time to the time you actually use the module. Which to use is depend on your code design.

####Code tracing
https://dxr.mozilla.org/mozilla-central/source/js/xpconnect/loader/XPCOMUtils.jsm#246
It call the 
https://dxr.mozilla.org/mozilla-central/source/js/xpconnect/loader/XPCOMUtils.jsm#187
which set the property "aName" to the object "aObject", so when the original code call aName.XXX, the call aName will execute the get function in https://dxr.mozilla.org/mozilla-central/source/js/xpconnect/loader/XPCOMUtils.jsm#187 and trigger the lambda function which call the Cu.import, and at last replace the aName''s property with the module we got.
