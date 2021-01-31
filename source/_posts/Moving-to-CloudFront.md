---
title: Moving to CloudFront
date: 2021-01-31 08:09:15
---

I've been maintaining a little server that runs docker containers hosting very small, static websites for a few years. Doing so gave me an opportunity to build with docker and work with a system based on containers. Every three months, when my Let's Encrypt certificate expired, I would create a new VM, run my scripts to rebuild the system, and launch it. When I updated the site, I would wait for Docker Hub to build a new image and I would pull and run it on production.

I've had a lot of fun working and maintaining the system, but it's time to simplify so I less maintenance requirements. I've moved my sites over to S3 buckets with CloudFront in front of them and used Route 53 to handle DNS.

## Get an SSL Certificate from Certificate Manager

I used Certificate Manager to store my certificates because they're free, easily integrated into other AWS services, and automatically renew. That removes my responsibility for renewing Let's Encrypt certs every three months.

There are many paths to get certificates through certificate manager. I'm not going to rewrite their docs here. Check out [Certificate Manager documentation](https://docs.aws.amazon.com/acm/latest/userguide/acm-overview.html) for more details.

## Upload Static Site to S3

I have four sites that I moved.

- My main landing page
  - assortedsolutions.com and www.assortedsolutions.com
- My blog
  - blog.assortedsolutions.com
- Donkey in the Dark
  - dind.assortedsolutions.com
- My band's website
  - thunderkittensband.com and www.thunderkittensband.com

I created four S3 buckets, `assortedsolutions.com`, `blog.assortedsolutions.com`, `dind.assortedsolutions.com`, `thunderkittensband.com`, and uploaded the static site contents. Note that I left out `www.assortedsolutions.com`. I used Route 53 to point to `assortedsolutions.com`. Some blogs say to use a separate S3 bucket with a redirect and have another CloudFront distribution that points to it. I found it simpler to eschew both of those and have Route 53 handle it at the DNS level.

### S3 Configuration

After creating a bucket, go to the `Properties` tab and edit `Static Website Hosting`. Enable it and set the default index document. In my case it was `index.html`. Save changes.

Open the `Permissions` tab. Uncheck "Block all public access" and add the following bucket policy. Make sure to change the resource to match the bucket's name.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::assortedsolutions.com/*"
    }
  ]
}
```

## CloudFront Configuration

I created four CloudFront distributions, one for each S3 bucket. Here are the settings that I changed.

### Origin Domain Name

This is an auto-complete list that I used to find my S3 bucket.

### Viewer Protocol Policy

Set to "Redirect HTTP to HTTPS".

### Compress Objects Automatically

Set to "Yes". This will automatically use any compression that is supported by the browser making the request, reducing the bytes over the wire.

### Price Class

I switched to "Use Only U.S., Canada and Europe" to reduce costs.

### Alternate Domain Names (CNAMEs)

Set this to the domain that you're hosting. For example, when setting up the CloudFront distribution for blog.assortedsolutions.com, the value in this field is "blog.assortedsolutions.com".

When setting up a site that should host both the top-level domain and "www", both must be specified. For example, when creating the distribution for assortedsolutions.com, this field should contain two values: assortedsolutions.com and www.assortedsolutions.com.

### SSL Certificate

Select "Custom SSL Certificate" and find the certificate in the auto-complete box that was uploaded to Certificate Manager.

## Route 53 Configuration

Create a hosted zone for the top-level domains you want to host. In my case, I have two: assortedsolutions.com and thunderkittensband.com. I'll use the assortedsolutions.com hosted domain in the rest of this example.

On the page for the hosted zone assortedsolutions.com select "Create record". Select "Simple routing". Select "Define simple record".

For the top-level domain, leave the "Record name" field blank. For other sub-domains, such as blog.assortedsolutions, use "blog".

In the "Value/Route traffic to" section, select "Alias to CloudFront distribution" and select the correct region. If set up correctly, the correct CloudFront distribution should be the only option in the "Choose distribution" field. Select it. Click "Define simple record".

Follow similar steps for the remaining subdomains: dind.assortedsolutions.com and blog.assortedsolutions.com.

For the www domain, the "Value/Route traffic to" should be set to "Alias to another record in this hosted zone", select the region, and choose the record "assortedsolutions.com."
