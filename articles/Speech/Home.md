---
title: Microsoft Speech API in Cognitive Services | Microsoft Docs
description: Use Microsoft Speech API to add speech-driven actions to your apps, including real-time interaction with users.
services: cognitive-services
author: priyaravi20
manager: yanbo

ms.service: cognitive-services
ms.technology: speech
ms.topic: article
ms.date: 09/14/2017
ms.author: prrajan
---
# Microsoft Speech API overview

The cloud-based Microsoft Speech API provides developers with access to advanced speech recognition technologies in order to create powerful speech-enabled features in their applications, like voice command control, user dialog using natural speech conversation, and speech transcription and dictation. The Microsoft Speech API supports both *Speech to Text* and *Text to Speech* conversion.

* **Speech to Text** API converts human speech to text that can be used as input or commands to control your application.
* **Text to Speech** API converts text to audio streams that can be played back to the user of your application.

## Speech to text (speech recognition)
The *Speech to Text* API *transcribes* audio streams into text that your application can display to the user or act upon as command input. The *Speech To Text* API provides developers an easy way to integrate Microsoft speech recognition technologies into their applications.

* Leverage powerful speech recognition technologies from Microsoft, which are used by Cortana, Office Dictation, Office Translator, and other Microsoft products.
* Multiple recognition modes to enable optimized results in different user scenarios. The Microsoft Speech API currently supports *interactive*, *conversation*, and *dictation* mode.
* Real-time continuous recognition. The *Speech to Text* supports client to receive the interim recognition results of the words that have been recognized so far. The speech service also supports end-of-speech detection.
* Support many spoken languages in multiple dialects. For the full list of supported languages in
each recognition mode, see [Recognition Languages](api-reference-rest/supportedlanguages.md). It also supports capitalization and punctuation, masking profanity, and text normalization.
* Integration with language understanding. Besides converting the input audio into text, the *Speech to Text* provides applications an additional capability to understand what the text means. It uses the [Language Understanding Intelligent Service(LUIS)](../cognitive-services/LUIS/Home.md) to extract intents and entities from the recognized text.
* Provide both Representational State Transfer (REST) APIs and client libraries for running on various platforms (Windows, Android, iOS) using different languages (C#, Java, JavaScript, ObjectiveC).

Developers can choose either [REST APIs](GetStarted/GetStartedREST.md) or [Microsoft Speech Client Libraries](GetStarted/GetStartedClientLibraries.md) to access Microsoft speech to text services.

[!div class="mx-tdBreakAll"]
| Use cases | [REST APIs](GetStarted/GetStartedREST.md) | [Client Libraries](GetStarted/GetStartedClientLibraries.md) |
|-----|-----|-----|
| Convert a short spoken audio, for example, commands (audio length < 15 s) without interim results | Yes | Yes |
| Convert a long audio (> 15 s) | No | Yes |
| Stream audio with interim results desired | No | Yes |
| Understand the text converted from audio using LUIS | No | Yes |

### Next Steps
* Get started to use Microsoft Speech Service with [REST APIs](GetStarted/GetStartedREST.md) or [Client Libraries](GetStarted/GetStarted.md).
* Check out [sample applications](samples.md) in your preferred programming language.
* Go to the Reference section to find [Microsoft Speech Protocol](API-Reference-REST/websocketprotocol.md) details and API references.

## Text to speech (speech synthesis)
*Text to Speech* APIs use REST to convert structured text to an audio stream. The APIs provide fast text to speech
conversion in various voices and languages. For the full list of supported languages and voices, see
[Supported Locales and Voice Fonts](api-reference-rest/bingvoiceoutput.md#SupLocales).

### Text to speech API
The maximum amount of audio returned for any single request is 15 seconds.

