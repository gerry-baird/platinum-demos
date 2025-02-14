---
title: Managing events and APIs from a unified environment <br/>300-level live demo
layout: demoscript
banner: images/managing-events-apis-script-banner-10-27-22.jpg
---


<span id="top"></span>

<details markdown="1">

<summary>Introduction</summary>

It is becoming increasingly common to need to expose data both synchronously, using APIs, and asynchronously, using events. This enables us to cater to the unique needs and characteristics of the different consumers of the data.

Some consumers need direct access to retrieve or change the data and will therefore prefer APIs. Others may not want to be directly tied to the availability and performance limitations of the backend system, and are only interested in being asynchronously notified of changes, which is therefore a good use case for events. Even a single consumer may need to retrieve data in different styles for different scenarios. As a data provider, however, you shouldn’t need two separate systems to achieve this. In this demo we’ll look at how events can be shared and governed in the same way as APIs.

The scenario focuses on an airline’s flight information system. It provides the flight data for applications such as airport ‘Flight Boards’, showing departures and arrival times. The data is also made available to other applications via APIs that are catalogued in IBM API Connect.

There are a number of businesses around the airport, such as those providing food for onboard catering services, flight maintenance, and teams such as the ground crew that are directly affected by flight delays. They would benefit from being notified as soon as those delays occur, and would be better suited to an event-based interaction pattern, than to that of an API. 

We will show how IBM's solution enables application developers to discover and use API or event-based interaction patterns depending on the needs of their solution. Specifically, we will look at how Apache Kafka event topics and APIs can be surfaced alongside one another in API Connect, and governed in exactly the same way. 

Let’s get started!

(Demo intro slides <a href="https://ibm.box.com/s/9bzas80jz80s3xechdl642ag80f4qjuz" target="_blank" rel="noreferrer">here</a>)

(Printer-ready PDF of demo script <a href="https://ibm.box.com/s/6siayii6qbus0v8m5pi293ntgphzbwh7" target="_blank" rel="noreferrer">here</a>)

<br/>

</details>

<span id="existingAPI"></span>

<details markdown="1">

<summary>1 - Existing API-driven information system - without events</summary>

<br/>

| **1.1** | **Explore the Flight Information Manager UI** |
| :--- | :--- |
| **Narration** | Let’s first take a look at the flight information system itself so we understand the data we’re dealing with. The user interface for the flight information system is known as the Flight Information Manager. It shows us the current flights, and also provides us with a mechanism to add a delay to the anticipated departure time of a flight. |
| **Action** &nbsp; 1.1.1 | Show the FlightBoard Manager UI. <br/><img src="images/300-eem-demo-1-1-1.png" width="500" /> |
| **Narration** | The Flight Information Manager uses a REST API available on the Flight Information Data Store. This API is only directly available to the Flight Information Manager user interface since it has the ability to perform sensitive functions, such as displaying the delayed arrival time of a flight. |

| **1.2** | **Explore the existing REST API** |
| :--- | :--- |
| **Narration** | There are other systems, such as the FlightBoards around the airport that also need access to flight data but do not need access to the whole API. For example, the FlightBoard should not have access to the API function to 'delay' a flight. An API management capability (IBM API Connect) is used to expose selected data and functions from the API in a Developer Portal. The developers of other applications can browse the various APIs available at the airport, and self-subscribe to use them in their applications. This means that their usage of the API can be tracked, controlled (e.g. rate limited) and indeed revoked. Let's take a quick look at the exposed REST API for flight information. |
| **Action** &nbsp; 1.2.1 | Go to the API management Portal and show the exposed REST API for flight information, noting that there is only a GET operation. |
| **Action** &nbsp; 1.2.2 | Show the IBM API Connect **Developer Portal** tab in the browser. <br/><img src="images/300-eem-demo-1-2-2.png" width="800" /> |
| **Action** &nbsp; 1.2.3 | Click **API Products**. <br/><img src="images/300-eem-demo-1-2-3.png" width="800" /> |
| **Action** &nbsp; 1.2.4 | Click **Flight API**. <br/><img src="images/300-eem-demo-1-2-4.png" width="800" /> |
| **Action** &nbsp; 1.2.5 | Click **Flight API 1.0.0**. <br/><img src="images/300-eem-demo-1-2-5.png" width="800" /> |
| **Action** &nbsp; 1.2.6 | Click the **GET /flights** operation. <br/><img src="images/300-eem-demo-1-2-6.png" width="800" /> |
| **Narration** | Note that we’ve been able to explore this API right down to what operations are available, and even example data that would be returned. However, we cannot make calls on this API because we have not yet requested access to it. API Connect ensures only known consumers of the API can use it. |

