---
title: Using REST API for speech recognition  | Microsoft Docs
description: Using REST to access Microsoft Speech API in Microsoft Cognitive Services to convert spoken audio to text.
services: cognitive-services
author: zhouwang
manager: wolfma

ms.service: cognitive-services
ms.technology: speech
ms.topic: article
ms.date: 15/09/2017
ms.author: zhouwang
---

# Get Started with Speech Recognition using REST API

With Microsoft Speech API you can develop applications using REST API to convert spoken audio to text. This article will help you to do speech recognition via REST end point. 

## Prerequisites

#### Subscribe to Speech API and get a free trial subscription key
To access the REST end point, you must first subscribe to Speech API which is part of Microsoft Cognitive Services (previously Project Oxford). After subscribing, you will have the necessary subscription keys which are needed in the following operations. Both the primary and secondary keys can be used. For subscription and key management details, see [Subscriptions](https://azure.microsoft.com/en-us/try/cognitive-services/).

#### Precorded audio file
In this example, we use a recorded audio file to illustrate the usage of the REST API. Please record a short audio file of you saying something short (e.g.: *"What is the weather like today?"* or *"Find funny movies to watch."*). The Microsoft Speech API also supports exteranl microphone input. Please see the [sample applications](samples) for how to use microphone input.

> [!NOTE]
> The example requires that audio is recorded as wav file with **PCM single channel (mono), 16000Hz**.

# Getting started
To use Speech API REST end point, the steps are as follows:
1. Authenticate and get a JSON Web Token (JWT) from the token service.
2. Set the proper request header and send the request to Bing Speech API REST end point.
3. Parse the response to get your transcribed text.

The sections following will provide more details.

## Get authorization token
To access the REST endpoint, you need a valid OAuth token. To get this token, you must first have a subscription key from the Speech API, as described [here](GetStartedREST##Prerequisites). Then you can send a POST request to the token service with the subscription key, and will receive in the response the access token back as a JSON Web Token (JWT), which will be passed through in the Speech request header.

> [!NOTE]
> The token has an expiry of 10 minutes. Please see the Authentication(How-to/how-to-authentication) section for how to renew the token. 

The token service URI is located here:

```
https://api.cognitive.microsoft.com/sts/v1.0/issueToken
```

The code below is an example implementation in C# for how to handle authentication. Please replace *YOUR_SUBSCRIPTION_KEY* with your own subscription key.

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
# [cURL](#tab/cURL)
The examples below assume running curl on Linux using bash or Cygwin on Windows. You may need to install curl if it is not available on your platform. The examples should work in Git Bash, zsh, and other shells too.

```
    curl -v -X POST "https://api.cognitive.microsoft.com/sts/v1.0/issueToken" -H "Content-type: application/x-www-form-urlencoded" -H "Content-Length: 0" -H "Ocp-Apim-Subscription-Key: YOUR_SUBSCRIPTION_KEY"
```

# [C#](#tab/CSharp)
```cs
    /*
     * This class demonstrates how to get a valid O-auth token.
     */
    public class Authentication
    {
        public static readonly string FetchTokenUri = "https://api.cognitive.microsoft.com/sts/v1.0";
        private string subscriptionKey;
        private string token;

        public Authentication(string subscriptionKey)
        {
            this.subscriptionKey = subscriptionKey;
            this.token = FetchTokenAsync(FetchTokenUri, subscriptionKey).Result;
        }

        public string GetAccessToken()
        {
            return this.token;
        }

        private async Task<string> FetchTokenAsync(string fetchUri, string subscriptionKey)
        {
            using (var client = new HttpClient())
            {
                client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", subscriptionKey);
                UriBuilder uriBuilder = new UriBuilder(fetchUri);
                uriBuilder.Path += "/issueToken";

                var result = await client.PostAsync(uriBuilder.Uri.AbsoluteUri, null);
                Console.WriteLine("Token Uri: {0}", uriBuilder.Uri.AbsoluteUri);
                return await result.Content.ReadAsStringAsync();
            }
        }
    }
```

---

The POST request sent to the token service by the command above looks like as follows:

```
POST https://api.cognitive.microsoft.com/sts/v1.0/issueToken HTTP/1.1
Ocp-Apim-Subscription-Key: *YOUR_SUBSCRIPTION_KEY*
Host: api.cognitive.microsoft.com
Content-type: application/x-www-form-urlencoded
Content-Length: 0
Connection: Keep-Alive
```

## Send recognition request to the speech service 

To preform speech recognition, you need to make a POST request to the Microsoft speech service with proper request header and body. 
 
### REST end points

The URI for the REST endpoints of speech service is built as follows:
```
https://speech.platform.bing.com/speech/recognition/<RECOGNITION_MODE>/cognitiveservices/v1?language=<LANG_CODE>&format=<OUTPUT_FORMAT>
```

<RECOGNITION_MODE> specifies the recognition mode. It must be of the following values: `interactive`, `conversation`, or `dictation`. Please replace <RECOGNITION_MODE> with the mode you want to use. Please find more information on recognition mode in the [How to choose recognition mode](How-to/how-to-choose-recognition-mode) page.

<LANG_CODE> specifies the target language for audio convertion. You can find the complete list of languages and their code supported by the Speech service in . For example, en-us represents English (United States). A complete list supported languages and their code can be found in the is described 

<OUTPUT_FOMAT> is optional. Allowed values are `simple` and `detailed`. By default the service returns simple results. Please fine detailed description in the [Reference](Reference) page. 

Some examples of service URI are 
| | URI |
|---|---|
| 
```
https://speech.platform.bing.com/speech/recognition/<YOUR_RECOGNITION_MODE>/cognitiveservices/v1?language=en-us
```

## Request headers

The follow fields must be set in the request header.

Atuorization: 
ContentType:

# [Powershell](#tab/Powershell)
```Powershell

$SpeechServiceURI =
'https://speech.platform.bing.com/speech/recognition/interactive/cognitiveservices/v1?language=en-us&locale=en&format=detailed&requestid=2ed7cbc9-a214-44f3-851c-ae53e5'

# $OAuthToken is the access token returned by the token service. 
$RecoRequestHeader = @{
  'Authorization' = 'Bearer '+ $OAuthToken;
  'Transfer-Encoding' = 'chunked'
  'Content-type' = 'audio/wav; codec="audio/pcm"; samplerate=16000'
}

# Read audio into byte array
$audioBytes = [System.IO.File]::ReadAllBytes("YOUR_AUDIO_FILE")

$Response = Invoke-RestMethod -Method POST -Uri $SpeechServiceURI -Headers $RecoRequestHeader -Body $audioBytes

# Show the result
$Response

```

# [cURL](#tab/cURL)
```
curl -v -X POST "https://speech.platform.bing.com/speech/recognition/interactive/cognitiveservices/v1?language=pt-BR&locale=your_locale&format=your_format&requestid=your_guid" -H "Transfer-Encoding: chunked" -H 'Authorization: Bearer your_access_token' -H 'Content-type: audio/wav; codec="audio/pcm"; samplerate=16000' --data-binary @YOUR_AUDIO_FILE
```

# [C#](#tab/CSharp)
```cs
HttpWebRequest request = null;
request = (HttpWebRequest)HttpWebRequest.Create(requestUri);
request.SendChunked = true;
request.Accept = @"application/json;text/xml";
request.Method = "POST";
request.ProtocolVersion = HttpVersion.Version11;
request.Host = @"speech.platform.bing.com";
request.ContentType = @"audio/wav; codec=""audio/pcm""; samplerate=16000";
request.Headers["Authorization"] = "Bearer " + token;
```
---

The follow is a sample request payload. Please note that the token in the Autorization header is just an example.

```
POST https://speech.platform.bing.com/speech/recognition/interactive/cognitiveservices/v1?language=en-US&format=detailed&requestid=39530efe-5677-416a-98b0-93e13ec93c2b HTTP/1.1
Accept: application/json;text/xml
Content-Type: audio/wav; codec="audio/pcm"; samplerate=16000
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzY29wZSI6Imh0dHBzOi8vc3BlZWNoLnBsYXRmb3JtLmJpbmcuY29tIiwic3Vic2NyaXB0aW9uLWlkIjoiMWRjYWQxZTQzZWZlNDM2MmIzMjg2ZWY2OTIzYTA5MjYiLCJwcm9kdWN0LWlkIjoiQmluZy5TcGVlY2guRjAiLCJjb2duaXRpdmUtc2VydmljZXMtZW5kcG9pbnQiOiJodHRwczovL2FwaS5jb2duaXRpdmUubWljcm9zb2Z0LmNvbS9pbnRlcm5hbC92MS4wLyIsImF6dXJlLXJlc291cmNlLWlkIjoiL3N1YnNjcmlwdGlvbnMvYTM0Y2FkYmYtNTU5My00ZWYxLWI0MjItMDJhMDMyNmQ2NmZkL3Jlc291cmNlR3JvdXBzL1Rlc3QvcHJvdmlkZXJzL01pY3Jvc29mdC5Db2duaXRpdmVTZXJ2aWNlcy9hY2NvdW50cy9UZXN0U1BlZWNoIiwiaXNzIjoidXJuOm1zLmNvZ25pdGl2ZXNlcnZpY2VzIiwiYXVkIjoidXJuOm1zLnNwZWVjaCIsImV4cCI6MTQ5MzQyOTE2OX0._Bhx7nneMto2gjAAwmIO6eiSejQ2Nqhd8xFl0odjk40
Host: speech.platform.bing.com
Transfer-Encoding: chunked
Expect: 100-continue
```

## Chunked transfer encoding
Bing Speech API supports chuncked transfer encoding for efficient audio streaming. To transcribe speech to text, you can send the audio as one whole chunk or you can chop the audio into small chunks. For efficient audio transcription it is recommended that you use [chunked transfer encoding](https://en.wikipedia.org/wiki/Chunked_transfer_encoding) to stream the audio to the service. Other implementations may result in higher user-perceived latency. 

> [!NOTE]
> You may not upload more than 10 seconds of audio in any one request and the total request duration cannot exceed 14 seconds. 

The code snippet below shows an example of an audio file being chunked into 1024 byte chunks.

```cs
using (fs = new FileStream(audioFile, FileMode.Open, FileAccess.Read))
{

    /*
    * Open a request stream and write 1024 byte chunks in the stream one at a time.
    */
    byte[] buffer = null;
    int bytesRead = 0;
    using (Stream requestStream = request.GetRequestStream())
    {
        /*
        * Read 1024 raw bytes from the input audio file.
        */
        buffer = new Byte[checked((uint)Math.Min(1024, (int)fs.Length))];
        while ((bytesRead = fs.Read(buffer, 0, buffer.Length)) != 0)
        {
            requestStream.Write(buffer, 0, bytesRead);
        }

        // Flush
        requestStream.Flush();
    }
}
```

## Speech recognition response
After processing the request, Bing Speech API returns the results in a response as JSON format. The code snippet below shows an example of how you can read the response from the stream:

```cs
/*
* Get the response from the service.
*/
Console.WriteLine("Response:");
using (WebResponse response = request.GetResponse())
{
    Console.WriteLine(((HttpWebResponse)response).StatusCode);

    using (StreamReader sr = new StreamReader(response.GetResponseStream()))
    {
        responseString = sr.ReadToEnd();
    }

    Console.WriteLine(responseString);
    Console.ReadLine();
}
```

The following is a sample JSON response:

```json
OK
{
	"RecognitionStatus": "Success",
	"Offset": 22500000,
	"Duration": 21000000,
	"NBest": [{
		"Confidence": 0.941552162,
		"Lexical": "find a funny movie to watch",
		"ITN": "find a funny movie to watch",
		"MaskedITN": "find a funny movie to watch",
		"Display": "Find a funny movie to watch."
	}]
}
```
