---
layout: post
title: "How to deploy a Jekyll blog on AWS using S3 and CloudFront with a TLS certificate"
categories: howto
---

## Assumptions
- You have a Jekyll blog
- You are willing to pay for S3, CloudFront, and Route 53
> [S3 pricing](https://aws.amazon.com/s3/pricing/), [CloudFront pricing](https://aws.amazon.com/cloudfront/pricing/), [Route 53 Pricing](https://aws.amazon.com/route53/pricing/) <br/>
> You get limited free S3 and CloudFront services on the [Free Tier](https://aws.amazon.com/free/?awsf.Free%20Tier%20Types=categories%23alwaysfree)
- You want a TLS certificate (HTTPS)
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

Verify the follow settings for you CloudFront distribution
- Origin Domain Name: The name of your S3 bucket without "http://"  e.g.  `sampledomain.com.s3-website-us-east-1.amazonaws.com`
- HTTP and HTTPS
- Use all edge servers
- SSL/TLS Certificate: Leave as default for now

Create the distribution. Wait until the status of the CloudFront distribution is "Deployed"

## Route 53

If you register your domain with Route 53, it will automatically create a Hosted Zone for you. This is recommended because DNS registrars suck. If you have your domain with another registrar, copy the NS (Name Server) to your domain registrar's name servers and Route 53 will handle the configuration steps.

Route 53 will link your domain name to the CloudFront distribution.  

- Go to Route 53 and "Create Hosted Zone" if not done already
- Select your hosted zone and "Create Record Set"
- Verify it is an 'A' record IPv4 address and check "Alias"
- The alias target should be your CloudFront distribution
- Wait a while for everything to propogate, but eventually your `domain.com` should hit your CloudFront distribution and load quickly

![arecord](public/img/arecord.png)

## AWS Certificate Manager (ACM)

Getting a TLS certificate specifically for this .xyz tld was a pain, but YMMV with more standard .com tld's. The main goal is to request a certificate for your domain and validate that you're the owner of that domain.

- Go to the ACM service in AWS and "Request a certificate"
- Select a public certificate and enter your domain  e.g. example.com
- Select "Add another name to this certificate" and enter `*.example.com` This will apply the certificate to all subdomains on your top level domain
- For the validation method, first try email validation. If your email address appears in the [WHOIS lookup](https://whois.icann.org/en) database, then this will probably work. See troubleshooting steps below if this does not work
- You should receive an email which will let you validate your domain ownership and activate the ACM certificate
- In ACM, your certificate should have an "Issued" status
- In CloudFront, navigate to your distribution and click "Edit"
- Select 'Custom SSL/TLS certificate' with your new ACM certificate
- At this point, your `domain.com` should show a TLS certificate in the browser

#### Troubleshooting ACM
- If your email address was not shown in the WHOIS database and/or you did not receive the verification email, try disabling privacy protection. In AWS, go to Route 53, Registered Domains, Edit Contacts. Scroll down and disable privacy protection
- Privacy protection hides some contact details for certain domains, so you might have to try DNS validation

##### Validating ACM with DNS
- Select DNS validation when requesting a new ACM certificate
- Add an additional domain with "www.example.com"
- After requesting a certificate with DNS validation, go to the ACM dashboard and expand the entry
- Select "Export DNS configuration to a file." This downloads a csv file with each domain name, the Record Name, Record Type, and Record Value
- Navigate to Route 53 and add a new 'CNAME' record set
- Enter the Record Name and Record Value fields for your `domain.com` from the csv file
- Wait for the update and hopefully your site will show your certificate in the browser

![cname](public/img/cname.png)

## www Redirect

Directions for setting up a redirect from `www.domain.com` to `domain.com`

- Create a new bucket in S3 named `www.domain.com`
- Verify that the bucket has Public permissions for the ACL and Bucket Policy. These settings are under the 'Permissions' tab in S3
- Under the 'Properties' tab, select 'Static Web Hosting' and 'Redirect requests' to the target bucket of your `domain.com`
- Protocol, `https`
- Navigate to CloudFront and set up a new distribution that links to your new S3 bucket
- Navigate to Route 53 and add a new 'A - IPV4' record set
- Set up a new alias that points to your new CloudFront distribution
- Save everything!

## Conclusion

I know I missed some troubleshooting steps, but most of what I ran into is in this guide. The ACM and CloudFront configuration is probably the most annoying part. Everything else is googleable. If you are interested in how all of this actually works read vasanthk's excellent guide on ![How the Web Works](https://github.com/vasanthk/how-web-works)

Thanks for reading

dcoo









