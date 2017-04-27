function import_calendar( start_date, end_date, spreadsheet, sheet_name) {
  var sheet = spreadsheet.getSheetByName(sheet_name);
  
  var cal = CalendarApp.openByName("Sanderson's Events");
  
  sheet.clear();
  events = cal.getEvents( start_date, end_date ); 
  
  irow = 2;
  for( i in events ) {
    evt = events[i];
    var date = evt.getStartTime();
    var formatted_date = (date.getMonth()+1) + '/' + date.getDate() + '/' + date.getYear();
    sheet.getRange("A"+irow).setValue( formatted_date );
    sheet.getRange("B"+irow).setValue( evt.getTitle() );
    irow ++;
  }
     
}

function import_calendars() {
  var start_date = new Date("May 13, 2016");
  var end_date   = new Date("Jan 1, 2018");

  var ss = SpreadsheetApp.getActiveSpreadsheet();
  import_calendar( start_date, end_date, ss, "sandersons" );

```}
