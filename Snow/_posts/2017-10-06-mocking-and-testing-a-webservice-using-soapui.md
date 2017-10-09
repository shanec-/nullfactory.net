---
layout: post
title: Mocking and Testing a WebService using SoapUI 
category: REST, SOAP, WCF
---
I have recently been working on an implementation of a WCF client interacts with a remote SOAP web service. Unfortunately this service did not have a development version that I could interactively test my requests against.

Armed with just the WSDL and the schema for expected responses, I needed to find a way to mock the remote service in order to confirm the functionality of the client. This is where SOAP UI comes in - SOAP UI is tool that allows me to point it to service contract (WSDL or REST) and mock the responses that each of the actions would generate. And finally provides the functionality to host and serve these mocked responses so that they can be consumed by different clients.

I will be using the freely available [http://www.webservicex.net/globalweather.asmx](http://www.webservicex.net/globalweather.asmx) web service to demonstrate the process.

<!--excerpt-->

Let's start off with a single request:

1. Open up the SOAP UI application and add connection to the SOAP service. Do this by navigating to the `File > New SOAP Project` menu option.

2. Provide a name for the project and set the url to the WSDL file to [http://www.webservicex.net/globalweather.asmx?wsdl](http://www.webservicex.net/globalweather.asmx?wsdl) . Keep the `Create Requests` check box checked: 

   ![Add new SOAP project](/images/posts/MockSoapUI/10_AddNewSoapProject.png)

3. Now let's create a mock version of our web service. Right click on the `GetCitiesByCountry` method and select `Add to MockService` option in the context menu.

   ![Add to mock service](/images/posts/MockSoapUI/20_AddToMockService.png)

4. Provide it with a name, and select `Yes` to open it within the editor.

   ![Enter name for service](/images/posts/MockSoapUI/30_EnterNameForService.png)

   ![Open mock service editor](/images/posts/MockSoapUI/40_OpenMockServiceEditor.png)

5. In the resulting dialog, paste the response you would like to receive when a request is made.

   ![Mocked response](/images/posts/MockSoapUI/50_MockedResponse.png)

6. Now that we have the response setup, let's start up the mock service to serve up our requests. Do this by double clicking the `WeatherMockService` node to bring up the service status dialog and clicking on the `Start` (play icon) button.

   ![Mocking service started](/images/posts/MockSoapUI/70_MockingServiceStarted.png)

7. Now let's see if our response works. The easiest way to do this is to right-click on the response that we previously populated, and on the context-menu select the `Open Request` option.

   ![Mocking service test](/images/posts/MockSoapUI/75_MockServiceTest.png)

8. Either select the pre-generated request, or select the option to create a new one. 

9. Modify the resulting request as necessary. 

10. Once ready, click the submit (play icon) button to send the request.

  ![Mocking service test result](/images/posts/MockSoapUI/78_MockServiceTestResult.png)

11.  Confirm that you have received the previously setup request.

Now we have very basic mock and responses working, let's make it a little bit more complex. In my live system, any time the service method is called it would reply with a different response subject to some external variables. I wanted to mock the same functionality so that I can better test how my client code reacts to the different sequences. 

I did a bit of research and found another individual who had the same requirement: 

- [https://stackoverflow.com/questions/32975240/soapui-mock-service-custom-sequence-of-responses](https://stackoverflow.com/questions/32975240/soapui-mock-service-custom-sequence-of-responses)

This approach allows us to orchestrate the responses that are returned. In this particular example there are no smarts built in apart from the service responding to the requests in sequence.

Let's setup our sequence of responses:

1. Create a second response for the request. 

  ![Mock second response](/images/posts/MockSoapUI/80_MockSecondResponse.png)

2. Provide it with a new response body as done previously.

  ![Mock second response body](/images/posts/MockSoapUI/90_MockSecondResponseBody.png)

3. Next, right-click on the `GetCitiesByCountry` method in the mock service and select `Show MockOperation Editor` in the resulting dialog.

  ![Mock operation editor](/images/posts/MockSoapUI/100_MockOperationEditor.png)

4. In the resulting dialog,  choose the `SEQUENCE` as the `Dispatch` method.

  ![Sciprt editor](/images/posts/MockSoapUI/110_ScriptEditor.png)

5. Paste the following script in the script section. This script controls the order in which the responses are returned. Update the `myRespList` array to change this sequence.

        // get the list from the context
        def myRespList = context.myRespList
       
        // if list is null or empty reinitalize it
        if(!myRespList || !myRespList?.size){   
        	// list in the desired output order using the response names that your
        	// create in your mockservice
        	myRespList = ["Response 2","Response 1"]  
        }
        // take the first element from the list
        def resp = myRespList.take(1)[0]
        
        // update the context with the list without this element
        context.myRespList = myRespList.drop(1)
        
        // return the response
        log.info "-->"+resp
        return resp

  ![Final script](/images/posts/MockSoapUI/120_FinalScript.png)

6. Test out the response by clicking on the test script (play button) just above the editor. SOAP UI responds with showing us the first request that would be sent.

  ![Script returned response 2](/images/posts/MockSoapUI/140_ScriptTestRun2.png)

7. Testing the script again, and receive the second response. Good, the script seems to be doing its job.

  ![Script returned response 1](/images/posts/MockSoapUI/130_ScriptTestRun1.png)

8. Let's do one final test with an actual request to make sure its working as expected. 

  ![Request first run](/images/posts/MockSoapUI/160_RequestSecondRun.png)

9. And run the request again to receive the second response.

  ![Request second run](/images/posts/MockSoapUI/150_RequestFirstRun.png)

## Final Thoughts
I love the fact that this tool provides the right combination of ease of use and functionality. And allows a new developer to quickly setup the basics and then extend the test scenarios as they become more complex.

This is going into my developer tool belt for sure.

## References
- [Your First SoapUI Project | Getting started with SoapUI](https://www.soapui.org/getting-started/your-first-soapui-project.html)
- [10 Tips for the SoapUI Beginner | Getting started with SoapUI](https://www.soapui.org/getting-started/10-tips-for-the-soapui-beginner.html)
- [groovy - SoapUI Mock Service Custom Sequence of Responses - Stack Overflow](https://stackoverflow.com/questions/32975240/soapui-mock-service-custom-sequence-of-responses)