# Export your Whatsapp contacts in Firefox 

With pure Javascript which you enter in the Dev Console of Firefox you can export your contacts along with the last message
into a CSV File.

## Instructions

- Open Whatsapp in Firefox
- Launch Developer Console (Press F12)
- Navigate to the "Console" Tab
- Copy and paste the Javascript of script.js into the Console and press Enter

The script should no go through all your chats and export Name, Phone Number, last Message
At the end a Tab separated file is generated and downloaded, which can be used with Excel / Libre Office
The script can be stopped by entering the following command into the Console: 

```js
window.__stopWhatsappExport = true; 
```