| **1.3** | **Display the FlightBoard** |
| :--- | :--- |
| **Narration** | The FlightBoard is one of the consumers of this flight information API. It retrieves the data every five minutes to get the latest flight information. |
| **Action** &nbsp; 1.3.1 | Bring up the FlightBoard UI. <br/><img src="images/300-eem-demo-1-3-1.png" width="500" /> |

| **1.4** | **Explore the update rate of the FlightBoard** |
| :--- | :--- |
| **Narration** | Note that if we delay a flight using the Flight Information Manager, although it shows the new time immediately in that user interface, the FlightBoard only picks up the change when it next polls the API, several minutes later. |
| **Action** &nbsp; 1.4.1 | Show both FlightBoard (ie, the display in the airport, shown on the left) and the Flight Information Manager (ie, the master flight information record, shown on the right) on screen at the same time. Both screens are showing the same information for now.<br/><img src="images/300-eem-demo-1-4-1.png" width="800" /> |
| **Action** &nbsp; 1.4.2 | On the Flight Information Manager, click on one of the **Delay this flight** (1) buttons, enter **60** (2) for the minutes to be delayed, and click **OK** (3).<br/><img src="images/300-eem-demo-1-4-2.png" width="800" /> |
| **Action** &nbsp; 1.4.3 | Show that the flight departure time has been updated on the Flight Information Manager only (you may need to refresh the browser).<br/><img src="images/300-eem-demo-1-4-3.png" width="800" /> |
| **Action** &nbsp; 1.4.4 | Remember that the board updates every five minutes. If required, wait to show that FlightBoard is updated. |
| **Narration** | Clearly, we could reduce the poll interval to something that might be more responsive, but as we reduce this interval, we are increasing the load on the Flight Information Data Store. Note that there could be many other systems also requiring this information in a timely fashion (eg., taxi companies, ground crew systems, airlines). If they were all to poll with short time intervals, the Flight Information Data Store would quickly become overwhelmed. The Flight Information Data Store could of course implement performance measures such as caching, but it would end up having to cater to constantly increasing demands of consumers, in terms of both performance, and availability. <inline-notification text="This is a good time to discuss how common this situation is across scenarios in other industries. APIs are a commonly used way to provide data, but do not make it easy to receive data changes in a timely way. Examples might include fraudulent payment attempts, exceptional stock price movements, fire or other hazard alerts, refrigerator temperature thresholds etc. "></inline-notification> |

<br/>

