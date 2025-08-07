# 🔍 Google Sheets Form Submission - Debug Guide

## Current Issue: Form submissions not appearing in Google Sheets

### Step 1: Test the Form Submission (Browser Console)

1. **Open your website** at `http://localhost:3000`
2. **Open Browser Developer Tools** (F12)
3. **Go to Console tab**
4. **Submit the form** with test data
5. **Check console output** - you should see:
   ```
   Response: {status: "success", message: "Data saved successfully"}
   ```

**If you see errors in console:**
- CORS errors → Apps Script deployment issue
- Network errors → Wrong URL or Apps Script not accessible
- JSON parse errors → Apps Script code issue

### Step 2: Check Apps Script Deployment

1. **Go to your Apps Script project**
2. **Click "Deploy" → "Manage deployments"**
3. **Verify settings:**
   - Type: Web app
   - Execute as: Me
   - Who has access: **Anyone** (CRITICAL!)

4. **If settings are wrong:**
   - Click "New deployment"
   - Choose "Web app"
   - Set "Who has access" to **Anyone**
   - Copy the new URL and update your HTML

### Step 3: Test Apps Script Directly

1. **In Apps Script editor, add this test function:**
```javascript
function testFormSubmission() {
  var testData = {
    postData: {
      contents: JSON.stringify({
        name: "Test User",
        email: "test@example.com",
        timestamp: new Date().toISOString(),
        source: "Debug Test"
      })
    }
  };
  
  var result = doPost(testData);
  console.log(result.getContent());
}
```

2. **Run the test function**
3. **Check execution log** - should show success
4. **Check your Google Sheet** - should have test data

### Step 4: Verify Sheet Configuration

1. **Check sheet name** in your Apps Script:
   ```javascript
   var sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName("Sheet1");
   ```

2. **Verify your actual sheet name matches** (case-sensitive!)

3. **Check column headers** (Row 1):
   - A1: Name
   - B1: Email  
   - C1: Timestamp
   - D1: Source

### Step 5: Enhanced Apps Script with Logging

Replace your `doPost` function with this enhanced version:

```javascript
function doPost(e) {
  console.log('doPost called');
  console.log('Request data:', e);
  
  try {
    // Check if postData exists
    if (!e.postData) {
      console.log('No postData found');
      return ContentService
        .createTextOutput(JSON.stringify({ status: "error", message: "No data received" }))
        .setMimeType(ContentService.MimeType.JSON);
    }
    
    console.log('postData contents:', e.postData.contents);
    
    // Get the active spreadsheet and sheet
    var sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName("Sheet1");
    console.log('Sheet found:', sheet.getName());
    
    // Parse the incoming JSON data
    var data = JSON.parse(e.postData.contents);
    console.log('Parsed data:', data);
    
    // Extract form data
    var name = data.name || 'Not provided';
    var email = data.email || '';
    var timestamp = data.timestamp || new Date().toISOString();
    var source = data.source || 'E-book Signup';
    
    console.log('Extracted data:', {name, email, timestamp, source});
    
    // Validate email
    if (!email || !email.includes('@')) {
      console.log('Invalid email:', email);
      return ContentService
        .createTextOutput(JSON.stringify({ status: "error", message: "Invalid email" }))
        .setMimeType(ContentService.MimeType.JSON);
    }
    
    // Add new row with form data
    sheet.appendRow([name, email, timestamp, source]);
    console.log('Row added successfully');
    
    // Return success response
    return ContentService
      .createTextOutput(JSON.stringify({ status: "success", message: "Data saved successfully" }))
      .setMimeType(ContentService.MimeType.JSON);
      
  } catch (error) {
    console.log('Error occurred:', error.toString());
    // Return error response
    return ContentService
      .createTextOutput(JSON.stringify({ status: "error", message: error.toString() }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}
```

### Step 6: Check Apps Script Execution Log

1. **In Apps Script, go to "Executions"**
2. **Submit your form**
3. **Check for new execution entries**
4. **Click on execution to see logs**

### Step 7: Alternative Testing Method

If nothing works, try this simple test:

1. **Create a simple test page:**
```html
<!DOCTYPE html>
<html>
<body>
<button onclick="testSubmission()">Test Submission</button>

<script>
async function testSubmission() {
  const data = {
    name: "Test User",
    email: "test@example.com",
    timestamp: new Date().toISOString(),
    source: "Direct Test"
  };
  
  try {
    const response = await fetch('YOUR_APPS_SCRIPT_URL_HERE', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(data)
    });
    
    const result = await response.json();
    console.log('Result:', result);
    alert('Result: ' + JSON.stringify(result));
  } catch (error) {
    console.error('Error:', error);
    alert('Error: ' + error.message);
  }
}
</script>
</body>
</html>
```

## Most Common Issues:

1. **Apps Script not deployed as "Anyone" access**
2. **Wrong sheet name in Apps Script**
3. **CORS issues due to deployment settings**
4. **Apps Script URL is incorrect**
5. **Sheet doesn't have proper headers**

## Next Steps:

Follow each step above and report back with:
1. Console output when submitting form
2. Apps Script execution log results
3. Any error messages you see

This will help identify the exact issue! 🔍