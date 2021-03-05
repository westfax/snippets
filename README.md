# WestFax Snippets
Random useful snippets for Fax API developement from WestFax HIPAA Compliant fax

## Snippet A: HTML Send via jQuery and Base64 inline file.

### Sending a Fax via API using jQuery and Base64 encoded documents.

Sending a fax via the API is easy if you have an actual fax file but what if your document is stored as a blob or object in a database. How do you securely send a fax with an in-memory document without saving it as a file?

 When would this situation come up? If you have a file saved in a database blob object and don’t want to (or can’t) encode the file into a temporary file.

  Today we will demonstrate sending a fax using jquery and HTML.

  Prereqs:

1.  Web Browser with developer tools (Chrome)
    
2.  Code editor (or notepad)
    
3.  WestFax account
    

  

Before we start if you are going to run this code in a local browser you will get a CORS error. There is a plugin that allows you to turn off CORS. You will see an error like this:

  

>Access to XMLHttpRequest at 'https://api2.westfax.com/REST/Fax_SendFax/json' from origin 'null' has been blocked by CORS policy: Response to preflight request doesn't pass access control check: No 'Access-Control-Allow-Origin' header is present on the requested resource.
15:31:44.592

  

There are browser extension which allow you to turn off this security feature temporarily for just this page. I like [this extension](https://chrome.google.com/webstore/detail/cross-domain-cors/mjhpgnbimicffchbodmgfnemoghjakai?hl=en) as it just works. Be sure to turn it off when you go to other sites.

  
Open the html file in your code editor of choice and you need to replace the values on line 44-47.
```javascript
//Replace these with your values
const user = "test@test.com";
const pass = "test1234";
const productId = "0000000-0000-0000-00000000000";
const yourFaxNumber = "2223334444"; //use your fax number
```


Now you want to save the file to a location on your hard drive. You will also want to make sure to allow Cross-domain queries on your location web browser. Normally this would be bad but this is pretty commonplace for debugging and coding react/angular/es application. You can use the extension above to disable Cross site scripting and allow jQuery to send a request to the API.

When you run this script correctly you should see the following in the console with a unique hash of course:
```json
   "Success": true,
   "Result": "6b98617b-932c-46d7-8800-a5325640c91b"
```
If you have any questions reach out to us at develop@westfax.com and we'll give you a hand.