**[Go to top](#place1)**

</details>

<span id="Events"></span>

<details markdown="1">

<summary>2 - Using events to deliver real-time information</summary>

<br/>

| **2.1** | **Introduce events, Kafka, and IBM Event Streams** |
| :--- | :--- |
| **Narration** | Modern applications are turning to Apache Kafka to distribute information using a publish/subscribe pattern. In Kafka’s terminology, a topic is a stream of events related to the same subject. Events are published to specific Kafka topics, then consumers decide (via subscription) which topics they would like to receive events from. Publish/subscribe capabilities are certainly not new and messaging products have provided this pattern for decades. However, Apache Kafka has risen to popularity due to its ability to retain an event history, making it well suited to certain application patterns. IBM Cloud Pak for Integration provides both messaging (IBM MQ) and also a production strength implementation of Apache Kafka (IBM Event Streams). IBM Event Streams provides a Kubernetes operator that enables rapid deployment of Apache Kafka clusters, including a graphical user interface that simplifies familiarization. |
| **Action** &nbsp; 2.1.1 | Show the **IBM Event Streams** tab in the browser. <br/><img src="images/300-eem-demo-2-1-1.png" width="800" /> |

| **2.2** | **View events on Kafka topic** |
| :--- | :--- |
| **Narration** | For the simplicity of the demo, we've already configured our flight information system to publish events to a Kafka topic when we delay a flight.  |
| **Action** &nbsp; 2.2.1 | Click **Topics** (1) and then select **flight-delays** (2).<br/><img src="images/300-eem-demo-2-2-1.png" width="800" /> |
| **Action** &nbsp; 2.2.2 | The events already emitted will be displayed. Select the top event (1) and show this corresponds to the flight you previously delayed (2). <br/><img src="images/300-eem-demo-2-2-2.png" width="800" /> |
| **Action** &nbsp; 2.2.3 | Show that as you delay additional flights these display in the event stream. |
| **Narration** | Note that the events for each delay we create are present in the event history, and there is a very clear representation of their sequential order based on the 'Offset' value. These will remain here, regardless of whether subscribers read them or not, and this is one of the key differences between Apache Kafka and messaging capabilities such as IBM MQ. Events can only be removed administratively, either by archiving those older than a certain time period, or a more sophisticated mechanism known as 'log compaction' which is useful when at least one event for each data record must be preserved. |

| **2.3** | **Connect an application to the Kafka cluster** |
| :--- | :--- |
| **Narration** | Applications consuming events from a Kafka cluster need to know the location of the bootstrap Kafka broker for initiation of their connection. You will notice that that IBM Event Streams only provides 'internal' connections. That means you can only connect from within the OpenShift cluster. This connection can be used, for example, by the flight information system to publish events. |
| **Action** &nbsp; 2.3.1 | Click **Connect to this topic**.<br/><img src="images/300-eem-demo-2-3-1.png" width="800" /> |
| **Action** &nbsp; 2.3.2 | Click **Internal** (1), then save the **endpoint information** (2) as the Kafka listener URL. We will use this later when configuring IBM Event Endpoint Manager. Note that we do not need to download any certificates because the demo Kafka cluster has security switched off for simplicity. <br/><img src="images/300-eem-demo-2-3-2.png" width="800" /> |
| **Narration** | The consumers ​of flight delay events (​such as the FlightBoard) will be external to the OpenShift cluster. We ​could do basic external exposure by configuring an 'external' listener ​in the screens we've just explored, but this provides only limited control when we are exposing Kafka to consumers beyond our immediate sphere. We are going to look at a much more powerful way of exposing a topic, using IBM Event Endpoint Manager. Since this is in the same OpenShift cluster as this Kafka cluster, the internal listener will be sufficient as IBM Event Endpoint Manager will do the external exposure for us, then route to the internal connection. |

<br/>

**[Go to top](#place1)**

</details>

<span id="makingAvailable"></span>

<details markdown="1">

<summary>3 - Making events available to external consumers</summary>

<br/>

| **3.1** | **Manage the Async API** |
| :--- | :--- |
| **Narration** | IBM Event Endpoint Manager performs exactly the same role for asynchronous interfaces as IBM API Connect does for synchronous APIs. It allows us, for example, to make a selection of Kafka topics available in a catalogue and display them in a portal such that a developer can easily discover and subscribe to use them. It then protects the actual Kafka cluster by controlling access to it via a gateway. Let’s add our Kafka topic to the IBM Event Endpoint Manager catalogue.  |
| **Action**  3.1.1 | Open the Cloud Pak for Integration Platform Navigator tab and click on the **menu**, expand the **Design** section, and select **APIs** (1). <br/><img src="images/300-eem-demo-3-1-1.png" width="800" /> <br/> |
| **Action**  3.1.2 | Click the **ademo** entry. <br/><img src="images/300-eem-demo-3-1-1-1.png" width="800" /> <br/> |
| **Action**  3.1.3 | In the **API Connect** page, if a login screen is presented, select **Cloud Pak User Registry**. <br/><img src="images/prep-image212.png" width="800" /> <br/> |
| **Action** &nbsp; 3.1.4 | Click **Develop APIs and products**.<br/><img src="images/300-eem-demo-3-1-2.png" width="800" /> |
| **Action** &nbsp; 3.1.5 | Select **Add** in the top right, and select **API**. <br/><img src="images/300-eem-demo-3-1-3.png" width="800" /> |
| **Action** &nbsp; 3.1.6 | Select **AsyncAPI** (1) and click **Next** (2).<br/><img src="images/300-eem-demo-3-1-4.png" width="800" /> |
| **Action** &nbsp; 3.1.7 | Upload the **Flight delay access.yaml** file downloaded in the preparation steps and click **Next**.<br/><img src="images/300-eem-demo-3-1-5.png" width="800" /> |
| **Narration** | The next page is where you can enter a vast array of optional detail about the API, that will ultimately be published to the catalog. However, we do not need to add any further details at this stage. |
| **Action** &nbsp; 3.1.8 | Click **Next**  |
| **Narration** | We will choose to publish the topic straight away, which will automatically create a Product and publish that within the Sandbox catalog. By default it will secure the topic in the same way we would a synchronous API, using an API Key and Secret. This enables us to hide what might be a more complex security model used on your Kafka cluster (such as mTLS). Note that AsyncAPIs exposed by the event gateway are always protected with TLS communication, even if the underlying Kafka broker is not, as is the case in this demo. |
| **Action** &nbsp; 3.1.9 | Click **Edit API**.<br/><img src="images/300-eem-demo-3-1-7.png" width="800" /> |
| **Action** &nbsp; 3.1.10 | Check the **Inactive** box which will change to Active.<br/><img src="images/300-eem-demo-3-1-8.png" width="800" /> |
| **Narration** | Managing granular access to topics would normally require a Kafka administrator to create and maintain multiple access control lists (ACLs). IBM Event Endpoint Manager enables consumers to self-administer their own access to topics. The Kafka administrator need only set up access between the Kafka cluster and the event gateway. From that point onward, the person deciding which topics the consumers can subscribe to no longer needs to be a Kafka specialist. |

<br/>

**[Go to top](#place1)**

</details>

<span id="realTimeApp"></span>

<details markdown="1">

<summary>4 - Consuming real-time events in an application</summary>

<br/>

| **4.1** | **Discover the AsyncAPI from an app developer’s perspective** |
| :--- | :--- |
| **Narration** | Let’s get back to our airport scenario. There are a number of businesses around the airport that are affected by flight delays and would benefit from being notified as soon as they occur. In other words, they would be better suited to the event-based interaction pattern, than to that of an API. Examples would include teams providing food for onboard catering services, flight maintenance crews, and ground crew. <br/> <br/>In this section we will consider an application used by such teams that notifies users when a flight has been delayed. The developers of that application will use API Connect's developer portal to discover the 'flight-delays' event topic and incorporate it into their solution. |
| **Action** &nbsp; 4.1.1 | Go to the API Connect Developer Portal tab and click on **API Products**. Note that if you are already on the API Products page, you may need to refresh the browser page as you just added a new AsyncAPI product. <br/><img src="images/300-eem-demo-4-1-1.png" width="800" /> |
| **Action** &nbsp; 4.1.2 | Explain the two entries, as highlighted in the narration above. <br/><img src="images/300-eem-demo-4-1-2.png" width="800" /> |
| **Narration** | Note that APIs are gathered together into 'Products'. You could gather multiple topics under one Product, and they could even originate from different Kafka clusters. Access is then defined at the Product level, so you can easily subscribe to a whole suite of events or APIs in one go. Simple products are automatically created for development purposes and that’s what we will use in this demonstration. |
| **Action** &nbsp; 4.1.3 | Click **flight-delays auto product** (the automatically-generated product we created earlier).<br/><img src="images/300-eem-demo-4-1-3.png" width="800" /> |
| **Action** &nbsp; 4.1.4 | Click **flight-delays**.<br/><img src="images/300-eem-demo-4-1-4.png" width="800" /> |
| **Narration** | This next page will show documentation we provided when we added this API to the catalogue. From this page you will find the values needed for connectivity. Note that the bootstrap server is that of the event gateway, not that of the original Kafka cluster. You will also find some example code samples to simplify application development. |
| **Action** &nbsp; 4.1.5 | Explain the screen, as highlighted in the narration above.<br/><img src="images/300-eem-demo-4-1-5.png" width="800" /> |

| **4.2** | **Subscribe to the AsyncAPI** |
| :--- | :--- |
| **Narration** | From the detail we’ve found so far on the API, it looks ideal for providing the notifications for our application, so we decide to collate the connection details and subscribe to the AsyncAPI. Just to reinforce what’s happening here – these are the connection details to the event gateway provided by IBM Event Endpoint Management, not to the actual underlying Kafka cluster. However, the event gateway 'looks' just like a Kafka cluster to the consuming application. |
| **Action** &nbsp; 4.2.1 | Click the arrow next to flight delays in the overview on the left, then select **Subscribe (operation)**. <br/><img src="images/300-eem-demo-4-2-1.png" width="800" /> |
| **Action** &nbsp; 4.2.2 | Scroll down on the right hand side to the **Properties** section (1) and save the **client.id** and **bootstrap.servers** (2). <br/><img src="images/300-eem-demo-4-2-2.png" width="800" /> |
| **Action** &nbsp; 4.2.3 | Click **Get access** in the top right. If for any reason you are not logged into the Portal, this is the point at which you will be forced to log in. <br/><img src="images/300-eem-demo-4-2-3.png" width="800" /> |
| **Narration** | Products are made available to subscribers through 'Plans'. This gives us the opportunity to have different policies depending on the plan that is chosen. If APIs are monetized APIs, each plan might have a different cost, or some plans might be only available to selected user groups. |
| **Action** &nbsp; 4.2.4 | Select the **Default** pricing plan to continue. <br/><img src="images/300-eem-demo-4-2-4.png" width="800" /> |
| **Narration** | As a developer, you may be writing several applications, each using different APIs. From a security perspective you might not want all applications you work with to have access to all of the APIs that you have subscribed to. For this reason, you can create an 'Application' within IBM Event Endpoint Manager for each of your applications, and individually subscribe them only to the Plans on the Products they need. Each application will get its own unique Key and Secret such that it can be identified when attempting to access the APIs.<br/><br/>Let’s assume we are a developer working on the application used by the ground crew, and we would like to improve it to show alerts when flights are delayed. You will need to create an ‘Application’ in the portal to represent the ground crew’s application. This will create an identity (a set of credentials) that the ground crew’s application will use to access the event topic. |
| **Action** &nbsp; 4.2.5 | Click **Create Application** on the right.<br/><img src="images/300-eem-demo-4-2-5.png" width="800" /> |
| **Action** &nbsp; 4.2.6 | Name the application **Ground Crew App** (1) and click **Save** (2). <br/><img src="images/300-eem-demo-4-2-6.png" width="800" /> |
| **Action** &nbsp; 4.2.7 | You will now see a dialog showing the **Key** and **Secret** for this application to access the API. <inline-notification text="It is very important that you save these, as they cannot be retrieved later. "></inline-notification><img src="images/300-eem-demo-4-2-7.png" width="800" /> |
| **Action** &nbsp; 4.2.8 | Close this dialog then select your newly created application on the left. <br/><img src="images/300-eem-demo-4-2-8.png" width="800" /> |
| **Action** &nbsp; 4.2.9 | Review the details then click **Next**. Your subscription will be complete. <br/><img src="images/300-eem-demo-4-2-9.png" width="800" /> |

| **4.3** | **Consume the events** |
| :--- | :--- |
| **Narration** | We now have all the credentials we need for our application to subscribe to the Kafka topic via IBM Event Endpoint Manager’s gateway. All we need now is an application to consume the events. We’re not going to create an actual application. We’re just going to give you an example, built using the sample code from the Portal, that picks up the events from the topic. <br/><br/>Recall that the IBM Event Endpoint Manager gateway always enforces TLS encryption on external requests, so our consumer will need to have a copy of the gateway’s public TLS certificate. We will download this certificate and store it somewhere that the consuming application can retrieve at runtime. |
| **Action** &nbsp; 4.3.1 | Install the server certificate from the event gateway endpoint ready for the client application. There is a script to do this:<br/><br/><code>./install-certificate.sh</code><br/><img src="images/300-eem-demo-4-3-1.png" width="500" /> |
| **Narration** | The example in our demo is simply a java consumer that simulates the way the application would receive Kafka events, such that we can see them arriving in real time. |
| **Action** &nbsp; 4.3.2 | The script that runs the consumer code itself is called ./setup-client-app.sh and is one of the files you downloaded in the beginning. This takes five arguments: <br/><br/> 1) **Target Namespace:** this is normally cp4i <br/><br/>2) **Kafka Client Id:** this is retrieved from the Developer Portal in the properties section<br/><img src="images/300-eem-demo-4-3-2-clientid.png" width="800" /> <br/>3) **Gateway Username:** this is the API **Key** that you copied in this step.<br/><img src="images/300-eem-demo-4-3-2-apikey.png" width="800" /> <br/>4) **Gateway Password:** this is the API **Secret** also shown in the screenshot above.<br/><img src="images/300-eem-demo-4-3-2-apisecret.png" width="800" /><br/>5) **Flight Number:** this is the flight number you want to monitor. This needs to be within quotes, e.g. "EZY 6005" |
| **Action** &nbsp; 4.3.3 | Run the following command, replacing each of the parameters:<br/><br/>**./setup-client-app.sh &lt;TargetNamespace&gt; &lt;KafkaClientId&gt; &lt;GatewayUsername&gt; &lt;GatewayPassword&gt; &lt;FlightNumber&gt;**<br/><br/>Here is an example. Remember, this needs to be customized with your own values for each of the parameters:<br/><br/>**./setup-client-app.sh cp4i 64a66394-84dd-4b8b-b78c-202e24125632 8515b170f10b6937c4ba89f1c6eaf325 746f7e1f89415723d00366acd0574410 “EZY 6005”** <br/><br/>This will automatically build a container and run it on Red Hat OpenShift. <br/><inline-notification text="Pick any flight in the FlightBoard Manager UI. When entering the flight number, be sure to type the quotation marks directly in your Terminal window. Copying and pasting from another text editor may result in no delays being reported."></inline-notification> |
| **Action** &nbsp; 4.3.4 | In the Flight Information Manager, click the button to delay the flight you entered in the previous step. Enter **180** minutes (1), then click **OK** (2). <br/><img src="images/300-eem-demo-4-3-4.png" width="500" /> <br/> |
| **Action** &nbsp; 4.3.5 | Now return to the terminal window where the script was run, and you will see a message. <br/><br/>"Alert: Flight EZY 6005 has been delayed by 180 minutes" <br/><img src="images/300-eem-demo-4-3-5.png" width="500" /><br/> |
| **Narration** | We see the following message, which of course in real life would appear on the application used by the catering staff or ground crew. <br/><br/>"Alert: Flight EZY 6005 has been delayed by 180 minutes"<br/><br/>Notice that it takes a while before the FlightBoard shows the delay, since it is still using API polling to get flight information. |
| **Action** &nbsp; 4.3.6 | Show the flight board message. <br/><img src="images/300-eem-demo-4-3-5b.png" width="800" /><br/> |

<br/>

**[Go to top](#place1)**

<br/>


</details>

<span id="Summary"></span>

<details markdown="1">

<summary>Summary</summary>

In this demo we showed how IBM API Connect enables application developers to discover and use API or event-based interaction patterns depending on the needs of their solution. Specifically, we looked at how Apache Kafka event topics and APIs could be surfaced alongside one another in the same catalogue, and governed in exactly the same way.

We explored a use case that was better served by listening to Kafka events than by polling synchronous APIs. Furthermore, the event-based solution would reduce the load on the backend system, as the Kafka event stream removes the need to poll the API for flight delay information. 

A key focus of the demo was to show how events could be shared, and governed in the same way as APIs, extending API management into “event endpoint management”. The topics were discoverable in a catalogue from which the consumer could self-subscribe to receive the events. The event topic was then securely exposed using IBM's unique event gateway that transparently routes the Kafka protocol to the underlying Kafka brokers.

Thank you for attending today’s presentation.

**[Go to top](#place1)**
<br/>

</details>