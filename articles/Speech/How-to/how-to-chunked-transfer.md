---
title: How to chunked transfer audio stream to the speech service | Microsoft Docs
description: How to use chunked trasfer to send audio stream to the speech service
services: cognitive-services
author: zhouwangzw
manager: wolfma61

ms.service: cognitive-services
ms.technology: speech
ms.topic: article
ms.date: 09/15/2017
ms.author: zhouwang
---


## Chunked transfer encoding
Bing Speech API supports chuncked transfer encoding for efficient audio streaming. To transcribe speech to text, you can send the audio as one whole chunk or you can chop the audio into small chunks. For efficient audio transcription it is recommended that you use [chunked transfer encoding](https://en.wikipedia.org/wiki/Chunked_transfer_encoding) to stream the audio to the service. Other implementations may result in higher user-perceived latency. 

> [!NOTE]
> You may not upload more than 10 seconds of audio in any one request and the total request duration cannot exceed 14 seconds. 

The code snippet below shows how to set the chunked transfer encoding and send an audio file being chunked into 1024 byte chunks.

```cs

HttpWebRequest request = null;
request = (HttpWebRequest)HttpWebRequest.Create(requestUri);
request.SendChunked = true;
request.Accept = @"application/json;text/xml";
request.Method = "POST";
request.ProtocolVersion = HttpVersion.Version11;
request.Host = @"speech.platform.bing.com";
request.ContentType = @"audio/wav; codec=audio/pcm; samplerate=16000";
request.Headers["Ocp-Apim-Subscription-Key"] = "YOUR_SUBSCRIPTION_KEY";

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
