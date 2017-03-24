```
Sub GQExportQuoteToWordDoc()
'Copies Quote Lite screen to a new word document

    Dim LastRow As Long
    Dim wrdApp As Word.Application
    
    If [DateLoaded] < GetLastUpdated([HDOpportunity], [HDQuoteNumber]) Then
        [GQError] = "Error - database has been updated since you loaded the screen, reload the screen and make updates again"
    Else
        Set wrdApp = CreateObject("Word.Application")
        wrdApp.Visible = True
    
        Call GQCopyAuditTable
    
        LastRow = Sheet10.Cells.Find("*", , xlValues, , xlRows, xlPrevious).Row
        
        Range("C9:E" & LastRow).Copy
    
        wrdApp.Documents.Add
        wrdApp.Selection.Paste
        wrdApp.ActiveDocument.Content.ParagraphFormat.SpaceAfter = 0
        wrdApp.ActiveDocument.SaveAs Filename:="D:\Users\" & [HDUsername] & "\Desktop\" & [HDQuoteReference] & " - " & [GQOpportunityName] & ".doc"
        Application.CutCopyMode = False
        
        'delete audit table
        Rows("44:" & LastRow).Select
        Selection.Delete Shift:=xlUp
    End If
    
End Sub
```
