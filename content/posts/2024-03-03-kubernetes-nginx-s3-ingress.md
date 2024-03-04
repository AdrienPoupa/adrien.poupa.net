---
title: "Serving a S3 Bucket with Kubernetes and Ingress-Nginx"
date: 2024-03-03T00:00:00-05:00
author: Adrien Poupa
url: /serving-s3-bucket-kubernetes-ingress-nginx/
tags:
    - kubernetes
    - s3
    - aws
    - devops
---

The traditional, production-ready way to serve a S3 bucket in production is usually to create a CloudFront distribution,
add a S3 origin and configure the desired behaviour.

However, this comes with a small caveat: while you can specify an 
[origin path](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/distribution-web-values-specify.html#DownloadDistValuesOriginPath), 
this will make CloudFront request the bucket from the path directory, then appending the requested URL.

For example, given the `test` origin path, `example.com/index.html` will retrieve `s3://bucket-name/test/index.html`;
`example.com/folder/index.html` will retrieve `s3://bucket-name/test/folder/index.html`. In other words, the folder structure
must match the URL structure, which is not always the desired outcome. 
While one could use [CloudFront Lambda@Edge](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/lambda-at-the-edge.html) 
to overcome this, if CloudFront is not necessary to begin with there is a simpler solution with Kubernetes and ingress-nginx.

In my case, the bucket contains a versioned application as follows:
```
└── app-name/
    ├── 1.0.0/
    │   └── index.js
    └── 1.0.1/
        └── index.js
```

I wanted a given URL to point to a given version, so that `example.com/folder/index.js` would retrieve
`s3://bucket-name/app-name/1.0.0/index.js`. This would not be possible with vanilla CloudFront.

Enter Ingress-Nginx; all we need to do is create a new Ingress as follows:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: s3-bucket-ingress
  namespace: app-frontend
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: "/app-name/1.0.0/$2"
    nginx.ingress.kubernetes.io/upstream-vhost: bucket-name.s3.amazonaws.com
    nginx.ingress.kubernetes.io/from-to-www-redirect: "true"
    nginx.ingress.kubernetes.io/use-regex: "true"
    nginx.ingress.kubernetes.io/backend-protocol: "https"
spec:
  ingressClassName: nginx
  rules:
  - host: example.com
    http:
      paths:
      - path: /folder(/|$)(.*)
        pathType: ImplementationSpecific
        backend:
          service:
            name: s3-bucket-service
            port:
              number: 443
```
Notice the `nginx.ingress.kubernetes.io/rewrite-target` specifies the bucket folder name, `$2` injects the path from the
rule (everything that comes after `/folder`) and points to its corresponding Service:
```yaml
kind: Service
apiVersion: v1
metadata:
  name: s3-bucket-service
  namespace: app-frontend
spec:
  type: ExternalName
  externalName: "bucket-name.s3.amazonaws.com"
```

Thanks to Ingress-Nginx's [Rewrite annotations](https://kubernetes.github.io/ingress-nginx/examples/rewrite/), content
from `/app-name/1.0.0` can be served at a different location, similar to what a `proxy_pass` rule in Nginx would do.

There is no need to enable the static website hosting property in the bucket, but the bucket must be reachable from
the Kubernetes cluster. Given the bucket is private, a rule to allow the cluster's VPC to perform `s3.GetObject` is enough:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Allow-bucket-VPC",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": ["arn:aws:s3:::bucket-name/*"],
      "Condition": {
        "StringEquals": {
          "aws:sourceVpc": "vpc-123qwe"
        }
      }
    }
  ]
}
```

Note that you will need to [create a VPC endpoint](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints-s3.html) if you do not already have one.