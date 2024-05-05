---
author: "Loko"
title: "Mysterious S3 Access Denied Error"
date: 2023-07-01
lastmod: 2024-01-04
description: "Access Denied error occurred while trying to retrieve a triggered object"
tags: ["aws", "go", "troubleshooting"]
thumbnail: /thumbnail/green-bucket.webp
toc: true
---

## How I discovered the error

I actually encountered the same error twice.  
Neither error occurred in the course of work, but both occurred on the side projects that I had implemented a feature to test work-related functionality.  
What I implemented was a **Lambda function that triggered by S3 object created event.**  
These Lambda functions had a common logic: **get the object's key name from the received event, and look up the object information again via that key name.**  
However, my Lambda functions returned an Access Denied error.  
Even though my lambda functions had enough permission to run `GetObject` from that bucket!

<img width="669" alt="aws s3 access denied" src="https://github.com/nmin11/blog/assets/75058239/2ebb354e-dab6-4e92-b4da-916d729dbef6">

## What was the problem

This is because the key name contains values such as **Korean, special characters, or spaces.**  
In such cases, the value of `Record.S3.Object.Key` received through the event was URL encoded.  
I mentioned earlier that I experienced the same problem twice, once because the name was Korean, and once because it contained special characters 😓

<img width="853" alt="url encoded key name" src="https://github.com/nmin11/blog/assets/75058239/e76bcbaf-7e9e-48fe-8ae2-1db58bf9a20c">

My guess is that this is because the object itself is in URL format.

<img width="618" alt="object url" src="https://github.com/nmin11/blog/assets/75058239/a26d5a94-4b88-4173-8566-9d49ec97378b">

Using the Object URL above, I tried to get the object information directly through the AWS SDK in my local code.

<script src="https://gist.github.com/nmin11/95c04703578e7099ec91091aac088b12.js"></script>

Then I got this error.

```sh
Failed to retrieve object: NoSuchKey: The specified key does not exist.
```

Not surprisingly, but one thing was different.  
In my local code, it returned the correct error that the object for that key did not exist,  
But my Lambda function returned Access Denied error.  
Again, my Lambda has a permission to run `GetObject` on that Bucket, even if there's no such key.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "VisualEditor0",
      "Effect": "Allow",
      "Action": ["s3:GetObject"],
      "Resource": "arn:aws:s3:::nmin-access-test/*"
    }
  ]
}
```

I suspect this is caused by AWS not properly branching for errors that can occur when a Lambda accesses S3.  
Because of this incorrect way of branching the error, I actually wasted about a week of my time trying to solve the problem by searching for "S3 Access Denied" related keywords.  
_So, despite the fact that it's a simple problem, I thought I'd publish this post._

## How to solve

If a Lambda function needs to look up information about an object received through an event again,  
it needs to parse the key name from the event as a URL-formatted string back to the original string.

```go
decodedObject, err := url.PathUnescape(object)
if err != nil {
  fmt.Println("Error occurred while decoding URL: ", err)
}

decodedObject = strings.Replace(decodedObject, "+", " ", -1)
```

In Go, there's a built-in module called `url.PathUnescape` that I used.  
However, that function doesn't handle empty space value(` `) being URL-encoded with special character(`+`), so I had to use the `strings.Replace` function in addition.  
And finally, I was able to retrieve the object successfully even if I received a key name with Korean characters, special characters, or empty spaces.

### Source code used for demonstration

<script src="https://gist.github.com/nmin11/26204a27da20909f5c18fc851b835dcc.js"></script>

---

## + Added on 2024/1/4

Today at work, I stumbled upon a reason as to why AWS S3 returns an Access Denied error even when the object doesn't exist.  
It's because AWS policy doesn't let me know if the object I am looking for via `GetObject` exists or not.  
I did a little more research after work and found [a question](https://stackoverflow.com/questions/56027399/why-am-i-getting-different-errors-when-trying-to-read-s3-key-that-does-not-exist) on StackOverflow.  
To summarize, it's designed to prevent me from exploring whether a particular key exists when I only have `GetObject` permission but not `ListObject` permission.  
In the example code I implemented, I had both `GetObject` and `ListObject` permissions in my local environment because I was running it directly with my AWS credentials.  
However, when I deployed it as a Lambda, I was only granted `GetObject` permissions, which is why I was experiencing permissions issues.
