---
title: "Linphone通话录音功能实现"
date: 2017-06-26 15:17:58
tags: ["LinPhone"]
categories: ["iOS","Objective-C"]
---

最近使用[`Linphone`][1]要实现一个通话录音的功能，`Linphone`倒是给了相关的方法，不过组合在一起还不知道怎么用。好在有安卓方面成功的代码：
```java
LinphoneCall call = linphoneCore.getCurrentCall();
LinphoneCallParams params = call.getCurrentParamsCopy();
params.setRecordFile(recordingPath);
linphoneCore.updateCall(myCall, params);
call.startRecording();
```
然后照着这部分代码试着写了一个`iOS`版本的:
```objc
LinphoneCall *call = linphone_core_get_current_call(LC);
LinphoneCallParams *params = linphone_call_params_copy(linphoen_call_get_current_params(call));
const char *file_path = [recordingPath cStringUsingEncoding:[NSString defaultCStringEncoding]];
linphone_call_params_set_record_file(params, file_path);
linphone_core_update_call(LC, call, params);
linphone_call_params_destroy(params);
```
这样算是设置好了录音文件的路径，接下来只需要调用对应的方法就开始录音了：
```objc
linphone_call_start_recording(call);
```
结束录音的时候，尝试看看能不能获取到对应的路径：
```objc
linphone_call_stop_recording(call);
const LinphoneCallParams *params = linphone_call_get_current_params(call);
const char *record_file = linphone_call_params_get_record_file(params);
if (record_file) {

}
```
结果却是这里的路径总是为`NULL`，看样子这样的写法是不成功了。

查了相关资料，看了那些方法的注释，才发现要在通话开始之前就要设置好录音文件的路径。
如果是在打电话的过程中，则需要在拨打之前进行设置：
```objc
LinphoneAddress *address = linphone_address_new(linphone_core_get_identity(LC));
LinphoneCallParams *params = linphone_core_create_call_params(LC, NULL);
linphone_call_params_set_record_file(params, file_path);
linphone_core_invite_address_with_params(LC, address, params);
linphone_call_params_destroy(params);
```
如果是在接听电话的时候需要录音，那则在收到对应的状态时就要设置：
```objc
if (state == LinphoneCallIncomingReceived) {
    LinphoneCallParams *params = linphone_core_create_call_params(LC, NULL);
    linphone_call_params_set_record_file(params, file_path);
    linphone_core_accept_call_with_params(LC, call, params);
    linphone_call_params_destroy(params);
}
```
接下来在通话的过程中使用`linphone_call_start_recording`开始录音，使用`linphone_call_stop_recording`方法结束录音。然后调用`linphone_call_params_get_record_file`就可以获取到对应的录音文件路径了。

[1]:http://www.linphone.org