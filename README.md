# Google Forms to PDF Email Automation

This Google Apps Script automates the process of sending Google Forms responses to your email in PDF format immediately after a form submission.

## Features

- Converts Google Forms responses into a formatted PDF.
- Sends an email with plain text content and the PDF as an attachment.
- Automatically triggers on each form submission.

## Prerequisites

1. A Google Form linked to a response sheet.
2. Access to Google Apps Script (from the Google Form or Drive).

## Installation

1. Open your Google Form.
2. Navigate to `Extensions > Apps Script`.
3. Replace the default script with the [code provided here](#script).
4. Save the project with a meaningful name.

## Script

Copy and paste the following script into the Google Apps Script editor:

```javascript
function sendFormResponseAsPDF(e) {
  try {
    // Set email recipient
    const recipient = "your_email@example.com"; // Replace with your email address
    
    // Get form responses
    const responses = e.values; // Array of form responses
    const formTitle = FormApp.getActiveForm().getTitle();
    const headers = e.namedValues;
    
    // Construct plain text content for the email
    let emailContent = `New submission for: ${formTitle}\n\n`;
    for (let [question, answer] of Object.entries(headers)) {
      emailContent += `${question}: ${answer.join(", ")}\n`;
    }

    // Generate PDF content
    const pdfContent = `
      <html>
        <body>
          <h2>${formTitle} - Submission Details</h2>
          <table border="1" cellpadding="5" cellspacing="0">
            ${Object.entries(headers)
              .map(
                ([question, answer]) =>
                  `<tr><td><b>${question}</b></td><td>${answer.join(", ")}</td></tr>`
              )
              .join("")}
          </table>
        </body>
      </html>
    `;

    const blob = Utilities.newBlob(pdfContent, 'text/html').getAs('application/pdf');
    blob.setName(`${formTitle}_Response.pdf`);

    // Send email with PDF attachment
    GmailApp.sendEmail(recipient, `New Form Submission - ${formTitle}`, emailContent, {
      attachments: [blob],
    });
  } catch (error) {
    console.error("Error in sendFormResponseAsPDF:", error);
  }
}

function setupTrigger() {
  // Add the form submit trigger
  const form = FormApp.getActiveForm();
  ScriptApp.newTrigger("sendFormResponseAsPDF")
    .forForm(form)
    .onFormSubmit()
    .create();
}
