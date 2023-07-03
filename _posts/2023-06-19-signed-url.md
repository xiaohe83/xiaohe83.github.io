---
title: How to create a private bucket with signed url/cookie in Google Cloud
author: xiao he
date: 2023-06-19
category: Jekyll
layout: post
---

Actually, the detailed instructions on how to configure this can already be found in the Google Cloud documentation. So why am I writing this article? As I encountered some misleading articles while trying to solve this problem, which prompted me to compile this article again.

### Before we start
I believe that if you are able to see this article, you have already used Google Cloud and are familiar with the gcloud tools (if not, you can learn about it by [install the gcloud CLI](https://cloud.google.com/sdk/docs/install) ). You should also have a clear understanding of why signed URLs/cookies are needed. Without further ado, let's move on to the next section.

### The right document
[This article](https://medium.com/@thetechbytes/private-gcs-bucket-access-through-google-cloud-cdn-430d940ebad9) describes a method using a backend service -> Internet network endpoint group to access a private bucket. However, this method is *no longer applicable*. If you haven't been misled by that, we can proceed.

### Creating a bucket
select *Cloud Storage* from the left navigation menu, and create:
![create private bucket](https://pics.gcs.hexhxiao.com/Google-Cloud-Bucket/create-private-bucket.png?Expires=1790000000&KeyName=signkey&Signature=sNQkrUoIOkSIKtEVi4iKilvKaFA=)
*Notice*:
1. enable **Public access prevention**
![enable public access prevention](https://pics.gcs.hexhxiao.com/Google-Cloud-Bucket/prevention-public-notice.png?Expires=1790000000&KeyName=signkey&Signature=j45upY8b4GW-GkufhHOnUGCMZZ8=)


### Upload a test image
![upload test file](https://pics.gcs.hexhxiao.com/Google-Cloud-Bucket/upload-file.png?Expires=1790000000&KeyName=signkey&Signature=3zPZ27MLU19pFNOVubkgoTjEOgA=)
And you can test the file was unable to visit from the public

### create load balancer
select the *Network Services* -> *Load balacing* from the left navigation menu, and create:
![create load balancing](https://pics.gcs.hexhxiao.com/Google-Cloud-Bucket/create-lb.png?Expires=1790000000&KeyName=signkey&Signature=BVvkchVXr4YsDuSbLHW_t9658Xw=)
*Notice*:
1. Global HTTP(S) Load Balancer (**classic**) was a must-have.

### create frontend
![create frontend](https://pics.gcs.hexhxiao.com/Google-Cloud-Bucket/create-frontend.png?Expires=1790000000&KeyName=signkey&Signature=0z0QERTscuB55uvlCbIV3LgWfAk=)
*Notice*:
1. the Network Service Tier **Premium** was a must-have.

### create backend
![create backend](https://pics.gcs.hexhxiao.com/Google-Cloud-Bucket/create-backend-bucket.png?Expires=1790000000&KeyName=signkey&Signature=XDkWvnBJQ0B4X5w0Vu8fwy8sETs=)
then
![use sign key](https://pics.gcs.hexhxiao.com/Google-Cloud-Bucket/restricted-content.png?Expires=1790000000&KeyName=signkey&Signature=aZyYpi0CJW8AACtOet_0ELZgtCs=)
*Notice*:
1. In the Restricted content, the **Url signing key** was a must-have. (the CDN will not generate the cloud-cdn-fill user in the next section when you choose allow public access)
2. The key shoulde be **copied**, it will be used when you test the signed url/cookie.

### grant read-only access from CDN
```
PROJECT_ID=xxxx
gcloud config set project $PROJECT_ID
PROJECT_NUMBER=$(gcloud projects list --filter="$PROJECT_ID" --format="value(PROJECT_NUMBER)")
gsutil iam ch serviceAccount:service-${PROJECT_NUMBER}@cloud-cdn-fill.iam.gserviceaccount.com:objectViewer gs://hex-demo-bucket
```
You can find the offical doc from here: [config permission](https://cloud.google.com/cdn/docs/using-signed-urls#configure_permissions)

*Notice*:
1. The user service-${PROJECT_NUMBER} is owned by Cloud CDN, so you can not find it from your service accounts list.
2. The restricted content must be set in the backend section

### Test the signed url
You can get the  test code snippets from [github](https://github.com/GoogleCloudPlatform/python-docs-samples/blob/HEAD/cdn/snippets.py)

For example:
```
[hex@demo] python3 surl.py sign-url https://pics.gcs.hexhxiao.com/Google-Cloud-Bucket/restricted-content.png signkey XXXXYYYY== 1790000000
https://pics.gcs.hexhxiao.com/Google-Cloud-Bucket/restricted-content.png?Expires=1790000000&KeyName=signkey&Signature=aZyYpi0CJW8AACtOet_0ELZgtCs=
```

At last, all the images in this blog are used the signed url. Have fun!
