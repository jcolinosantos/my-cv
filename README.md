# my-cv

This repositorie contains my personal website code hosted in AWS.

## About

## Built with
GitHub:

This repositorie contains all the code needed for my website.

I pretend to include an AWS CloudFormation document to build the whole infraestructure (see [issue #3](https://github.com/jcolinosantos/my-cv/issues/3)).

I used the [Hugo](https://gohugo.io/) framework to build the website.

Initialy, I tried to use GitHub Actions in order to communicate with AWS CodeBuild and deploy the code to a S3 Bucket.

In order to do that, I used the official GitHub Action from AWS to request the credentials (aws-actions/configure-aws-credentials). 
I hit an [open issue](https://github.com/aws-actions/configure-aws-credentials/issues/271) that made me reconsider the original approach. It would be much easier to trigger an AWS CodePipeline automatically with each commit and discard completely the usage of GitHub Actions.  

AWS CodePipeline:

This service listens for changes in this repository and triggers an AWS CodeBuild execution.

AWS CodeBuild:

This service consists on three different stages:

- Source: It clones this repo into an aarch64 Linux container. It's an Amazon Linux image based on CentOS7.

- Build: It installs Hugo and its dependencies. Then, it builds the project and leaves the artifact in the `public` directory. It is defined [here](https://github.com/jcolinosantos/my-cv/blob/main/configuration/buildspec.yml)

- Deploy: It uploads the artifact directory to a S3 bucket. 

AWS S3:

In this part of the project I faced several issues.

Firstly, I created a S3 Bucket with a random name. Inside the bucket I created a prefix named `public/` inspired by the ["Best practices design patterns" document](https://docs.aws.amazon.com/AmazonS3/latest/userguide/optimizing-performance.html).

I made this bucket public. Then, I modified the bucket policie to make `public/` public.

I thought this would be the perfect solution. Given the `index.html` URL I was able to load the website with minor issues. Some assets didn't load correctly. 

I realized that I was viewing a document, not a website. Hence the assets issues. The bucket wasn't set as a static website host. 

While enabling this, I realised that I couldn't use any prefix as the root path for my website. This leaded to a change in the CodeBuild Deploy stage to point to the right path.

This also made me update the bucket policie.

There will be another issue with S3, that I'll proceed to explain in the next section. Route 53 will arise new issues.

AWS Route53:

I registered a domain using Route53. Then I pointed the Alias registry to the S3 Bucket. Then I realised that, in order to do this, the name of the bucket needs to be equal to the name of the domain (this also applies to subdomains). So, I created a new bucket. Changed the permissions. Enabled the static web hosting. Etc.

AWS CloudFront:

Route 53 only provides domains. The static web hosting propierty only supports HTTP. In order to provide a HTTPS connection to my website, I requested a new certificate and created a CloudFront distribution. 

CloudFront provides a new bucket policy that should be updated in S3. 

I also created a new CNAME registry in the hosted zone to properly validate the SSL certificate. And an A registry to link Route 53 with the CloudFront distribution. 
