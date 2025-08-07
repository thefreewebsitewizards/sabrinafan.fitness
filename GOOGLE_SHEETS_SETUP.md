# Google Sheets Integration Setup Guide

This guide will help you set up a serverless form submission system using Google Apps Script and Google Sheets.

## ✅ What You'll Build

- **No backend server required** - Google Apps Script acts as your serverless backend
- **Direct Google Sheets integration** - Form data saves directly to your spreadsheet
- **Real-time data collection** - Submissions appear instantly in your sheet
- **Free solution** - No hosting costs or server maintenance

## 🔧 Step-by-Step Setup

### Step 1: Create a Google Sheet

1. Go to [Google Sheets](https://sheets.google.com)
2. Create a new spreadsheet
3. Name it "E-book Signups" (or any name you prefer)
4. In row 1, add these column headers:
   - **A1:** Name
   - **B1:** Email
   - **C1:** Timestamp
   - **D1:** Source

### Step 2: Open Apps Script Editor

1. In your Google Sheet, click **Extensions > Apps Script**
2. Delete any default code in the editor
3. Paste the following code:

```javascript
function doPost(e) {
  try {
    // Get the active spreadsheet and first sheet
    var sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName("Sheet1");
    
    // Extract form data from parameters (FormData)
    var name = e.parameter.name || 'Not provided';
    var email = e.parameter.email || '';
    var timestamp = e.parameter.timestamp || new Date().toISOString();
    var source = e.parameter.source || 'E-book Signup';
    
    // Log received data for debugging
    console.log('Received data:', {name: name, email: email, timestamp: timestamp, source: source});
    
    // Validate email
    if (!email || !email.includes('@')) {
      return ContentService
        .createTextOutput('Error: Invalid email')
        .setMimeType(ContentService.MimeType.TEXT);
    }
    
    // Check for duplicate emails (optional)
    var lastRow = sheet.getLastRow();
    if (lastRow > 1) { // Only check if there are data rows beyond the header
      var existingEmails = sheet.getRange(2, 2, lastRow - 1, 1).getValues();
      for (var i = 0; i < existingEmails.length; i++) {
        if (existingEmails[i][0] === email) {
          return ContentService
            .createTextOutput('Error: Email already exists')
            .setMimeType(ContentService.MimeType.TEXT);
        }
      }
    }
    
    // Add new row with form data
    sheet.appendRow([name, email, timestamp, source]);
    console.log('Data saved successfully');
    
    // Return success response
    return ContentService
      .createTextOutput('Success: Data saved successfully')
      .setMimeType(ContentService.MimeType.TEXT);
      
  } catch (error) {
    console.log('Error:', error.toString());
    // Return error response
    return ContentService
      .createTextOutput('Error: ' + error.toString())
      .setMimeType(ContentService.MimeType.TEXT);
  }
}

// Test function (optional)
function testDoPost() {
  var testData = {
    postData: {
      contents: JSON.stringify({
        name: "Test User",
        email: "test@example.com",
        timestamp: new Date().toISOString(),
        source: "Test Submission"
      })
    }
  };
  
  var result = doPost(testData);
  console.log(result.getContent());
}
```

### Step 3: Deploy as Web App ⚠️ CRITICAL FOR CORS

1. Click **Deploy > New deployment**
2. Click the gear icon ⚙️ next to "Type" and select **Web app**
3. **IMPORTANT:** Fill in the deployment settings EXACTLY as shown:
   - **Description:** E-book Form Submission
   - **Execute as:** Me (your email)
   - **Who has access:** Anyone ⚠️ **MUST be "Anyone" for CORS to work**
4. Click **Deploy**
5. **Grant permissions** when prompted:
   - Click "Review permissions"
   - Choose your Google account
   - Click "Advanced" if you see a warning
   - Click "Go to [Your Project Name] (unsafe)"
   - Click "Allow"
6. **Copy the Web App URL** - you'll need this for your frontend

**🚨 TROUBLESHOOTING CORS ERRORS:**
If you get CORS errors ("Access to fetch... has been blocked"):
1. Go back to **Deploy > Manage deployments**
2. Click the pencil icon ✏️ to edit
3. **Verify "Who has access" is set to "Anyone"**
4. Click **Deploy** (this creates a new version)
5. **Use the NEW URL** in your frontend code

### Step 4: Update Your Frontend Code

1. Open your `index.html` file
2. Find this line in the JavaScript:
   ```javascript
   const GOOGLE_SCRIPT_URL = 'https://script.google.com/macros/s/YOUR_SCRIPT_ID/exec';
   ```
3. Replace `YOUR_SCRIPT_ID` with the actual URL you copied from Step 3

### Step 5: Test Your Setup

1. Open your website
2. Fill out the e-book signup form
3. Submit the form
4. Check your Google Sheet - you should see the new submission!

## 🔧 Advanced Configuration

### Custom Sheet Name

If you want to use a different sheet name:

1. In your Google Sheet, rename the tab (right-click > Rename)
2. Update the Apps Script code:
   ```javascript
   var sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName("Sheet1"); // Change "Sheet1" to your actual sheet name
   ```

### Email Notifications

To get email notifications for new submissions, add this to your Apps Script:

```javascript
// Add this after sheet.appendRow([name, email, timestamp, source]);
GmailApp.sendEmail(
  'your-email@gmail.com',
  'New E-book Signup',
  `New signup from ${name} (${email}) at ${timestamp}`
);
```

### Data Validation

The script already includes:
- Email format validation
- Duplicate email checking
- Error handling

## 🚀 Benefits of This Approach

✅ **No server required** - Completely serverless
✅ **Free hosting** - Google Apps Script is free
✅ **Real-time updates** - Data appears instantly
✅ **Automatic backups** - Google handles data security
✅ **Easy to manage** - View/edit data directly in Google Sheets
✅ **Scalable** - Handles thousands of submissions
✅ **No maintenance** - Google manages the infrastructure

## 🔍 Troubleshooting

### Common Issues:

1. **"Script not found" error**
   - Make sure you deployed the script as a web app
   - Check that the URL is correct

2. **Permissions error**
   - Redeploy the script
   - Make sure "Who has access" is set to "Anyone"

3. **Data not appearing**
   - Check the Apps Script logs (View > Logs)
   - Verify the sheet name matches your code

4. **CORS errors**
   - Make sure you're using `mode: 'no-cors'` in your fetch request

### Testing the Script

You can test your Apps Script directly:

1. In the Apps Script editor, select the `testDoPost` function
2. Click the "Run" button
3. Check your Google Sheet for the test data

## 📊 Viewing Your Data

Your Google Sheet will automatically collect:
- **Name:** Visitor's name
- **Email:** Email address
- **Timestamp:** When they signed up
- **Source:** Where the signup came from

You can:
- Sort and filter the data
- Create charts and reports
- Export to other formats
- Share with team members

## 🚨 TROUBLESHOOTING COMMON ISSUES

### CORS Error: "Access to fetch... has been blocked"

**This is the most common issue!** If you see this error in your browser console:

```
Access to fetch at 'https://script.google.com/macros/s/...' from origin 'http://localhost:3000' 
has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present
```

**Solution:**
1. **Check deployment settings:**
   - Go to Apps Script → Deploy → Manage deployments
   - Click the pencil icon ✏️ to edit your deployment
   - **Verify "Who has access" is set to "Anyone"** (not "Anyone with Google account")
   - Click "Deploy" to create a new version
   - **Copy the NEW URL** and update your frontend

2. **Verify your Apps Script URL:**
   - Make sure you're using the Web App URL (ends with `/exec`)
   - NOT the Apps Script editor URL

3. **Test the Apps Script directly:**
   - Open the Web App URL in a new browser tab
   - You should see "Error: Invalid email" (this means it's working)

### Form Submits But No Data in Sheet

1. **Check sheet name:** Verify `getSheetByName("Sheet1")` matches your actual sheet tab name
2. **Check column headers:** Ensure row 1 has: Name, Email, Timestamp, Source
3. **Check Apps Script logs:** In Apps Script editor → Executions tab

### "Unauthorized" Error (401)

- Your Apps Script deployment permissions are incorrect
- Redeploy with "Execute as: Me" and "Who has access: Anyone"

### Still Having Issues?

1. **Test with this simple HTML file:**
```html
<!DOCTYPE html>
<html>
<body>
<form id="testForm">
  <input name="name" value="Test User" required>
  <input name="email" value="test@example.com" required>
  <button type="submit">Test Submit</button>
</form>

<script>
document.getElementById('testForm').addEventListener('submit', async (e) => {
  e.preventDefault();
  const formData = new FormData(e.target);
  formData.append('timestamp', new Date().toISOString());
  formData.append('source', 'Test');
  
  try {
    const response = await fetch('YOUR_APPS_SCRIPT_URL_HERE', {
      method: 'POST',
      body: formData
    });
    const result = await response.text();
    alert('Result: ' + result);
  } catch (error) {
    alert('Error: ' + error.message);
  }
});
</script>
</body>
</html>
```

2. **Replace `YOUR_APPS_SCRIPT_URL_HERE` with your actual URL**
3. **Open this file in your browser and test**

If this simple test works but your main site doesn't, the issue is in your main site's JavaScript code.

## 🔒 Security Notes

- The script validates email formats
- Prevents duplicate email submissions
- All data is stored securely in Google Sheets
- You can restrict access to the spreadsheet as needed

That's it! You now have a fully functional, serverless form submission system that saves directly to Google Sheets. No backend server required! 🎉