---
title: Using REST API for speech recognition  | Microsoft Docs
description: Using REST to access Speech API in Microsoft Cognitive Services to convert spoken audio to text.
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

With Microsoft Speech Service, you can develop applications using REST API to convert spoken audio to text.

To use Speech API REST end points, the steps are as follows:
1. Authenticate and get a JSON Web Token (JWT) from the token service. For that you need first to subscribe to Microsoft Speech Serivce.
2. Set the proper request header and send the request to the appropriate Microsoft Speech API REST end point.
3. Parse the response to get your transcribed text.

## Prerequisites

### Subscribe to Speech API and get a free trial subscription key
To access the REST end point, you must first subscribe to Speech API, which is part of Microsoft Cognitive Services (previously Project Oxford). After subscribing, you will have the necessary subscription keys that are needed in the following operations. Both the primary and secondary keys can be used. For subscription and key management details, see [Subscriptions](https://azure.microsoft.com/en-us/try/cognitive-services/).

### Precorded audio file
In this example, we use a recorded audio file to illustrate the usage of the REST API. Please record a short audio file of you saying something short (e.g.: *"What is the weather like today?"* or *"Find funny movies to watch."*). The Microsoft Speech API also supports external microphone input. Please see the [sample applications](samples) for how to use microphone input.

> [!NOTE]
> The example requires that audio is recorded as wav file with **PCM single channel (mono), 16000 Hz**.

## Get authorization token
To access the REST endpoint, you need a valid authorization token. To get this token, you must first have a subscription key from the Speech API, as described [here](GetStartedREST##Prerequisites). Then you can send a POST request to the token service with the subscription key, and receives in the response the access token back as a JSON Web Token (JWT), which is passed through in the Speech request header.

> [!NOTE]
> The token has an expiry of 10 minutes. Please see the Authentication(How-to/how-to-authentication) section for how to renew the token. 

The token service URI is located here:

```
https://api.cognitive.microsoft.com/sts/v1.0/issueToken
```

The code following is an example implementation in C# for how to handle authentication. Please replace *YOUR_SUBSCRIPTION_KEY* with your own subscription key.

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
The example assumes running curl on Linux using bash or Cygwin on Windows. You may need to install curl if it is not available on your platform. The examples should work in Git Bash, zsh, and other shells too.

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

To perform speech recognition, you need to make a POST request to the Microsoft speech service end points with proper request header and body. 
 
#### REST end points

The URI for the REST endpoints of speech service is defined as follows:
```
https://speech.platform.bing.com/speech/recognition/<RECOGNITION_MODE>/cognitiveservices/v1?language=<LANGUAGE_TAG>&format=<OUTPUT_FORMAT>
```

`<RECOGNITION_MODE>` specifies the recognition mode, and must be of the following values: `interactive`, `conversation`, or `dictation`. It is a part of resource path in the URI. For more information on recognition mode see the [How to choose recognition mode](How-to/how-to-choose-recognition-mode.md) page.

`<LANGUAGE_TAG>` describes the target language for audio conversion. It is required, and specified as a part of query string in the URI. It has the value as defined in the IETF language tag [BCP 47](https://en.wikipedia.org/wiki/IETF_language_tag), and must be one of the languages that are supported by the service. For example, en-us represents English (United States). The complete list of languages supported by the Speech service can be found in the page [Supported Languages](API-Reference-REST/supportedlanguages.md).

`<OUTPUT_FOMAT>` is an optional parameter in the query string. Its allowed values are `simple` and `detailed`. By default the service returns results in `simple` format. For details see the [Output Format](api-reference-rest/bingvoicerecognition#output-format) page.

Some examples of service URI are 
| Recognition mode  | Language | Output format | End point URI |
|---|---|---|---|
| interactive | pt-BR | default | https://speech.platform.bing.com/speech/recognition/interactive/cognitiveservices/v1?language=pt-BR | 
| conversation | en-US | detailed | https://speech.platform.bing.com/speech/recognition/conversation/cognitiveservices/v1?language=en-US&format=detailed |
| dictation | fr-FR | simple | https://speech.platform.bing.com/speech/recognition/dictation/cognitiveservices/v1?language=fr-FR&format=simple |

#### Request headers

The follow fields must be set in the request header.

* `Authorization`: The autorization field must specify `Bearer` as type and use the authorization token you have gotten from the token service as credentials. 
* `Content-type`: The Content-type field describes the format and technical properties of the audio stream. Currently only wav file and PCM Mono 16000 encoding is supported, and the Content-type value for this format is `audio/wav; codec=audio/pcm; samplerate=16000`.

The field `Transfer-Encoding` is optional. Setting this field to `chunked` allows you to chop the audio into small chunks. More information is provided in the page [Chunked Transfer](How-to/how-to-chunked-transfer).

The follow is a sample request header. Please note that the token in the Authorization header is just an example.

```
POST https://speech.platform.bing.com/speech/recognition/interactive/cognitiveservices/v1?language=en-US&format=detailed HTTP/1.1
Accept: application/json;text/xml
Content-Type: audio/wav; codec=audio/pcm; samplerate=16000
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzY29wZSI6Imh0dHBzOi8vc3BlZWNoLnBsYXRmb3JtLmJpbmcuY29tIiwic3Vic2NyaXB0aW9uLWlkIjoiMWRjYWQxZTQzZWZlNDM2MmIzMjg2ZWY2OTIzYTA5MjYiLCJwcm9kdWN0LWlkIjoiQmluZy5TcGVlY2guRjAiLCJjb2duaXRpdmUtc2VydmljZXMtZW5kcG9pbnQiOiJodHRwczovL2FwaS5jb2duaXRpdmUubWljcm9zb2Z0LmNvbS9pbnRlcm5hbC92MS4wLyIsImF6dXJlLXJlc291cmNlLWlkIjoiL3N1YnNjcmlwdGlvbnMvYTM0Y2FkYmYtNTU5My00ZWYxLWI0MjItMDJhMDMyNmQ2NmZkL3Jlc291cmNlR3JvdXBzL1Rlc3QvcHJvdmlkZXJzL01pY3Jvc29mdC5Db2duaXRpdmVTZXJ2aWNlcy9hY2NvdW50cy9UZXN0U1BlZWNoIiwiaXNzIjoidXJuOm1zLmNvZ25pdGl2ZXNlcnZpY2VzIiwiYXVkIjoidXJuOm1zLnNwZWVjaCIsImV4cCI6MTQ5MzQyOTE2OX0._Bhx7nneMto2gjAAwmIO6eiSejQ2Nqhd8xFl0odjk40
Host: speech.platform.bing.com
Transfer-Encoding: chunked
Expect: 100-continue
```

#### Send request to the service

The following example shows how to send a speech recognition request to Microsoft speech service. We use the `interactive` recognition mode. Please replace `YOUR_AUDIO_FILE` with path to your prerecorded audio file, and `YOUR_ACCESS_TOKEN` with the authorization token you got from the token service.

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

# [cURL](#tab/cURL)
```
curl -v -X POST "https://speech.platform.bing.com/speech/recognition/interactive/cognitiveservices/v1?language=en-us&format=detailed" -H "Transfer-Encoding: chunked" -H "Authorization: Bearer YOUR_ACCESS_TOKEN" -H "Content-type: audio/wav; codec=audio/pcm; samplerate=16000" --data-binary @YOUR_AUDIO_FILE
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
request.ContentType = @"audio/wav; codec=audio/pcm; samplerate=16000";
request.Headers["Authorization"] = "Bearer " + token;

// Send an audio file by 1024 byte chunks
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
---

#### Speech recognition response
After processing the request, Bing Speech API returns the results in a response as JSON format. The code snippet below shows an example of how you can read the response from the stream:

# [Powershell](#tab/Powershell)
```Powershell
# show the response in JSON format
ConvertTo-Json $RecoResponse
```

# [C#](#tab/CSharp)
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
---

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
