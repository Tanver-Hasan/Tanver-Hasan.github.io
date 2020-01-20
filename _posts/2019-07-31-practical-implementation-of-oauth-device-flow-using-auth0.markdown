---
title: Practical Implementation of Oauth Device Flow using Auth0
layout: post
category: oauth2.0
date: '2019-07-31 23:16:29'
comments: true
---

In this blog post, I am going to dicuss Oauth Device Flow and how it can be applied to secure IOT devices. This blog post will not focuss on the details   how device flow work under the hood. Rather I will develop a CLI to demonastrate  how to implement it and secure IOT devices. 

Excited! Lets get started. At first, lets discuss the scenario where we need this and why we need device flow. In recent years, IOT security is a growing concern. We have seen many incidets that malicious users took over home appliacnes and started either listening your bed room conversation or live streaming your activities. Pretty scary. Most of these does not have any built in security and does not have an interface. Even some deivices have very small screen and few buttons at most. So, what we do to secure this devices. Lets step back and think how we secure our software application. Mostly, in web, we use oauth2.0 and openID connect to seuce our applications and APIs. Some manged to use Resource Owner Password Grant to secure these devices. However, as we already know, ROPG is not really a secure approach and it is not recommend by oauth2.0 working group. And there are many more constranints.  To address this issue, the Oauth2.0 provided an extension to this flow to secure input constrained devics (IOT) which known as Device Authorization flow. As I mentiond, I will not focus on the details on its architecture. So, I am attaching some good documentation on it which explains the flow pretty well. 

