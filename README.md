# cfn-cognito-logout-listener

## Introduction

So, you implemented Cognito on your AWS Elastic Load Balancer and now your application doesn't want to log out?

## Issue

This is caused by the `AWSELBAuthSessionCookie` remaining active. The only way to end your session is to invalidate this cookie server-side.

## Solution

When you are running a 3rd party application, logically you cannot integrate this into the application easily. To solve this, I have written a small Lamba which listens under the same load balancer by means of a listener rule and thus can expire the cookie.

## Code snippet

```python
import os
import urllib

location = "https://{}.auth.{}.amazoncognito.com/logout?client_id={}&redirect_uri={}&response_type=code&state=STATE&scope=openid".format(
    os.environ.get("USER_POOL_DOMAIN"),
    os.environ.get("REGION"),
    os.environ.get("USER_POOL_CLIENT"),
    urllib.parse.quote(os.environ.get("REDIRECT_URI"))
)

def send_response(event):
    return({
        "isBase64Encoded": False,
        "statusCode": 302,
        "statusDescription": "Found",
        "headers": {
            "content-type": "content-type: text/html; charset=utf-8",
            "set-cookie": "AWSELBAuthSessionCookie-0=empty;max_age=-3600",
            "location": location
        }
    })

def handler(event, context):
    return(send_response(event))
```

## Example

Please view the [cloudformation.yaml](cloudformation.yaml) for a full implementation.
