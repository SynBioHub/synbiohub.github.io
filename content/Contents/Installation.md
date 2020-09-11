---
title: "Installation"
date: 2020-09-04T20:30:59+05:30
draft: true
---
## From Prebuilt Image
### Install Docker


## General
The docker-compose files in this repository represent various configurations for deploying SynBioHub.
The files can be layered with Docker Compose's [multiple file](https://docs.docker.com/compose/reference/overview/#specifying-multiple-compose-file) capabilities. 

The base configuration, described with `docker-compose.yml`, is simply SynBioHub, its graph database Virtuoso, and an autohealer.

To run the base configuration:
1. Open terminal
2. `git clone https://github.com/synbiohub/synbiohub-docker`
3. `docker-compose --f ./synbiohub-docker/docker-compose.yml up`

### With Explorer
To add [SBOLExplorer](https://github.com/michael13162/SBOLExplorer), add the `docker-compose.explorer.yml` to the main docker-compose, i.e. for step 3 run `docker-compose --f ./synbiohub-docker/docker-compose.yml -f ./synbiohub-docker/docker-compose.explorer.yml up`

### With Plugins
To add plugins to the configuration change step 3 to: `docker-compose --f ./synbiohub-docker/docker-compose.yml -f ./synbiohub-docker/docker-compose.explorer.yml -f ./synbiohub-docker/docker-compose.<Plugin 1 File Name>.yml -f ./synbiohub-docker/docker-compose.<Plugin 2 File Name>.yml up`

Note that all plugins are added before the `up` and each is preceeded by `-f `. For example, to run the configuration with the VisualIgem plugins and the VisualSeqviz plugin run:

`docker-compose --f ./synbiohub-docker/docker-compose.yml -f ./synbiohub-docker/docker-compose.explorer.yml -f ./synbiohub-docker/docker-compose.pluginVisualIgem.yml -f ./synbiohub-docker/docker-compose.pluginVisualSeqviz.yml up`

### With Developmental SynBioHub
The `docker-compose.version.yml` can be added to another configuration, and simply contains the latest version of the SynBioHub docker image. 
This version does not even contain the Virtuoso image, so it should only be used by someone who knows what they are doing. 

## Plugins
This is a sortable table with plugin information. Click on the header to sort by that column.

  

|Name|Type|Port|Language|Test|Description|Multi-Endpoint|Repository|File|
|--- |--- |--- |--- |--- |--- |--- |--- |--- |
|DownloadSnapgene|Download|8083|Python|No|||Plugin-Download-Snapgene|docker-compose.pluginDownloadSnapgene.yml|
|SubmitSnapgene|Submit|8084|Python|No|||Plugin-Submit-Snapgene|docker-compose.pluginSubmitSnapgene.yml|
|SubmitTest|Submit|8087|Python|Yes|Simply indicates that submit plugins are working and provides a framework to play with for plugin developers|No|Plugin-Submit-Test|docker-compose.pluginSubmitTest.yml|
|VisualComponentUse|Visual|8080|Python|No|Containing a co-use component sankey diagram, and the most used components bar graph endpoints|Yes|Plugin-Visual-Component-Use|docker-compose.pluginVisualComponentUse.yml|
|VisualIgem|Visual|3000|TypeScript|No|Containing endpoints for iGEM Main Page, iGEM Design Page, and iGEM Experience Page|Yes|Plugin-Visual-Igem|docker-compose.pluginVisualIgem.yml|
|VisualSeqviz|Visual|8085|Javascript|No|Shows the plasmid view and sequence view of components|No|sequence-view-plugin|docker-compose.pluginVisualSeqviz.yml|
|VisualServelet|Visual|8086|Javascript|Yes|Allows testing of file serving and provides a framework to play with for plugin developers|No|Plugin-Visual-Serve-Test-js|docker-compose.pluginVisualServelet.yml|
|VisualTest|Visual|8081|Python|Yes|Smply indicates that visualisation plugins are working and provides a framework to play with for plugin developers|No|Plugin-Visual-Test|docker-compose.pluginVisualTest.yml|
|VisualTestJS|Visual|8082|Javascript|Yes|Aimply indicates that submit plugins are working and provides a framework to play with for plugin developers|No|Plugin-Visual-Test-js|docker-compose.pluginVisualTestJS.yml|

## From Source

Follow the instructions on the [GitHub README](https://github.com/synbiohub/synbiohub) to install SynBioHub locally on your system. If you would like SynBioHub to run as a service, you can enable Virtuoso using systemd or open a virtual terminal using tmux or GNU screen and run 

``` sudo /usr/local/bin/virtuoso-t +configfile $YOUR_CONFIG_FILE ```

 You should also run SynBioHub as a system service or using a virtual terminal and the command

``` npm start ```

If you are doing development work, you can start SynBioHub with the command 

``` npm run-script dev ```

which will restart the application with any change to the JavaScript source. 

## NGINX configuration

Instructions for managing nginx server blocks can be found [here](https://www.digitalocean.com/community/tutorials/how-to-set-up-nginx-server-blocks-virtual-hosts-on-ubuntu-16-04#step-three-create-server-block-files-for-each-domain).

The server block for a SynBioHub installation listening on port 7777 is to the right 

```
client_max_body_size 800m;
proxy_connect_timeout 6000;
proxy_send_timeout 6000;
proxy_read_timeout 6000;
send_timeout 6000;
server {
        listen 80;
        server_name _;

        location / {
                proxy_set_header X-Real-IP  $remote_addr;
                proxy_set_header X-Forwarded-Server $host;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header Host $host;
                proxy_pass http://127.0.0.1:7777$request_uri;
        }
}
```
This is most useful when you would like to host SynBioHub on a subdomain alongside other content (using nginx as an HTTP proxy) or using HTTPS. 

# User Endpoints

Endpoints that control user related functions

## Login

`POST <SynBioHub URL>/login` 

This POST request requires email/password and returns a user token that should be passed in the X-authorization header to view private objects and submit new objects, etc. 

```plaintext
curl -X POST -H "Accept: text/plain" -d "email=<email>&password=<password>" <SynBioHub URL>/login
```

```python
import requests

response = requests.post(
    '<SynBioHub URL>/login',
    headers={
        'X-authorization': '<token>',
        'Accept': 'text/plain'
    },
    data={
        'email': '<email>',
        'password' : '<password>',
        },
)

print(response.status_code)
print(response.content)
```

```javascript
const fetch = require("node-fetch");
const { URLSearchParams } = require('url');
var url ='<SynBioHub URL>/login';
var headers = {
    "Accept": "text/plain",
}

const params = new URLSearchParams();
params.append('email', '<email>');
params.append('password', '<password>');

fetch(url, { method: 'POST', headers: headers, body: params})
    .then(res => res.text())
    .then(body => console.log(body));
```

Parameter | Description
--------- | ------- | -----------
email | The e-mail address of the user to login with.
password | The password of the user.

## Logout

`POST <SynBioHub URL>/logout`

This post request logs out the user specified in the X-authorization header.


```plaintext
curl -X POST -H "Accept: text/plain" -H "X-authorization: <token>" <SynBioHub URL>/logout
```

```python
import requests

response = requests.post(
    '<SynBioHub URL>/logout',
    headers={
    'X-authorization': '<token>',
    'Accept': 'text/plain'
    }
)

print(response.status_code)
print(response.content)
```

```javascript
const fetch = require("node-fetch");
const { URLSearchParams } = require('url');
const url = '<SynBioHub URL>/logout'
var headers={
    "Accept" : "text/plain; charset=UTF-8",
    "X-authorization" : "<token>"
};

fetch(url, { method: 'POST', headers: headers, body:params})
    .then(res => res.buffer()).then(buf => console.log(buf.toString()))
    .catch (error=>console.log(error))
```

## Register

`POST <SynBioHub URL>/register`

Register a new user on SynBioHub.

```plaintext
curl -X POST -H "Accept: text/plain" -d "username=<username>&name=<name>&affiliation=<affiliation>&email=<email>&password1=<password1>&password2=<password2>" <SynBioHub URL>/register
```

```python
import requests
response = requests.post(
    '<SynBioHub URL>/register',
    headers={
        'Accept': 'text/plain'
    },
    data={
        'username': '<username>',
        'name' : '<name>',
        'affiliation' : '<affiliation>',
        'email' : '<email>',
        'password1' : '<password1>',
        'password2' : '<password2>'
        },
)
print(response.status_code)
print(response.content)
```

```javascript
const fetch = require("node-fetch");
const { URLSearchParams } = require('url');
const url = '<SynBioHub URL>/register'
var headers={
    "Accept" : "text/plain; charset=UTF-8"
};

const params = new URLSearchParams();
params.append('username', '<username>');
params.append('name', '<name>');
params.append('affiliation', '<affiliation>');
params.append('email', '<email>');
params.append('password1', '<password1>');
params.append('password2', '<password2>');

fetch(url, { method: 'POST', headers: headers, body:params})
    .then(res => res.buffer()).then(buf => console.log(buf.toString()))
    .catch (error=>console.log(error))
```

Parameter | Description
--------- | ------- | -----------
username | Username of the user
name | Name of the user
affiliation | Affiliation of the user
email | Email address of the user
password1 | Password of the user
password2 | Password confirmation


## Request Reset Password Token

`POST <SynBioHub URL>/resetPassword`

Request a reset password token.

```plaintext
curl -X POST -H "Accept: text/plain" -d "email=<email>" <SynBioHub URL>/resetPassword
```

```python
import requests
response = requests.post(
    '<SynBioHub URL>/resetpassword',
    headers={
        'Accept': 'text/plain'
    },
    data={
        'email': '<email>'
        },
)
print(response.status_code)
print(response.content)

```

```javascript
const fetch = require("node-fetch");
const { URLSearchParams } = require('url');
const url = '<SynBioHub URL>/resetPassword'
var headers={
    "Accept" : "text/plain; charset=UTF-8"
};

const params = new URLSearchParams();
params.append('email', '<email>');

fetch(url, { method: 'POST', headers: headers, body:params})
    .then(res => res.buffer()).then(buf => console.log(buf.toString()))
    .catch (error=>console.log(error))
```

Parameter | Description
--------- | ------- | -----------
email | Email address of the user to send password reset link

## Set New Password

`POST <SynBioHub URL>/setNewPassword`

Resets the user's password with the provided token and two copies of the new password.

```plaintext
curl -X POST -H "Accept: text/plain" -d "token=<token>&password1=<password1>&password2=<password2>" <SynBioHub URL>/setNewPassword
```

```python
import requests
response = requests.post(
    '<SynBioHub URL>/setNewPassword',
    headers={
        'Accept': 'text/plain'
    },
    data={
        'token': '<token>',
        'password1': '<password1>',
        'password2': '<password2>'
        },
)
print(response.status_code)
print(response.content)
```

```javascript
const fetch = require("node-fetch");
const { URLSearchParams } = require('url');
const url = '<SynBioHub URL>/setNewPassword'
var headers={
    "Accept" : "text/plain; charset=UTF-8"
};

const params = new URLSearchParams();
params.append('token', '<token>');
params.append('password1', '<password1>');
params.append('password2', '<password2>');

fetch(url, { method: 'POST', headers: headers, body:params})
    .then(res => res.buffer()).then(buf => console.log(buf.toString()))
    .catch (error=>console.log(error))
```

Parameter | Description
--------- | ------- | -----------
token | Token received by email to reset a password
password1 | New password
password2 | Confirm new password

## View Profile

`GET <SynBioHub URL>/profile`

View the user's profile.

```plaintext
curl -X GET -H "Accept: text/plain" -H "X-authorization: <token>" <SynBioHub URL>/profile

This endpoint returns JSON metadata of the form 

{
	"id":1,
	"name":"Test User",
	"username":"testuser",
	"email":"test@user.synbiohub",
	"affiliation":"jimmy",
	"password":"",
	"graphUri":"http://localhost:7777/usertestuser",\
	"isAdmin":true,\
	"resetPasswordLink":"",\
	"isCurator":true,\
	"isMember":true,
	"createdAt":"2020-05-19T14:44:44.204Z",
	"updatedAt":"2020-05-20T21:11:21.934Z","user_external_profiles":[]
}
```

```python
import requests

response = requests.get(
    '<SynBioHub URL>/profile',
    headers={
	'X-authorization': '<token>',
	'Accept': 'text/plain'
    }
)

print(response.status_code)
print(response.content)
```

```javascript
const fetch = require("node-fetch");
const url = '<SynBioHub URL>/profile'	
const headers={
        "Accept" : "text/plain; charset=UTF-8",
	"X-authorization" : "<token>"
};
fetch(url, { method: 'GET', headers: headers})
    .then(res => res.buffer()).then(buf => console.log(buf.toString()))
    .catch (error=>console.log(error))
```

Note that the X-authorization header is required for this endpoint.

## Update Profile

`POST <SynBioHub URL>/profile`

Update the user's profile.

```plaintext
curl -X POST -H "Accept: text/plain" -H "X-authorization:<token>" -d "name=<name>&affiliation=<affiliation>&email=<email>&password1=<password1>&password2=<password2>" <SynBioHub URL>/profile
```

```python
import requests
response = requests.post(
    '<SynBioHub URL>/profile',
    headers={
        'X-authorization': '<token>',
        'Accept': 'text/plain'
    },
    data={
        'name': '<name>',
        'affiliation' : '<affiliation>',
        'email' : '<email>',
        'password1' : '<password1>',
        'password2' : '<password2>',

        },
)
print(response.status_code)
print(response.content)
```

```javascript
const fetch = require("node-fetch");
const { URLSearchParams } = require('url');
const url = '<SynBioHub URL>/profile'
var headers={
    "Accept" : "text/plain; charset=UTF-8",
    "X-authorization" : "<token>"
};

const params = new URLSearchParams();
params.append('name', '<name>');
params.append('affiliation', '<affiliation>');
params.append('email', '<email>');
params.append('password1', '<password1>');
params.append('password2', '<password2>');

fetch(url, { method: 'POST', headers: headers, body:params})
    .then(res => res.buffer()).then(buf => console.log(buf.toString()))
    .catch (error=>console.log(error))
```
