
---
title: Peer-to-peer or Channel Messaging
description: cpp
platform: Windows CPP
updatedAt: Mon Sep 30 2019 13:06:17 GMT+0800 (CST)
---
# Peer-to-peer or Channel Messaging
You can use this guide to quickly start messaging with the [Agora RTM C++ SDK for Windows](https://docs.agora.io/en/Real-time-Messaging/downloads). 

## Try the demo

We provide an open-source demo project on GitHub, [Agora-RTM-Tutorial-Windows](https://github.com/AgoraIO/RTM/tree/master/Agora-RTM-Tutorial-Windows), which implements an elementary messaging system. You can try this demo out and view our source code.

## Prerequisites

- Microsoft Visual Studio 2017 or later
- A Windows device running Windows 7 or later
- A valid [Agora developer account](https://sso.agora.io/login/).


<div class="alert note">Open the ports specified in <a href="https://docs.agora.io/cn/Agora%20Platform/firewall?platform=All%20Platforms">Firewall Requirements</a> if your network has a firewall.</div> 



<a name="setup"></a>

## Set up the development environment

We will walk you through the following steps in this section:

- [Get an App ID](#appid)
- [Create a project](#create)
- [Integrate the SDK into your project](#sdk)


### <a name="appid"></a>Get an App ID

You can skip to [Create a project](#create) if you already have an App ID. 

<details>
	<summary><font color="#3ab7f8">Get an App ID</font></summary>
	
1. Sign up for a developer account at [Agora Dashboard](https://dashboard.agora.io/). See [Sign in and Sign up](../../en/Real-time-Messaging/sign_in_and_sign_up.md).

2. Click **Get Started** under **Projects**.

	![](https://web-cdn.agora.io/docs-files/1563523371446)

3. Input your project name in the pop-up window and click **Create**. Follow the on-screen instructions to get to know the basic steps to start a video call. Once the project is created, you can find it under **Projects**.

	![](https://web-cdn.agora.io/docs-files/1563523478084)
	
4. Click the **Edit** button behind the new project, or the **Project Management** button ![](https://web-cdn.agora.io/docs-files/1551254998344) in the left navigation menu to go to the **Project Management** page.

 ![](https://web-cdn.agora.io/docs-files/1563523678240)

5. On the **Project Management** panel, find the **App ID** of your project.

 ![](https://web-cdn.agora.io/docs-files/1563523737158)
</details>

### <a name="create"></a> Create a project

Now, let's build a project from scratch. Skip to [Integrate the SDK](#sdk) if a project already exists.

<details>
	<summary><font color="#3ab7f8">Create a Windows project</font></summary>
	
1. Open <b>Microsoft Visual Studio</b> and click <b>Create new project</b>.
2. On the <b>New Project</b> panel, choose <b>MFC Application</b> as the project type, input the project name, choose the project location, and click <b>OK</b>.
3. On the <b>MFC Application</b> panel, choose <b>Application type > Dialog based</b>, and click <b>Finish</b>.
	
</details>


### <a name="sdk"></a> Integrate the SDK

Follow these steps to integrate the Agora RTM C++ SDK for Windows into your project.

**1. Configure the project files**

- Go to [SDK Downloads](https://docs.agora.io/en/Agora%20Platform/downloads), download the latest version of the Agora SDK for Windows, and unzip the downloaded SDK package.

- Copy the **sdk** folder of the downloaded SDK package to your project files.

**2. Configure the project properties**

Right-click the project name in the **Solution Explorer** window, click **Properties** to configure the following project properties, and click **OK**.

- Go to the **C/C++ > General > Additional Include Directories** menu, click **Edit**, and input **$(SolutionDir)include** in the pop-up window.

- Go to the **Linker > General > Additional Library Directories** menu, click **Edit**, and input **$(SolutionDir)** in the pop-up window.

- Go to the **Linker > Input > Additional Dependencies** menu, click **Edit**, and input **agora_rtc_sdk.lib** in the pop-up window.


## Implement peer-to-peer and channel messaging

This section provides sample codes and considerations related to peer-to-peer messaging and channel messaging. 



### Create and Initialize an Agora RTM Client


```cpp
#include "IAgoraService.h" 
#include "IAgoraRtmService.h"

std::unique_ptr<agora::base::IAgoraService> agoraInstance_;
agora::base::AgoraServiceContext context_;
std::unique_ptr<agora::rtm::IRtmServiceEventHandler> RtmEventCallback_;
std::unique_ptr<agora::rtm::IChannelEventHandler> channelEventCallback_;
std::shared_ptr<agora::rtm::IRtmService> rtmInstance_;
std::shared_ptr<agora::rtm::IChannel> channelHandler_;

void Init() {
  agoraInstance_.reset(createAgoraService());
  if (!agoraInstance_) {
    cout << "core service created failure!" << endl;
    exit(0);
  }

  if (agoraInstance_->initialize(context_)) {
    cout << "core service initialize failure!" << endl;
    exit(0);
  }

  RtmEventCallback_.reset(new RtmEventHandler());
  agora::rtm::IRtmService* p_rs = agoraInstance_->createRtmService();
  rtmInstance_.reset(p_rs, [](agora::rtm::IRtmService* p) {
  p->release();
  });

  if (!rtmInstance_) {
    cout << "rtm service created failure!" << endl;
    exit(0);
  }

  if (rtmInstance_->initialize(APP_ID.c_str(), RtmEventCallback_.get())) {
    cout << "rtm service initialize failure! appid invalid?" << endl;
    exit(0);
  }
}
```

### Log in and log out of the Agora RTM system

Only when you successfully log in the Agora RTM system, can you use most of the core features provided by the Agora RTM SDK. 

```cpp
bool login(const std::string& token, const std::string& uid) {
  if (rtmInstance_->login(token.c_str(), uid.c_str())) {
    cout << "login failed!" << endl;
    return false;
  }
  return true;
}
```

To log out of the system:

```cpp
void logout() {
  rtmInstance_->logout();
}
```

### Peer-to-peer messaging

#### Send a peer-to-peer message

To send a peer-to-peer message to a specified user

```cpp
void sendMessageToPeer(std::string peerID, std::string msg) {
  agora::rtm::IMessage* rtmMessage = rtmInstance_->createMessage();
  rtmMessage->setText(msg.c_str());
  int ret = rtmInstance_->sendMessageToPeer(peerID.c_str(),
                                  rtmMessage);
  rtmMessage->release();
  if (ret) {
    cout << "send message to peer failed! return code: " << ret
         << endl;
  }
}
```



### Channel messaging

Ensure that you have logged in the Agora RTM system before being able to use the channel messaging function. 

#### Create and join a channel

When creating a channel, you need to pass a channel ID, a string that must not be empty, null, "null", or exceed 64 Bytes in length. You also need to specify a channel listener: 

```cpp
void joinChannel(const std::string& channel) {
  string msg;
  channelEventCallback_.reset(new ChannelEventHandler(channel));
  channelHandler_.reset(rtmInstance_->createChannel(channel.c_str(), channelEventCallback_.get()))
  if (!channelHandler_) {
    cout << "create channel failed!" << endl;
  }
  channelHandler_->join();
}
```

```java
try {
		mRtmChannel = mRtmClient.createChannel("demoChannelId", mRtmChannelListener);
} catch (RuntimeException e) {
		Log.e(TAG, "Fails to create channel. Maybe the channel ID is invalid," +
						" or already in use. See the API Reference for more information.");
}

		mRtmChannel.join(new ResultCallback<Void>() {
				@Override
				public void onSuccess(Void responseInfo) {
						Log.d(TAG, "Successfully joins the channel!");
				}

				@Override
				public void onFailure(ErrorInfo errorInfo) {
						Log.d(TAG, "join channel failure! errorCode = "
																+ errorInfo.getErrorCode());
				}
		});
```

#### Channel messaging

```cpp
void sendGroupMessage (const std::string& msg) {
  agora::rtm::IMessage* rtmMessage = rtmInstance_->createMessage();
  rtmMessage->setText(msg.c_str());
  channelHandler_->sendMessage(rtmMessage);
  rtmMessage->release();
}
```

#### Leave a channel

You can call the `leave()` method to leave a channel. 

## Considerations

- You can create and join a maximum of 20 `RtmChannel` instances. But ensure that each channel has a differnt channel ID and a different listener. 
- If the channel ID you put is invalid, or if an `RtmChannel` with the same channel ID already exists, the SDK returns `null`. 
- You cannot reuse a received `RtmMessage` instance. 
- When you no longer need an `RtmChannel` instance after leaving it, you can call its `release()` method to release all resources it is using. 

