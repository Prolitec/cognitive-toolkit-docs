---
title: How to authenticate to use Speech Client APIs | Microsoft Docs
description: How to authenticate to use Speech Client APIs
services: cognitive-services
author: zhouwang
manager: wolfma

ms.service: cognitive-services
ms.technology: speech
ms.topic: article
ms.date: 09/15/2017
ms.author: zhouwang
---

# How to authenticate to use Speech Client APIs

**Not Completed Yet**

## Using Subscription Key
#### Subscribe to Speech API and get a free trial subscription key
To access the REST end point, you must subscribe to Speech API which is part of Microsoft Cognitive Services (previously Project Oxford). After subscribing, you will have the necessary subscription keys to execute this operation. Both the primary and secondary keys can be used. For subscription and key management details, see [Subscriptions](https://azure.microsoft.com/en-us/try/cognitive-services/). 


After you select the services that you want, click Subscribe to get the key. Each service returns a primary and secondary key. Both keys are tied to the same quota, so you may use either key. If you feel that the key has been compromised, click Regenerate to get a new key. Regenerating the key will not reset your quota.
If you need a higher number of transactions per second or per month, click Buy to get a paid subscription.

## Authorization 

In addition to the standard web socket handshake headers, speech requests require an *Authorization* header. Connection requests without this header are rejected 
by the service with a 403 Forbidden HTTP response.

The *Authorization* header must contain a JSON Web Token (JWT) access token.

For information about subscribing and obtaining API keys that are used to retrieve valid JWT access tokens, see [Get Started for Free](https://www.microsoft.com/cognitive-services/en-US/sign-up?ReturnUrl=/cognitive-services/en-us/subscriptions?productId=%2fproducts%2fBing.Speech.Preview).

The API key is passed to the token service. For example:

```
POST https://api.cognitive.microsoft.com/sts/v1.0/issueToken
Content-Length: 0
```

The required header information for token access is as follows.

Name	| Format	| Description
---------|---------|--------
Ocp-Apim-Subscription-Key |	ASCII	| Your subscription key

The token service returns the JWT access token as `text/plain`. Then the JWT is passed as a `Base64 access_token` to the handshake as an authorization header prefixed with the string `Bearer`. For example:

`Authorization: Bearer [Base64 access_token]`


## Get access token
The Speech REST API requires the client to authenticate with a valid access token as proof of the authentication/authorization. To get this token, you must first obtain a subscription key from the Speech API, as described [here](GetStartedREST##Prerequisites). Then you send a POST request to the token service with the subscription key, and receives in the response the access token back as a JSON Web Token (JWT).

> [!NOTE]
> The token has an expiry of 10 minutes. See the [Authentication](How-to/how-to-authentication.md) page for how to renew the token. 

The token service URI is located here:
```
https://api.cognitive.microsoft.com/sts/v1.0/issueToken
```

The code sample next shows how to get access token, after you have obtained the subscription key. Note to replace *YOUR_SUBSCRIPTION_KEY* with your own subscription key.

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

# [C#](#tab/CSharp)

```cs
    /*
     * This class demonstrates how to get a valid access token.
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

The POST request sent to the token service by the preceding example looks like as follows:

```
POST https://api.cognitive.microsoft.com/sts/v1.0/issueToken HTTP/1.1
Ocp-Apim-Subscription-Key: YOUR_SUBSCRIPTION_KEY
Host: api.cognitive.microsoft.com
Content-type: application/x-www-form-urlencoded
Content-Length: 0
Connection: Keep-Alive
```