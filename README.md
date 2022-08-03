# Submit a HTML form to Google Sheets

How to submit a simple HTML form to a Google Sheet using only HTML and JavaScript. *Updated for Google Script Editor 2022 Version.*

This example shows how to set up a mailing list form that sends data to Google Sheets but you can use it for any sort of data.

### 1. Set up a Google Sheet

1. Go to [Google Sheets](https://sheets.google.com) and create a new sheet. This is where we'll store the form data.
2. Set the following headers in the first row:

|   |     A     |   B   | C | ... |
|---|:---------:|:-----:|:-:|:---:|
| 1 | Date | Email | Name   |     |


### 2. Create a Google App Script

<img src="https://sheetmonkey.io/img/guides/1-app-script.gif?ts=1" width="450" />

Click on `Extensions -> Apps Script`. This will open new Google Script. Rename it to something like _"Mailing List"_.

<img src="https://sheetmonkey.io/img/guides/2-script-editor.png" width="450" />

Replace the `myFunction() { ...` section with the following code snippet:

```js
// Original code from https://github.com/jamiewilson/form-to-google-sheets
// Updated for 2021 and ES6 standards

const sheetName = 'Sheet1'
const scriptProp = PropertiesService.getScriptProperties()

function initialSetup () {
  const activeSpreadsheet = SpreadsheetApp.getActiveSpreadsheet()
  scriptProp.setProperty('key', activeSpreadsheet.getId())
}

function doPost (e) {
  const lock = LockService.getScriptLock()
  lock.tryLock(10000)

  try {
    const doc = SpreadsheetApp.openById(scriptProp.getProperty('key'))
    const sheet = doc.getSheetByName(sheetName)

    const headers = sheet.getRange(1, 1, 1, sheet.getLastColumn()).getValues()[0]
    const nextRow = sheet.getLastRow() + 1

    const newRow = headers.map(function(header) {
      return header === 'Date' ? new Date() : e.parameter[header]
    })

    sheet.getRange(nextRow, 1, 1, newRow.length).setValues([newRow])

    return ContentService
      .createTextOutput(JSON.stringify({ 'result': 'success', 'row': nextRow }))
      .setMimeType(ContentService.MimeType.JSON)
  }

  catch (e) {
    return ContentService
      .createTextOutput(JSON.stringify({ 'result': 'error', 'error': e }))
      .setMimeType(ContentService.MimeType.JSON)
  }

  finally {
    lock.releaseLock()
  }
}
```

Save the project before moving on to the next step.

### 3. Run the initialSetup function

<img src="https://sheetmonkey.io/img/guides/3-initial-setup.png" width="450" />

You should see a modal asking for permissions. Click `Review permissions` and continue to the next screen.

Because this script has not been reviewed by Google, it will generate a warning before you can continue. You must click the "Go to Mailing List (Unsafe)" for the script to have the correct permissions to update your form.

<img src="https://sheetmonkey.io/img/guides/5-warning.png" width="450" />

After giving the script the correct permissions, you should see the following output in the script editor console:

<img src="https://sheetmonkey.io/img/guides/6-success.png" width="450" />

Now your script has the correct permissions to continue to the next step.

### 4. Add a trigger for the script

<img src="https://sheetmonkey.io/img/guides/7-triggers.png" width="450" />

Select the project "Triggers" from the sidebar and then click the `Add Trigger` button.

In the window that appears, select the following options:

- Choose which function to run: `doPost`
- Choose which deployment should run: `Head`
- Select event source: `From spreadsheet`
- Select event type: `On form submit`

<img src="https://sheetmonkey.io/img/guides/8-trigger-config.png" width="450" />

Then select "Save".

### 5. Publish the project

Now your project is ready to publish. Select the `Deploy` button and `New Deployment` from the drop-down.

<img src="https://sheetmonkey.io/img/guides/9-deploy.gif" width="450" />

Click the "Select type" icon and select `Web app`. 

In the form that appears, select the following options:

- Description: `Mailing List Form` (This can be anything that you want. Just make it descriptive.)
- Web app → Execute As: `Me`
- Web app → Who has access: `Anyone`

Then click `Deploy`.

**Important:** Copy and save the web app URL before moving on to the next step.

### 6. Configure your HTML form

Create a HTML form like the following, replacing `YOUR_WEBAPP_URL` with the URL you saved from the previous step.

```html
<form 
  method="POST" 
  action="YOUR_WEBAPP_URL"
>
  <input name="Email" type="email" placeholder="Email" required>
  <input name="Name" type="text" placeholder="Name" required>
  <button type="submit">Send</button>
</form>
```

Now when you submit this form from any location, the data will be saved in the Google Sheet. 🥳

<img src="https://sheetmonkey.io/img/guides/10-working-form.gif" width="450" />

- **Please note:** The input names are _case sensitive_. They must match the same casing as the script. More here: https://github.com/levinunnink/html-form-to-google-sheet/issues/3#issuecomment-1054464935

_Note: If you want to intercept the submit event so the user isn't redirected to the webapp, [you can do this by attaching a JavaScript event listener to the form submission and creating the `POST` request yourself](https://codepen.io/levinunnink-the-bashful/pen/YzxPyoG?editors=0010)._

## Issues? 

If you want to submit your HTML forms to Google Sheets without using App scripts, try a free service like [Sheet Monkey](https://sheetmonkey.io), which allows you to do submit forms to Google Sheets without any backend code.

## Thanks

Thanks to the following articles and projects that inspired this guide

- https://medium.com/@dmccoy/how-to-submit-an-html-form-to-google-sheets-without-google-forms-b833952cc175
- https://github.com/jamiewilson/form-to-google-sheets
