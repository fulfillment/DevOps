# AWS Elasticbeanstalk Single Instance

## Public Networks

More Soon

## Private Networks

Note: A common theme will be to have an ENV var `APP_DOMAIN` & `APP_ENV`

## Update Route 53 DNS

AWS is quirky when it comes to assigning an IP to the Environment URL (x-environment-x.x-app-x.us-east-1.elasticbeanstalk.com)

1. Ideally you would place the instance in a private subnet.
2. Obviously you disabled the Public IP address.
3. BUT they assigned an EIP (Elastic IP) address to the instance.
4. AND they point the R53 Alias and Environment URL to the EIP.

So let's assign the IPv4 internal address of the EC2 to the domain of `APP_DOMAIN`. You will need to get the Route 53 Hosted Zone ID and edit line 8.

[99-dns.config](99-dns.config)

What we are doing:

1. Getting the Local IPv4 Address from the [Instance Metadata](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html)
2. Getting the domain from the Instance Environment Variables
3. Using the AWS CLI issuing an UPSERT (Update if you can, Insert if you must) to Route 53. Note the `APP_DOMAIN` will not include a trailing period (.) but it is expected in the DNS world, this command will add the period.
