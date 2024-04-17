# Solving and Bypassing Amazon captcha Waf Captcha Automatically When scraping


This article provides insights into AWS WAF Captcha and its role in preventing automated activities such as web scraping, spam, and credential stuffing. AWS WAF Captcha, which stands for "Completely Automated Public Turing test to tell Computers and Humans Apart," presents puzzles or challenges to verify the user's human identity. While it may not be foolproof against advanced machine learning techniques, it effectively deters less sophisticated bot traffic and adds an extra layer of security.

## What is AWS WAF Captcha and how does it work
To understand what is AWS WAF Captcha, I suggest that we first understand what is this captcha and how it works

AWS WAF includes a feature called CAPTCHA that helps determine if a user is human or a bot. CAPTCHA stands for "Completely Automated Public Turing Test to tell Computers and Humans Apart." It presents puzzles or challenges that users need to solve to verify their human identity and prevent activities like web scraping, spam, and credential stuffing. While some automated techniques can solve CAPTCHA puzzles using machine learning and artificial intelligence, CAPTCHA still serves as a useful tool to deter less sophisticated bot traffic and make large-scale operations more difficult.

AWS WAF generates random CAPTCHA puzzles and regularly updates them to ensure unique challenges for users. The CAPTCHA script collects client data to confirm human interaction and prevent replay attacks.

Each CAPTCHA puzzle includes standard controls for users to request a new puzzle, switch between audio and visual puzzles, access instructions, and submit their solution. The puzzles are designed to be accessible, supporting screen readers, keyboard controls, and contrasting colors.

So if a requisition from a visitor looks suspicious, Amazon displays the CAPTCHA in the form of a puzzle. Users also have the option to complete an audio CAPTCHA. If the CAPTCHA is successfully broken, the user is redirected back to the page. If the CAPTCHA is not successfully completed, the user is offered a new puzzle until the CAPTCHA is solved.

## Different Types of AWS WAF Captcha when Web Scraping
An example of a picture grid puzzle is shown below. The puzzle asks you to select all pictures in the grid that contain objects of a particular type.


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/bvzroaag4nk9b2ac34m0.png)


The screenshot below shows an example puzzle that requires you to determine the end point of a car's path in a drawing.


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/aynkpvy0r0jkmq8635uy.png)


The following shows the display for the audio puzzle choice. Audio CAPTCHAs use background noise superimposed on the voice. Just like puzzles, audio CAPTCHAs can be solved automatically.


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/wam0zeqtjv7zcw7cwkos.png)




## Solving Amazon WAF with CapSolver 
In many scenarios, Amazon randomly generates and even rotates puzzles to ensure that users face distinct challenges. More critically new puzzle types are also added periodically to remain effective against automated methods. As well as through the puzzles themselves, of course, Amazon also uses user data collection to verify that tasks are being completed by humans to prevent repeat requests. So in most cases, Amazon CAPTCHA is able to block simple bot access, but there is a way for us to achieve compliant automated puzzle solving, and that's through [CapSolver](https://www.capsolver.com/) is a service that provides solutions for captcha recognition. It offers various task types for different captcha systems, including Amazon WAF. 

Capsolver provides two CAPTCHA solving services that can help you to easily solve Amazon WAF. One service is using Capsolver's [API](https://docs.capsolver.com/guide/api-server.html), and the other one is downloading the [Extension](https://docs.capsolver.com/guide/extension/introductions.html). In the API, the task type used by Amazon WAF is `AwsWafClassification`.

Next, follow my steps to see how to implement an automated solution to Amazon's captcha in web scraping, it's simple, let's dig in!

### Step 1 Login

You can [sign up](https://dashboard.capsolver.com/passport/register
) for CapSolver and get access to our CAPTCHA service, which is currently supported with a free trial.

### Step 2 Get your free API!
Once you have registered, you can obtain your api key from the home page panel.  


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ike3tvwgsuyzpfmxqnrp.png)



### AwsWafCaptcha: solving AwsWaf

- tip Create the task with the [createTask](../api-createtask.md) method and get the result with
the [getTaskResult](../api-gettaskresult.md) method.


The task type `types` that we support:

- `AntiAwsWafTask`  this task type require your own proxies.
- `AntiAwsWafTaskProxyLess`  this task type don't require your own proxies.

### Step 3 Create Task

Create a recognition task with the [createTask](../api-createtask.md) method.

#### Task Object Structure

