---
layout: post
title: "How to Deploy a Jekyll Blog on AWS Using S3 and CloudFront With an SSL Certificate"
categories: howto
---

## Assumptions
- You have a Jekyll blog
- You are willing to pay for S3, CloudFront, and Route53
> [S3 pricing](https://aws.amazon.com/s3/pricing/), [CloudFront pricing](https://aws.amazon.com/cloudfront/pricing/), [Route53 Pricing](https://aws.amazon.com/route53/pricing/) <br/>
> You get limited free S3 and CloudFront services on the [Free Tier](https://aws.amazon.com/free/?awsf.Free%20Tier%20Types=categories%23alwaysfree)
- You want an SSL certificate (HTTPS)
- You want a highly available, fault tolerant, static website (yes)

## [Create AWS Account](https://aws.amazon.com/)

## Create S3 Bucket

- Log into your AWS account and go to the AWS S3 storage service
- Create a new bucket with a unique name
- Select the closest region to where you're based
- On the Permissions tab, allow Public Read access
- Once the bucket is created, verify that it has public permissions
![s3permissions](public/img/s3permissions.png)
- Sample Bucket Policy json
```
{
    "Version": "2008-10-17",
    "Statement": [
        {
            "Sid": "PublicReadForGetBucketObjects",
            "Effect": "Allow",
            "Principal": {
                "AWS": "*"
            },
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::<bucket name>/*"
        }
    ]
}
```
- Under the Permissions tab, enable Static Web Hosting and set the ```index.html``` and ```error.html``` pages from your jekyll blog
![static](public/img/static.png)

## Create IAM User

Creating an Identity Access Management (IAM) user is standard practice when interacting with multiple services. This user will have S3 and CloudFront privileges to manage the website.

- Log into your AWS account and click on the IAM service
- Click on Users and add a new user
- Name your user and select Programmatic Access
![createUser](public/img/iam1.png)
- Attach full S3 privileges
![s3privileges](public/img/iam2a.png)
- Attach full CloudFront privileges
![cloudFrontPrivileges](public/img/iam2b.png)
- Verify User has correct privileges and continue
![verify](public/img/iam3.png)
- Download the credentials csv file and *save* the Access Key ID and Secret Access Key
![keys](public/img/iam4.png)

## Add S3 Website Gem to Jekyll
- Go into your Jekyll directory and install the [s3_website gem](https://github.com/laurilehmijoki/s3_website)
- ```gem install s3_website```
- Run ```s3_website cfg create``` to create new config file (s3_website.yml)
- You can optionally create a CloudFront distribution when prompted
- Create an environment variables file ```.env``` and make your env variables
    ```
    S3_ID=ABCDEFGHIJKLMNOP
    S3_SECRET=ABCDEFH012346
    ```
- Add your access keys and S3 bucket name to ```s3_website.yml```
    ```
    s3_id: <%= ENV['S3_ID'] %>
    s3_secret: <%= ENV['S3_SECRET'] %>
    s3_bucket: youruniquebucketname
    ```
- If the environment variables are not loading correctly, install the [dotenv gem](https://github.com/bkeepers/dotenv)
- Run ```jekyll build``` to build your site
- Run ```s3_website cfg apply``` to apply your new configuration changes
- Run ```s3_website push``` to push your website to S3
- Your S3 bucket should now have the contents of your jekyll blog and be live at the S3 endpoint
- The S3 endpoint can be found in Properties \> Static Web Hosting in the S3 dashboard

## CloudFront

CloudFront distributes your website across AWS edge servers all over the world which drastically speeds up load times for people outside of your region.

Go to the CloudFront dashboard in the AWS console. If you created a distribution during the s3_website create command, you will see the distribution, if not create a new one.

## Route53

## ACM

TOWRITE
Verify Certificate troubleshooting

## www Redirect

## Conclusion