[https://auth0.com/docs/flows/concepts/device-auth
https://www.identityserver.com/articles/an-introduction-to-the-oauth-device-flow](https://auth0.com/docs/flows/concepts/device-auth
https://www.identityserver.com/articles/an-introduction-to-the-oauth-device-flow)

If you have the patience to read more, please do visit the this link to read the official specification. 
[https://tools.ietf.org/html/draft-ietf-oauth-device-flow-15](https://tools.ietf.org/html/draft-ietf-oauth-device-flow-15)


Once you have the basic understanding, you should be able to move forward with the implementation. As authorization server, I am going to use Auth0 

Step 1: Create an application 
* 			Login to Auth0 
* 			Create a Native Application 
* 			Enable connections. 
* 			Enable device Grant types. (Application settings -> Advanced -> Grant Types-> Check device )

Step 2: Configure Device User code settings

This settings is located tenant advanced settings.  [https://auth0.com/docs/dashboard/guides/tenants/configure-device-user-code-settings](https://auth0.com/docs/dashboard/guides/tenants/configure-device-user-code-settings)
Before jumping on building actual CLI, first get an understanding and get it working with simple API request using CURL. Steps we will follow are described  in the docuemnation: 

* Request device code (Device Flow): Request a device code that the user can use to authorize the device.
* Request device activation (Device Flow): Request that the user authorize the device using their laptop or smartphone.
* Request Tokens (Device Flow): Poll the token endpoint to request a token.
* User authorization (Browser Flow): The user authorizes the device, so the device can receive tokens.
* Receive Tokens (Device Flow): After the user successfully authorizes the device, receive tokens.
* Call your API (Device Flow): Use the retrieved Access Token to call your API.
* Refresh Tokens (Device Flow): Use a Refresh Token to request new tokens when the existing ones expire.

Reqest Device code : 

```
curl --request POST \
    --url "https://login.tanverhasan.com/oauth/device/code" \
    --header 'Content-Type: application/json' \
    --data '{"client_id":"zEHWunlajvzQKAYQ54o0D5sZ3iWz9BvE","scope":"openid","audience":"https://api.timesheet.com/"}'
		
```

It should return a response like below. 

```

{
    "device_code": "ja-1vlJg5no3QVUclkSxZsK_",
    "user_code": "800-591-307-087",
    "verification_uri": "https://login.tanverhasan.com/activate",
    "expires_in": 900,
    "interval": 5,
    "verification_uri_complete": "https://login.tanverhasan.com/activate?user_code=800-591-307-087"
}

```

Now open the verification_uri_complete URL in phone or laptop browser which should redirect you to the authorization server. If you open just  the verification_uri, you will need to enter the user_code manually.  After that, it  rediret the user to   auth0 Hosted login page and show your appropriate connetion enabled previously. I have enabled google and username and password connetion. Thereore, you should be able to login and authorization server  

Once your finish the authentication,  then you can make an http reqeust /oauth/token endpoint to for token. 

```
curl --request POST \
    --url "https://login.tanverhasan.com/oauth/token" \
    --header 'Content-Type: application/json' \
    --data '{"client_id":"zEHWunlajvzQKAYQ54o0D5sZ3iWz9BvE","device_code":"2Y8gqDjT2-tit-xSRCbGJ_rX","grant_type":"urn:ietf:params:oauth:grant-type:device_code"}'
```
Pass the device_code obtained in the previous steps. If everything goes ok, you should receive the token. Wonderfull! We got our initial demenostration. 

Lets move on to develop and CLI. I am going to use Golang to develop the CLI. My previous post contains how to get up and running with golang. In this project, I am going to use go modules, so , I do not need to worry about go path. 

* Open a terminal
*  Create a directory. 
*  Run `go mod init {project name}`
*  Run `touch main.go`
*  Open the directory in a IDE. 

Here is the complete source code. 

```
package main

import (
	"bytes"
	"encoding/json"
	"flag"
	"fmt"
	"io/ioutil"
	"log"
	"net/http"
	"os"
	"time"

	"github.com/mdp/qrterminal"
)

var loginCmd bool
var m map[string]interface{}

func main() {
	fmt.Println("Test Device Credentials")
	flag.BoolVar(&loginCmd, "login", true, "Authenticate device")
	flag.Parse()

	if loginCmd == true {
		login()

	}
}

func login() {
	url := "https://login.tanverhasan.com/oauth/device/code"

	reqBody, err := json.Marshal(map[string]string{
		"client_id": "zEHWunlajvzQKAYQ54o0D5sZ3iWz9BvE",
		"scope":     "openid",
		"audience":  "https://api.timesheet.com/",
	})
	if err != nil {
		log.Fatalln(err)
	}
	res, _ := http.Post(url, "application/json", bytes.NewBuffer(reqBody))

	defer res.Body.Close()
	body, _ := ioutil.ReadAll(res.Body)
	//fmt.Println(string(body))
	var f interface{}
	json.Unmarshal(body, &f)

	m := f.(map[string]interface{})
	deviceCode := fmt.Sprintf("%v", m["device_code"])
	fmt.Println("Prinint Device Code " + deviceCode)
	str := fmt.Sprintf("%v", m["verification_uri_complete"])

	generateQrCode(str)

	for {
		time.Sleep(20 * time.Second)
		fmt.Println("Pooling loop")
		resBody, err := poolingToken(deviceCode)

		if err != nil {
			fmt.Print(err)
		}

		fmt.Println(string(resBody))

		if len(resBody) > 0 {
			fmt.Println(string(resBody))
			var ff interface{}
			json.Unmarshal(resBody, &ff)
			result := ff.(map[string]interface{})
			accessToken := fmt.Sprintf("%v", result["access_token"])
			fmt.Println(accessToken)
			if len(accessToken) > 5 {
				break
			}
		}

	}
}

func generateQrCode(url string) {

	qrterminal.Generate(url, qrterminal.L, os.Stdout)
}

func poolingToken(deviceCode string) ([]byte, error) {
	// fmt.Println("Calling pooloing token function")
	url := "https://login.tanverhasan.com/oauth/token"

	fmt.Println("Priting device code in poling function :" + deviceCode)

	reqBody, err := json.Marshal(map[string]string{
		"grant_type":  "urn:ietf:params:oauth:grant-type:device_code",
		"device_code": deviceCode,
		"client_id":   "zEHWunlajvzQKAYQ54o0D5sZ3iWz9BvE",
	})

	if err != nil {
		fmt.Println(err)
	}

	res, err := http.Post(url, "application/json", bytes.NewBuffer(reqBody))
	defer res.Body.Close()
	body, err := ioutil.ReadAll(res.Body)

	return body, err

}

```
Let me know describe what I did .  At first , as I am developing CLI, flag is built in libary in go for that purpose.  User can login to the CLI by executing the --loing command. 

`main()` function builds the CLI option. Flag is a built in library in go for parsing command line arguments. In this project, I am going to just develop one argument to demonastrate the device flow. The argument name is `-login`. If the -login is provided, the main() funciton calls login function. 

`login()` function takes care of executing the device flow as described eairlier. At first, I developed a valid JSON  using Josn.Marsal function and then passed the JSON object in the HTTP request body when calling` /oauth/device/code` endpoint. For http request, I am usig built in HTTP library in go. ( I am not going to delv in how to write. Rather, I will describe from high level and the purpose of each function. ). Once the response `/oauth/device/code` returns the response, the next things we need to open this `verification_uri_complete` or `verification_uri ` in the browser to login. The difference between this two param is that verification_uri does not populate the device code when navigated to the web page. However, `verification_uri_complete` automatically populate the device code. For simplicity, I provided a function to generate qr code for `verification_uri_complete`. Therefore, user does not have to write the URL manually. They can just scan the QR code.  Once we verify the user idnetity, we need to obtain the token form server. As the device flow is designed for input constrat device, there is no way to know when to make request to obtain the toke. Instead, desinged proposed to constally pool the token from the server.

`poolingToken()` method makes an HTTP reqest to /oauth/token eendpoint in every 30 sec to check user has finised authentiation. Once user finished the authentication, the sever will return HTTP status code 200 and returns the token.


{% if site.disqus.shortname %}
  {% include disqus_comments.html %}
{% endif %}