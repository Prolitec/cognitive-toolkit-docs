﻿---
title: Bing Speech API in Microsoft Cognitive Services | Microsoft Docs
description: Use the Bing Speech API to add speech-driven actions to your apps, including real-time interaction with users.
services: cognitive-services
author: priyaravi20
manager: yanbo

ms.service: cognitive-services
ms.technology: speech
ms.topic: article
ms.date: 09/14/2017
ms.author: prrajan
---
# Bing Speech API overview

Microsoft Speech API provides you easy-to-use APIs to create powerful speech-enabled features in your applications, like voice command control, user dialog using natual speech conversation, and speech transcription and dictation. The Microsoft Speech API supports both *Speech to Text* and *Text to Speech* conversion.

* **Speech to Text** API converts human speech to text that can be used as input or commands to control your application.
* **Text to Speech** API converts text to audio streams that can be played back to the user of your application.

## Speech to text (speech recognition)
The *Speech to Text* API *transcribes* audio streams into text that your application can display to the user or act upon as command input. The *Speech To Text* API provides developers an easy way to integrate Microsoft speech reconition technologies into their applications.

* Support many spoken languages in multiple dialects. For the full list of supported languages in
each recognition mode, see [Recognition Languages](api-reference-rest/bingvoicerecognition.md#recognition-language).
* Leverage powerful speech recognition technolgoies that are used by Cortana, Office Dictation, Office Translator, and other Microsoft products.
* Multiple recognition modes to enable optimized results in different user scenarios. The *Speech to Text* APIs currently support *interactive*, *conversation*, and *dictation* mode.
* Real-time continuous recognition. The *Speech to Text* supports client to receive the interim recognition results of the words that have been recognized so far. The speech service also supports end-of-speech detection.
* Support capitalization and punctuation, masking profanity, and text normalization.
* Integration with language understanding. Besides converting the input audio into text, the *Speech to Text* provides applications an additional capability to understand what the text means. It uses the [Language Understanding Intelligent Service(LUIS)](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/Home) to extract intents and entities from the recognized text.
* Provide both REST and client libraries for running on various platforms (Windows, Android, iOS) using different languages (C#, Java, JavaScript, ObjectiveC).

Developers can choose either [REST API](GetStarted/GetStartedREST) or [Microsoft Speech Client Libraries](GetStarted/GetStartedClientLibraries) to access Microsoft speech to text services.

| Use cases | [REST](GetStarted/GetStartedREST) | [Client Libraries](GetStarted/GetStartedClientLibraries) |
|-----|-----|-----|
| Convert a short spoken audio, e.g. commands (audio length < 15s) without intermit results | Yes | Yes |
| Convert a long audio (> 15s) | No | Yes |
| Stream audio with interim results desired | No | Yes |
| Undertand the text converted from audio using LUIS | No | Yes |

### Next Steps
* Get started to use the *Speech to Text* APIs with [REST API](GetStarted/GetStartedREST) or [Client Libraries](GetStarted/GetStarted)
* Check out [sample applications](samples) in your perferred programming lanugage.
* Go to the Reference section to find [Microsoft Speech Protocol](API-Reference-REST/websocketprotocol) details and API references.

## Text to speech (speech synthesis)
*Text to Speech* APIs use REST to convert structured text to an audio stream. The APIs provide fast text to speech
conversion in various voices and languages. For the full list of supported languages and voices, see
[Supported Locales and Voice Fonts](api-reference-rest/bingvoiceoutput.md#SupLocales).

### Text to speech API
The maximum amount of audio returned for any single request is 15 seconds.