| Properties     | Type     | Required | Description                                                                                            |
|----------------|----------|----------|--------------------------------------------------------------------------------------------------------|
| type           | String   | Required | `AntiAwsWafTask` <br> `AntiAwsWafTaskProxyLess`                                                        |
| websiteURL     | String   | Required | The URL of the page that returns the  captcha info                                                     |
| awsKey         | Optional | Required | When the status code returned by the websiteURL page is 405, you need to pass in awsKey                |
| awsIv          | Optional | Required | When the status code returned by the websiteURL  page is 405, you need to pass in awsIv                |
| awsContext     | Optional | Required | When the status code returned by the websiteURL  page is 405, you need to pass in awsContext           |
| awsChallengeJS | Optional | Required | When the status code returned by the websiteURL  page is 202, you only need to pass in awsChallengeJs; |
| proxy          | String   | Required | Learn [Using proxies](../api-how-to-use-proxy)                                                         |

- warning
If the obtained token is not available, it may be because of the ip
please try to use the AntiAwsWafTask mode to pass in your own proxy.

#### Example Request

``` json
POST https://api.capsolver.com/createTask
Host: api.capsolver.com
Content-Type: application/json

{
    "clientKey": "YOUR_API_KEY",
    "task": {
        "type": "AntiAwsWafTask", //Required
        "websiteURL": "https://efw47fpad9.execute-api.us-east-1.amazonaws.com/latest", //Required
        "awsKey": "",
        "awsIv": "",
        "awsContext": "",
        "awsChallengeJS": "",
        "proxy": "http:ip:port:user:pass" // socks5:ip:port:user:pass // Optional
    }
}
```

After you submit the task to us, you should receive in the response a 'Task id' if it's successfull. Please
read [errorCode: full list of errors](../api-createtask.md) if you didn't receive the task id. For more information, you can
also refer to this blog post [How to solve aws amazon captcha token](https://www.capsolver.com/blog/All/how-to-solve-aws-amazon-captcha-token)

#### Example Response

``` json
{
    "errorId": 0,
    "errorCode": "",
    "errorDescription": "",
    "taskId": "61138bb6-19fb-11ec-a9c8-0242ac110006"
}
```

## Getting Results

After you have the taskId, you need to submit the taskId to retrieve the solution. Response structure is explained
in [getTaskResult](../api-gettaskresult.md).

Depending on the system load, you will get the results within the interval of `5s` to `30s`

#### Example Request

``` json
POST https://api.capsolver.com/getTaskResult
Host: api.capsolver.com
Content-Type: application/json

{
    "clientKey": "YOUR_API_KEY",
    "taskId": "61138bb6-19fb-11ec-a9c8-0242ac110006"
}
```

#### Example Response

```json
{
  "errorId": 0,
  "taskId": "646825ef-9547-4a29-9a05-50a6265f9d8a",
  "status": "ready",
  "solution": {
    "cookie": "223d1f60-0e9f-4238-ac0a-e766b15a778e:EQoAf0APpGIKAAAA:AJam3OWpff1VgKIJxH4lGMMHxPVQ0q0R3CNtgcMbR4VvnIBSpgt1Otbax4kuqrgkEp0nFKanO5oPtwt9+Butf7lt0JNe4rZQwZ5IrEnkXvyeZQPaCFshHOISAFLTX7AWHldEXFlZEg7DjIc="
  }
}
```

## Solving AwsWafCaptcha using Capsolver SDK:

::: code-group

```python
# pip install --upgrade capsolver
# export CAPSOLVER_API_KEY='...'

import capsolver

# capsolver.api_key = "..."
solution = capsolver.solve({
    "type": "AntiAwsWafTask",
    "websiteURL": "https://efw47fpad9.execute-api.us-east-1.amazonaws.com/latest",
    "proxy": "ip:port:user:pass"
})
```

```go [golang]
package main

import (
	"fmt"
	capsolver_go "github.com/capsolver/capsolver-go"
	"log"
)

func main() {
	// first you need to install sdk
	//go get github.com/capsolver/capsolver-go
	//export CAPSOLVER_API_KEY='...' or
	//capSolver := CapSolver{ApiKey:"..."}

	capSolver := capsolver_go.CapSolver{}
	solution, err := capSolver.Solve(map[string]any{
		"type": "AntiAwsWafTaskProxyLess",
		"websiteURL": "AntiAwsWafTask",
		 "proxy":"ip:port:user:pass"
	})
	if err != nil {
		log.Fatal(err)
		return
	}
	fmt.Println(solution)
}
```
```


## Conclusion
In conclusion, AWS WAF Captcha is an essential tool in distinguishing between human users and bots, protecting websites from malicious activities. By generating random and regularly updated puzzles, AWS WAF ensures unique challenges for users. Capsolver, a captcha recognition service, offers solutions for solving Amazon WAF Captcha, making it possible to automate puzzle-solving tasks during web scraping. However, it's important to note that while automated solutions exist, captchas continue to evolve to stay effective against automated methods, highlighting the ongoing battle between security measures and adversaries seeking to solve them.
