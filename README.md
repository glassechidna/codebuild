# A completely unofficial source of CodeBuild Docker images

AWS CodeBuild has two features that are both super useful:

* [Curated images](https://docs.aws.amazon.com/codebuild/latest/userguide/build-env-ref-available.html) with 
  a **lot** of useful software pre-installed. 
* Custom images: the ability to specify your own Docker image to use as your
  build environment.
  
Unfortunately, where it falls down is _combining_ these two features. If you want
to customise one of the curated images, you _can_ because AWS provides the [Dockerfiles](https://github.com/aws/aws-codebuild-docker-images)
used to build those images. But you won't want to, for a few reasons:

* Building `aws/codebuild/standard:4.0` takes 25 minutes on a `general1.large`
* Running a build job on CodeBuild using that custom image takes about 350 seconds
  on a `general1.small` - that's about five and a half minutes more than the usual
  20-something seconds. Massive overhead!
  
The second issue is because CodeBuild has to download your ~3.7GB image and decompress
it to ~8.7GB. It has to do all that because your image doesn't share any Docker image
_layers_ with the curated images.

## Solution

The solution is to build your custom image and have your Dockerfile build on
top of the curated image's layers. AWS doesn't publish these, so I did it myself.
You use them like so:

```
FROM glassechidna/codebuild:ubuntu-4.0-20.05.05
RUN echo 'hello world' > /etc/example.txt
```

You can then publish that image to ECR, Docker Hub, anywhere you want. You'll
still have to upload (and pay for on ECR) ~4GB to store the entire image, but
when CodeBuild runs your custom image it will incur essentially no overhead. But
don't take my word for it, look at this graph:

<img width="642" alt="Benchmark" src="https://user-images.githubusercontent.com/369053/86545325-772f0000-bf71-11ea-86da-85135cff3b05.png">

## Available images

* `glassechidna/codebuild:ubuntu-4.0-20.05.05`
* `glassechidna/codebuild:ubuntu-3.0-20.05.05`
* `glassechidna/codebuild:ubuntu-2.0-20.05.05`
* `glassechidna/codebuild:ubuntu-1.0-20.05.05`
