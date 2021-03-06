# node-chat-js

A simple file based, real time chat client-server solution in node.js, ready to be deployed in your applications in a single line of code.

## Table of content
* [Intro](#intro)
* [Deploy](#deploy)
* [Administration](#administration)
* [Auth](#auth)
* [Create a chat client in another page](#create-a-chat-client-in-another-page)
* [Files](#files)
	* [banned-addresses](#banned-addresses)
	* [users](#users)
	* [log](#log)
* [Functions & Events](#functions)

## Intro

node-chat-js is a node.js package to deploy a mono-room chat server that includes :

* Real time communication based on web sockets
* Administration panel
* Ban users by @IP address
* Manage administrators with login and password
* Log system

The solution comes with a single page application as the chat room at **<your_domain>/chat**, note that with the javascript client file you can create a chat where you want.

## Deploy

**First of all you need the node-chat-js folder in your node_modules.**

```bash
$ npm install node-chat-js
```

Then, you can createChatServer.

```javascript
const app = require('express')();
const server = require('http').createServer(app);
// create a chat server
const nodeChat = require('node-chat-js').createChatServer(app, server);
```
Now your application has 3 new urls
* **/chat** : a single page application which is the chat room.
* **/nc-login** : the login page.
* **/nc-admin** : the administration panel, you need to auth on /nc-login to get here.

All clients can now chat in real time either at **<your_domain>/chat** or anywhere in a page configured. see [create a chat client in another page](#create-a-chat-client-in-another-page)

## Administration

You can manage your chat server :
* add new administrators
* ban @IP addresses
* talk as an admin

First, you have to be authentified.

## Auth

Go to **/nc-login** and log with a correct user/password to get **/nc-admin**.

If it's the first time you try to log-in, you will be in **setup mode** and you'll have to create the first admin.

Then disconnect as asked and log in with correct user/password.

Users are specified in the node-chat-js/users file and can be added in the admin panel, passwords are salted and hashed with bcrypt, see [users].(#users)

## Create a chat client in another page

If you want to include the chat elsewhere, you can create a chat client in the HTML page you want.
This client will still be related to the /nc-admin panel.

**Don't forget to add the node-chat-js-client.js file and the socket.io dependency**
```javascript
<script src="/socket.io/socket.io.js"></script>
<script src="/node-chat-js/js/node-chat-js-client.js"></script>
```
Note : you already have the socket.io folder since it's a specified dependency of node-chat-js in package.json

Minimum elements :

```html
<div id="ncChatZone"></div> <!-- where messages are appendend -->
<div id="ncClientNumber"></div> <!-- real time number of clients connected -->

<!-- form for your users to chat -->
<form id="ncFormChat">
	<input type="text" id="ncName">
	<input type="text" id="ncMessage">
	<input type="submit">
</form>
```

You can look at the chat.ejs core content to see how it's done in the **/chat** app.
```html
        <div class="container">
            <div class="row d-flex flex-column justify-content-center">
		<!-- you need an element with #ncChatZone, it's where all messages are appended -->
                <div id="ncChatZone">
                    <div class="alert div-info text-center">
                        <img src="/node-chat-js/img/user.svg" width=20 height=20 alt="">
			<!-- element #ncClientNumber gets in real time the number of clients connected to the chat -->
                        <span id="ncClientNumber"></span>
                    </div>
                </div>
		<!-- most important part, you need a form with #ncFormChat and an input #ncName, input #ncMessage -->
                <form action="/" method="post" class="row form-group d-flex" id="ncFormChat">
                    <input class="form-control col-md-2" type="text" placeholder="Name" id="ncName" autofocus>
                    <input class="form-control col-md-8" type="text" placeholder="Hi, enter a message..." id="ncMessage">
                    <input class="btn btn-primary col-md-2" type="submit" placeholder="Send">
                </form>
            </div>
        </div>
```

## Files

**node-chat-js is based on files and not dependent to any database.**

### banned-addresses

Always edit this file from /nc-admin
Or restart the node server after. 

banned-addresses stores all @IP that are banned from the chat server
the syntax is the following :

@ip1

@ip2

### users
	
users is a file where all admins are registered with this syntax :

name1 password1

name2 password2

admins must be created from the admin panel and cannot be added directly to this file since passwords are salted and encrypted with bcrypt.

If the users file is empty, the first try to get **/nc-login** will put you in setup mode and you'll have to set the first admin.

### log

type :	INFO_CONN -- datetime -- client @IP

type :	NEW_MSG -- datetime -- client @IP -- client name -- client message

type :	ADMIN_MSG -- datetime -- admin name -- admin message

type :	ADMIN_BAN -- datetime -- @IP banned

admin name is auto set by authentication but **can be changed by the admin**, 
this is the only case where you can see an admin name that is not registered
in the [users] file in your logs.

## Functions

If you want to custom the node-chat-js, you need to know the functions and events under the hood :

### Client

**node-chat-js client IO event listeners**
* updateClientNumber
* message
* messageFromAdmin
* banned
```javascript
// LISTENERS

// update client number
socket.on('updateClientNumber', (data) => {
    getClientNumber(data.clientNumber);
});

// add message from server to DOM
socket.on('message', (data) => {
    appendMessageToDOM(data.name, data.message, null, data.time);
});

// add message from server (admin) to DOM
socket.on('messageFromAdmin', (data) => {
    appendMessageToDOM(data.name, data.message, 'admin', data.time);
});

// banned from chat
socket.on('banned', (data) => {
    appendMessageToDOM('BAN' , data.message, 'banned', data.time);
});
```
**node-chat-js clients can only emit message**
```javascript
// EMITTER

socket.emit('message', { name : ncName.value, message : ncMessage.value });
```

### Server

**node-chat-js server IO event emitters / listeners**
* connection
* message
* messageFromAdmin
* messageForAdmin
* isAdmin
* banIp
* addAdmin
* disconnect

**check index.js file fore more**


Made by Nicolas Coutable - 2018
