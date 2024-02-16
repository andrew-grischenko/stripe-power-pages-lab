# stripe-power-pages-lab
A hands on lab to explore native integration of Microsoft Power Pages and Stripe


## Register a new Stripe account

1. Navigate to the [Stripe registration page](https://dashboard.stripe.com/register), and set up a new Stripe account. Remember the password you used! 
1. On the Stripe dashboard, select the Developers section at the top left and the API keys tab, then under the Developers heading. 
1. There are two keys listed: a publishable key and a secret key (masked). You will need these in the next steps, so keep this tab open. 

Enable Stripe integration in Power Pages

	1. Navigate to https://make.powerpages.microsoft.com/ and login using provided by the facilitator credentials. 
	2. After login, you will see a Power Pages site pre-created for you. Click Edit to enter Power Pages Studio.
	3. In the Power Pages Studio select Set up section on the left panel and then find the "External apps (preview)" section 
	4. You will see 2 integrations available - DocuSign and Stripe. We are interested in the latter today, so select "Install" in the "Stripe" row below. Click "Start Installation" to confirm and wait until it completes. 
	5. Once "Installing…" status turned into "Installed", click "Enable" button and enter Secret key and Publishable key. Copy and paste those from the respective fields in the Stripe account you created in the previous section. Click Save to save the changes.

Create a payment form

	1. In the Power Pages Studio, select "Data" section on the left panel and click on the cogs icon next to Data header. Select a predefined (and empty at the moment) solution "Power Platform Bootcamp 2024". This will ensure that the changes made in the next steps are associated with that Dataverse solution and not the Default solution.
	2. Click "+ Table" button and create a new table called "MyOrder" (or other name of your choice). Select "Autonumber" for the Name field type and add a prefix "ORD-" or another as you wish.
	3. Add 2 fields to the created table:
		a. "Customer name": Text
		b. "Amount": Currency
	4. In the Forms tab of the table, find and select "Information" form and add to it the fields created on the above step ("Customer name" and "Amount").
	5. Create a new form for the table, and name it "Payment". Add to this form a field "Amount" and make it read only. 
	

Create a payment page

	1. In the Power Pages Studio, click "Sync" button on the top right corner. This is necessary for the changes made in the previous section to take effect. 
	2. Select "Pages" section and create a brand new page and call it "Order payment" or similar. 
	3. In the new page add a new section and select "Multistep form" component from the list of section components available. Name it "Payment form" or similar and click OK to save it.
	4. On the Multistep component select "+ Add step" and set the step properties and click OK:
		a. Step name: "Order details"
		b. Choose a table: "MyOrder"
		c. Select a form: "Information"
		d. Data from this form: "Creates a new record" (on the "More options" section). 
	5.  You will see the message "Set the permissions on this form…" with a button "New permission" on the right. Click it and set the permission properties:
		a. Permission to: Create, Read, Update
		b. Roles: "Anonymous users" and "Authenticated users" are selected. 
	6. Please note, this lab is just a demo and doesn't authenticate users. In the real world scenario, you would need to remove "Anonymous users" from the permissions list. Leave it there for the purpose of this lab.
	7. Save the permissions record. 
	8. On the Multistep component select "+ Add step" to add another step and set the step properties:
		a. Step name: "Payment"
		b. Choose a table: "MyOrder"
		c. Select a form: "Payment"
		d. Data from this form: "Updates an existing record" (on the "More options" section). 
	9. On the App integrations select "Enable digital payments". If this option is disabled, make sure that you've done step #1 (Sync). 
		a. If that was done and the option is still not available, you may need to return to https://make.powerpages.microsoft.com/  and click "Admin center" and restart website. 
	10. In the "Choose amount field" select the "Amount" filed and click OK. 
	11. Now you should see the Stripe Credit card payments form appear on the newly created step, hooray! 

Test the changes made

	1. Click "Preview" button and a new tab with the website shall open. 
	2. Select "Order payment" page and enter some test information to validate that the integration works end to end:
		a. Customer name: any
		b. Amount: any amount greater than 0
	3. Click Next and enter the payment details. Please do NOT enter real card details. While it will not create a charge, it's not a good practice for testing from the security point of view!
		a. Card number 4242424242424242 (this is a test card number). 
		b. Expiration: 01/30 (or any other future date)
		c. CVC: 123 (or any 3 digit number)
		d. Other details as required
	4. Click "Pay now" and in a second or few you should see a message "Your payment for … was successfully processed."
	5. Navigate to you Stripe account (which still may be open in your browesr tab) and select "Payments" section on the left. You should see a successful payment created a few seconds ago with the amount you specified. 
	6. Click on the payment record and in the payment details scroll to the Events and logs section and examine the API request and responses and data that were initiated during this test. 

Add failed payments handling (optional)

From here: https://cloudminded.blog/2024/01/09/how-to-improve-stripe-payments-in-power-pages/ 

	1. Navigate to https://make.powerpages.microsoft.com/  and select "Solutions" tab. 
	2. Find the solution "Power Platform Bootcamp 2024", click to enter the solution content. 
	3. Select  "Add existing \ Table" and select "Payments" table. This table was included into the environment by the Power Pages integration when you selected "Enable" at the earlier steps in the lab.  
	4. Add new column "Payment status code” (single line of text).
	5. In the solution, I created a new instant cloud flow with a trigger of type “HTTP Request”. It’s set to be available to anyone who calls it, and I defined an empty schema.
	6. For the next step, you need to know the structure of the webhook payload, so for now, let’s just add a “Compose” action to store the payload (body) of the request and return to it after we set up Stripe webhook.
	7. Save the new flow and then copy the “HTTP POST URL” value – this is what Stripe will call on the payment events we will define next.
	8. Now, go to the Stripe developer section and find the “Webhook” section, where you should already see a few webhooks registered by the out-of-the-box integration. We will add another one – paste the URL to the flow created earlier into the “Enpoint URL” field and then select “payment_intent.payment_failed” as the event type to track.
	9. Create a successful test payment - follow the steps in the previous section "Test the changes made"
	10. Navigate back to the solution "Power Platform Bootcamp 2024" and find the flow created at step 5 and select it history. Check the "Compose" action output.
	11. Remove the “Compose” action; it served us well. As the next step of the cloud flow, we will find the record in the Payment table that has the “Payment identifier” field value as the payment intent reference from the webhook request data: triggerBody()?['data']?['object']?['id']
	12. Then, the other step is to update this record with the payment event information we received:
		○ Row ID – the first record of the previous List operation (you may want to do it better by checking the record count, etc): first(outputs('List_rows')?['body/value'])?['pp_paymentid']
		○ Payment status – Failed
		○ Payment status code – the failure_code from the event data: first(triggerBody()?['data']?['object']?['charges']?['data'])?['failure_code']
		○ Payment status reason – the failure_message from the event data: first(triggerBody()?['data']?['object']?['charges']?['data'])?['failure_message']
	• Now simulate a failed payment event. Enter the card number as 40000000000XXXXX and see a declined message
	• Navigate back to the Payments table, find the record for the payment and see the details in the Payment status   code and status code reason.
![image](https://github.com/andrew-grischenko/stripe-power-pages-lab/assets/17406795/790554dd-4616-4892-9f8b-36323c7a7edd)

