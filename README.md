This builds a web application on AWS Elastic Beanstalk.

Prereqs: Bootstrap your AWS account
         CDK Installed 
         
Part 1

Bash Script to create directory
      
      mkdir my_webapp
      cd my_webapp

Next we are going to initialize the nodejs project
      
      npm init -y

Now run Bash script to install Express as a dependency for Nodejs
      
      npm install express

Create a new file called <app.js> and add this code to it
      
      var express = require('express');
      var app = express();
      var fs = require('fs');
      var port = 8080;
      
      app.get('/test', function (req, res) {
    res.send('the REST endpoint test run!');
      });


      app.listen(port, function() {
      console.log('Server running at http://127.0.0.1:', port);
      });
      
Create a new file called <index.html> and add this html to it
      
      <html>
     <head>
        <title>Elastic Beanstalk App</title>
    </head>

    <body>
        <h1>Welcome to the demo for ElasticBeanstalk</h1>
        <a href="/test">Call the test API</a>
    </body>
     </html>
     
Add this code to the <app.js> file above the /test call
      
      app.get('/', function (req, res) {
      html = fs.readFileSync('index.html');
      res.writeHead(200);
      res.write(html);
      res.end();
      });
      
In the package.json file replace the scripts section with this 
    
    "scripts": {
    "start": "node app.js"
    },
  
This will start a local server with the URL http://127.0.0.1:8080 or http://localhost:8080.  


Part 2

Create the CDK app

First make sure you have the latest cdk version by running this bash script

         npm install -g cdk
         
Next create the directory and move into it

         mkdir cdk-eb-infra
         cd cdk-eb-infra
   
Now initialize the CDK application
         
         cdk init app --language typescript
         
Intall the S3 assests module

         npm i @aws-cdk/aws-s3-assets
         
         
Locate the file /lib/cdk-eb-infra-stack.ts here is where you will write the code for the resource stack you are going to create
At the top of the file
                  
                  import s3assets = require('@aws-cdk/aws-s3-assets');
                  
Inside the stack, under the commented line that says "The code that defines your stack goes here" add the following code.

         const webAppZipArchive = new s3assets.Asset(this, 'WebAppZip', {
         path: `${__dirname}/../app.zip`,
                 });
                 
Then add the dependency to the top of the /lib/cdk-eb-infra-stack.ts file.

                 import elasticbeanstalk = require('@aws-cdk/aws-elasticbeanstalk');
                 
Add this underneath where you added the S3 assets
                  
                  const appName = 'MyWebApp';
                  const app = new elasticbeanstalk.CfnApplication(this, 'Application', {
                  applicationName: appName,
                  });
                  
                  const appVersionProps = new elasticbeanstalk.CfnApplicationVersion(this, 'AppVersion', {
                  applicationName: appName,
                  sourceBundle: {
                  s3Bucket: webAppZipArchive.s3BucketName,
                  s3Key: webAppZipArchive.s3ObjectKey,
                  },
                  });
                  
                  appVersionProps.addDependsOn(app);
                  
Creating the instance profile

First we need to install the module into the CDK by running this bash script
                  
                  npm i @aws-cdk/aws-iam
             
 Import the dependency with the rest of the resources you've created so far
 
                  import iam = require('@aws-cdk/aws-iam');
                  
Add this underneath where you've added assets like s3 and ELB

                  const myRole = new iam.Role(this, `${appName}-aws-elasticbeanstalk-ec2-role`, {
                  assumedBy: new iam.ServicePrincipal('ec2.amazonaws.com'),
                          });

                  const managedPolicy = iam.ManagedPolicy.fromAwsManagedPolicyName('AWSElasticBeanstalkWebTier')
                  myRole.addManagedPolicy(managedPolicy);

                  const myProfileName = `${appName}-InstanceProfile`

                  const instanceProfile = new iam.CfnInstanceProfile(this, myProfileName, {
                  instanceProfileName: myProfileName,
                  roles: [
                   myRole.roleName
                   ]
                  });
                  
Create Elastic Beanstalk environment

Add this underneath where you have been logging all your resources so far

                           const optionSettingProperties: elasticbeanstalk.CfnEnvironment.OptionSettingProperty[] = [
                             {
                           namespace: 'aws:autoscaling:launchconfiguration',
                           optionName: 'IamInstanceProfile',
                            value: myProfileName,
                              },
                             {
                           namespace: 'aws:autoscaling:asg',
                           optionName: 'MinSize',
                           value: '1',
                           },
                            {
                           namespace: 'aws:autoscaling:asg',
                           optionName: 'MaxSize',
                           value: '1',
                           },
                            {
                            namespace: 'aws:ec2:instances',
                           optionName: 'InstanceTypes',
                            value: 't2.micro',
                            },
                           ];
                           
                           const elbEnv = new elasticbeanstalk.CfnEnvironment(this, 'Environment', {
                           environmentName: 'MyWebAppEnvironment',
                           applicationName: app.applicationName || appName,
                           solutionStackName: '64bit Amazon Linux 2 v5.4.4 running Node.js 14',
                           optionSettings: optionSettingProperties,
                           versionLabel: appVersionProps.ref,
                           });
