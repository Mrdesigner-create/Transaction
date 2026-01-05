# Transaction

const SHEET_NAME = "Sheet1";

function doPost(e) {
  const lock = LockService.getScriptLock();
  try {
    lock.waitLock(10000);
    const doc = SpreadsheetApp.getActiveSpreadsheet();
    const sheet = doc.getSheetByName(SHEET_NAME) || doc.getSheets()[0];
    const data = JSON.parse(e.postData.contents);
    const timestamp = new Date();

    if (data.action === "UPLOAD_FILE") {
      data.rows.forEach(row => {
        sheet.appendRow([data.uploadDate, timestamp.toLocaleTimeString(), data.fileName, row.join(" | "), "", "ACTIVE", data.fileId]);
      });
    } else if (data.action === "UPDATE_COMMENT") {
      const values = sheet.getDataRange().getValues();
      for (let i = 1; i < values.length; i++) {
        if (values[i][3] === data.rowData) {
          let existing = values[i][4];
          let newHistory = existing ? existing + " | " + data.comment : data.comment;
          sheet.getRange(i + 1, 5).setValue(newHistory);
          break;
        }
      }
    } else if (data.action === "CHANGE_STATUS") {
      const values = sheet.getDataRange().getValues();
      for (let i = 1; i < values.length; i++) {
        if (values[i][6] === data.fileId) sheet.getRange(i + 1, 6).setValue(data.status);
      }
    } else if (data.action === "PERMANENT_DELETE") {
      const values = sheet.getDataRange().getValues();
      for (let i = values.length - 1; i >= 1; i--) {
        if (values[i][6] === data.fileId) sheet.deleteRow(i + 1);
      }
    }
    return ContentService.createTextOutput("Success").setMimeType(ContentService.MimeType.TEXT);
  } finally { lock.releaseLock(); }
}

function doGet() {
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName(SHEET_NAME) || SpreadsheetApp.getActiveSpreadsheet().getSheets()[0];
  const data = sheet.getDataRange().getValues();
  return ContentService.createTextOutput(JSON.stringify(data)).setMimeType(ContentService.MimeType.JSON);
}
