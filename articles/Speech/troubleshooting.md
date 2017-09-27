
---
title: Troubleshooting | Microsoft Docs
description: How to resolve issues when using Microsoft Speech Service.
services: cognitive-services
author: zhouwangzw
manager: wolfma61

ms.service: cognitive-services
ms.technology: speech
ms.topic: article
ms.date: 09/15/2017
ms.author: zhouwang
---

# Troubleshooting

This page collects some of the most common pitfalls/errors users encounter, and helps you trobuleshoot these issues.

## I got HTTP error of "403 Forbidden".

When using Speech API, it returns a HTTP "403 Forbidded" error. 

### Cause

This is often caused by authentication issues. Connection requests without valid `Ocp-Apim-Subscription-Key` or `Authorization` header are rejected by the service with a 403 Forbidden HTTP response.

If you are using subscription key for authentication, the reason could be
- the subscription key is missing or invalid, or
- the subscription key exceeds usage quota, or
- the `Ocp-Apim-Subscription-Key` field is not set in the request header, when REST API is called.

If you are using authorization token for authentication, the following reasons could cause the error.
- the `Authorization` header is missing in the request when using REST, or
- the authorization token specified in the Authorization header is invalid, or
- the authorization token is expired. The access token has an expiry of 10 minutes.

For more information on authentication, refer to the [Authentication](How-to/how-to-authentication.md) page.

### Troubleshooting steps
1. Verfiy that your subscription key is valid. You can run the following command for verfication. Note to replace *YOUR_SUBSCRIPTION_KEY* with your own subscription key. If your subscription is valid, you will receive in the response the authorization token as a JSON Web Token (JWT). Otherwise you will get an error in response.

> [!NOTE]
> Replace `YOUR_SUBSCRIPTION_KEY` with your own subscription key.

# [Powershell](#tab/Powershell)

```Powershell
$FetchTokenHeader = @{
  'Content-type'='application/x-www-form-urlencoded';
  'Content-Length'= '0';
  'Ocp-Apim-Subscription-Key' = 'YOUR_SUBSCRIPTION_KEY'
}

$OAuthToken = Invoke-RestMethod -Method POST -Uri https://api.cognitive.microsoft.com/sts/v1.0/issueToken -Headers $FetchTokenHeader

# show the token received
$OAuthToken

```

# [curl](#tab/curl)

The example uses curl on Linux with bash. You may need to install curl, if it is not available on your platform. The example should work on Cygwin on Windows, Git Bash, zsh, and other shells too.

```
curl -v -X POST "https://api.cognitive.microsoft.com/sts/v1.0/issueToken" -H "Content-type: application/x-www-form-urlencoded" -H "Content-Length: 0" -H "Ocp-Apim-Subscription-Key: YOUR_SUBSCRIPTION_KEY"
```
---

2. Make sure that you use the same subscription key in your application or in the REST reqeust as that is used in step 1.

3. If you use authorization token for authentication, verfiy that the token is still valid. Run the following command to make a POST request to the service, and expect a response message from the service. If you still receive HTTP 403 error, please double-check the access token is set correctly and not expired.

> [!IMPORTANT]
> The token has an expiry of 10 minutes.

> [!NOTE]
> Replace `YOUR_AUDIO_FILE` with the path to your prerecorded audio file, and `YOUR_ACCESS_TOKEN` with the authorization token returned in the previous step.

# [Powershell](#tab/Powershell)

```Powershell

$SpeechServiceURI =
'https://speech.platform.bing.com/speech/recognition/interactive/cognitiveservices/v1?language=en-us&format=detailed'

# $OAuthToken is the authrization token returned by the token service.
$RecoRequestHeader = @{
  'Authorization' = 'Bearer '+ $OAuthToken;
  'Transfer-Encoding' = 'chunked'
  'Content-type' = 'audio/wav; codec=audio/pcm; samplerate=16000'
}

# Read audio into byte array
$audioBytes = [System.IO.File]::ReadAllBytes("YOUR_AUDIO_FILE")

$RecoResponse = Invoke-RestMethod -Method POST -Uri $SpeechServiceURI -Headers $RecoRequestHeader -Body $audioBytes

# Show the result
$RecoResponse

```

# [curl](#tab/curl)

```
curl -v -X POST "https://speech.platform.bing.com/speech/recognition/interactive/cognitiveservices/v1?language=en-us&format=detailed" -H "Transfer-Encoding: chunked" -H "Authorization: Bearer YOUR_ACCESS_TOKEN" -H "Content-type: audio/wav; codec=audio/pcm; samplerate=16000" --data-binary @YOUR_AUDIO_FILE
```

---

## I got HTTP error of "400 Bad Reqeust".

This is usually because that the request body contains invalid audio data. Currently we only support WAV file.

## I got HTTP error of "408 Request Timeout".

The most common reason is that no audio data is sent to the service, and the service returns this error after timeout. For REST API, the audio data should be put in the request body.

## The RecognitionStatus in the response is `InitialSilenceTimeout`.

This is usually caused by issues in audio data, such as
- the audio has a long sileince time at the begining. The service will stop the recognition after some number of seconds and returns `InitialSilenceTimeout`.
- the audio uses unsupported codec format, which makes the audio data be treated as slience.


