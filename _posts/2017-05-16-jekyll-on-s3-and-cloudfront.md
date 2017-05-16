---
title: "Jekyll on S3 and CloudFront using s3_website"
description: Deploying a Jekyll site to S3 and CloudFront using s3_webiste
date: 2017-05-16 12:00:00 -0700
---

[GitHub Pages](https://pages.github.com/) is an easy way to host static websites for your projects or personal sites. It even supports using [a custom domain](https://help.github.com/articles/using-a-custom-domain-with-github-pages/). Unfortunately HTTPS is only supported when you are hosting on the default `github.io` domain. This post documents how to setup Jekyll with S3 and CloudFront using [s3_website](https://github.com/laurilehmijoki/s3_website), giving you an easy solution for hosting a static websites using a custom domain and HTTPS.

### Prerequisites

* [AWS console access](https://console.aws.amazon.com/). S3 and CloudFront are AWS services.
* Email access for your custom domain to be able to receive [the domain validation emails](https://docs.aws.amazon.com/acm/latest/userguide/gs-acm-validate.html), ie. `admin@YOUR-DOMAIN.com`. This allows you to generate free SSL certificates.
* Control of your domain's DNS, preferably via AWS Route53.

### Setup

#### S3

1. Create an [S3 bucket](https://console.aws.amazon.com/s3/home) to host your website content, for example `YOUR-DOMAIN.com`. I'd recommend picking the cheapest regions, like `us-east-1`.
2. [Allow the bucket's contents to be publicly accessibly](https://docs.aws.amazon.com/AmazonS3/latest/dev/HostingWebsiteOnS3Setup.html#step2-add-bucket-policy-make-content-public) by setting the bucket policy to the following:
    ```json
    {
      "Version":"2012-10-17",
      "Statement":[{
        "Sid": "PublicReadForGetBucketObjects",
        "Effect": "Allow",
        "Principal": "*",
        "Action": ["s3:GetObject"],
        "Resource": ["arn:aws:s3:::YOUR-BUCKET-NAME/*"]
      }]
    }
    ```

#### SSL Certificate

If you want to use HTTPS to access your website, either request or upload a certificate using the [AWS Certificate Manager](https://console.aws.amazon.com/acm/home). This will be used by your CloudFront distribution to provide HTTPS access.

If you request a certificate, create it in the `us-east-1` region so it can be used by CloudFront, and be sure include both `YOUR-DOMAIN.com` and `*.YOUR-DOMAIN.com` so you can accept requests from both the root domain and sub-domains. Do not continue with setting up CloudFront until you have verified the certificate via email.

#### CloudFront

Create a new [CloudFront Distribution](https://console.aws.amazon.com/cloudfront/home) with the following settings:

* For delivery method select **Web**.
* For _Origin Domain Name_, enter the [website endpoint](https://docs.aws.amazon.com/AmazonS3/latest/dev/WebsiteEndpoints.html) for your S3 bucket. See [the documentation for a list of website endpoints for each region](https://docs.aws.amazon.com/general/latest/gr/rande.html#s3_website_region_endpoints). **Do not use the auto-filled S3 endpoint**, otherwise you cannot serve paths that end in `/` or do not have an extension.
* For _Origin ID_, enter `S3-YOUR-BUCKET-NAME`.
* Change the _Viewer Protocol Policy_ value to suit your needs, I recommend **Redirect HTTP to HTTPS**.
* Under _Distribution Settings_
  * Change _Price Class_ to suit your needs.
  * Update _Alternate Domain Names (CNAMEs)_ with `YOUR-DOMAIN.com` and other domains you will access the site with (ie. `www.YOUR-DOMAIN.com`).
  * Update _SSL Certificate_ with the certificate you requested or uploaded above.

##### DNS

You will also need to update your domain's DNS to point to this new CloudFront distribution. The recommended and easiest way is with AWS Route53.

If you are using AWS Route53 you can setup an alias resource record set to point to your CloudFront distribution by following [this documentation](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-to-cloudfront-distribution.html).
If you have enabled IPv6, be sure to create both `A` and `AAAA` alias records.

If you are using another DNS provider, you **cannot** host the website on the zone apex (ie. `YOUR-DOMAIN.com`) because you cannot create a `CNAME` record there. You can only create a `CNAME` record for subdomains (ie. `www.YOUR-DOMAIN.com`). The `CNAME` record must point to the CloudFront distribution's domain name, which will be the in the format `dXXXXXXXXXXXXX.cloudfront.net`, and can be found on the distribution's details page.

#### s3_website

1. [Install the `s3_website` gem](https://github.com/laurilehmijoki/s3_website/tree/34f45a4dd5d4a89e74cd418ffd3ca5a2d4232e46#install), `gem install s3_website`.
2. Create a `s3_website.yml` configuration based on [this example](https://github.com/laurilehmijoki/s3_website/blob/34f45a4dd5d4a89e74cd418ffd3ca5a2d4232e46/additional-docs/example-configurations.md#optimised-for-speed).
3. Follow steps for [using environment variables](https://github.com/laurilehmijoki/s3_website/tree/34f45a4dd5d4a89e74cd418ffd3ca5a2d4232e46#using-environment-variables) and move secrets to a `.env` file. Be sure to add `.env` to your repo's `.gitignore`.
4. Run `s3_website cfg apply` to apply the `s3_website` configuration settings to your S3 bucket and CloudFront distribution.
5. Generate your Jekyll site by running `bundle exec jekyll b`.
6. Push your content to S3 by running `s3_website push`.
7. Test your website by browsing to `YOUR-DOMAIN.com` or `www.YOUR-DOMAIN.com`.
