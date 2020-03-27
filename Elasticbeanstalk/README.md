# AWS Elasticbeanstalk <!-- omit in toc -->

All `.config.yaml` files should be renamed to `.config` and placed in the `.ebextensions` folder.

- [LB & Single Instance](#lb--single-instance)
  - [Custom Logging & Log Rotation](#custom-logging--log-rotation)
- [Single Instance](#single-instance)
  - [Public Networks](#public-networks)
  - [Private Networks](#private-networks)
    - [Update Route 53 DNS](#update-route-53-dns)

## LB & Single Instance

### Custom Logging & Log Rotation

[05-logging-nodejs.config.yaml](DevOps/blob/master/Elasticbeanstalk/All/05-logging-nodejs.config.yaml)

In this example we are logging from node.js, these logs can grow large so periodically we want to rotate and compress them out.

Details are provided in the `.config` file.

References:
* https://aws-labs.com/understanding-logrotate-utility/
* https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.logging.html#health-logs-logrotate

Example:

```js
let log = function(entry) {
    entry = new Date().toISOString() + ' - ' + entry + '\n'

    fs.appendFile('/var/log/applogs/app.log', entry, (err) => {
        if (err) {
            console.log('Cannot write to log', err)
        }
    })
}

// -- OR --

let log = function(entry) {
    try {
        fs.appendFileSync('/tmp/appLogs/app.log', new Date().toISOString() + ' - ' + entry + '\n')
    } catch (e) {
        console.log('Cannot write to log', e)
    }
}
```

## Single Instance

### Public Networks

More Soon

### Private Networks

Note: A common theme will be to have an ENV var `APP_DOMAIN` & `APP_ENV`

#### Update Route 53 DNS

AWS is quirky when it comes to assigning an IP to the Environment URL (x-environment-x.x-app-x.us-east-1.elasticbeanstalk.com)

1. Ideally you would place the instance in a private subnet.
2. Obviously you disabled the Public IP address.
3. BUT they assigned an EIP (Elastic IP) address to the instance.
4. AND they point the R53 Alias and Environment URL to the EIP.

So let's assign the IPv4 internal address of the EC2 to the domain of `APP_DOMAIN`. You will need to get the Route 53 Hosted Zone ID and edit line 8.

[Single-Instance/99-dns.config](99-dns.config.yaml)

What we are doing:

1. Getting the Local IPv4 Address from the [Instance Metadata](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html)
2. Getting the domain from the Instance Environment Variables
3. Using the AWS CLI issuing an UPSERT (Update if you can, Insert if you must) to Route 53. Note the `APP_DOMAIN` will not include a trailing period (.) but it is expected in the DNS world, this command will add the period.
