This builds a web application on AWS Elastic Beanstalk.

Prereqs: Bootstrap your AWS account
         CDK Installed 
         
Part 1

Bash Script to create directory :
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
