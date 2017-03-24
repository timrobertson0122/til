```
'Module for batch uploading Opportunities from the postlist in the Utility Module
'
'
' Written by Tim Robertson - 06/02/17
'
' Amendments:
'         None
'
' Modules:
'   PLUploadOpportunities()      - upload records to the DB, insert or update as appropriate
'   PLOppCopyColumns()           - copies required columns over from postlist
'   PLOppConvertData()           - Sets Opportunity Status field, converts usernames/initials and broker name to ID's for DB
'   PLOppGetOpportunityStatus    - figure out status of Opportunity from postlist and enter it in Opportunity View table
'   PLOppCleanseData()           - remove unwanted characters from postlist data
'   PLOppClearFormatting()       - removes cell formatting from postlist
'   HasInformationToQuote        - check postlist for whether initial review is completed
'   OtherActiveOppInfo()         - Determine if other active Opportunities, get Opp info
'   SetChaserID                  - set AllocatedChaserID based on Opportunity status
'   PLOppcleartable()            - Removes data rows from table

Option Explicit


Sub PLUploadOpportunities()
' upload records to the DB, insert or update as appropriate

    Dim Rw As Integer
    Dim Opportunity As EBGeneralAddIn.COpportunity
    Dim InsertCount As Integer
    Dim InsertBool As Boolean
    
    InsertCount = 0
        
    For Rw = 1 To Range("PLOpportunityView_Table").Rows.Count
    
    Set Opportunity = InstantiateOpportunity(Range("PLOpportunityView_Table[Opportunity Number]")(Rw))
    
        If Range("PLOpportunityView_Table[Opportunity Number]")(Rw) <> "" Then
            Call Opportunity.PLGetOpportunity(Rw)
            Call Opportunity.PLUploadOpportunity(InsertBool)
            If InsertBool Then InsertCount = InsertCount + 1
        End If

    Next
    
    MsgBox "I've inserted " & InsertCount & " records "
    
End Sub


Sub PLOppCopyColumns()
' copies required columns over from postlist

Dim i As Integer
Dim LastRow As Long
Dim Columns
Dim Column As Variant

'clear table data
Call PLOppClearTable

Call PerformanceStart

'these are the columns from the 2017 Open Quotes tab (postlist), copied across in the exact correct order. ZZ:ZZ is used where empty columns are required
Columns = Array("C:C", "D:D", "R:R", "A:A", "AE:AE", "L:L", "M:M", "B:B", "V:V", "W:W", "Y:Y", "N:N", "N:N", "ZZ:ZZ", _
                "O:O", "P:P", "T:T", "Q:Q", "U:U", "D:D", "E:E", "ZZ:ZZ", "F:F", "G:G", "F:F", "K:K", "J:J", "H:H", _
                "ZZ:ZZ", "ZZ:ZZ", "ZZ:ZZ", "ZZ:ZZ", "ZZ:ZZ", "S:S", "X:X", "BJ:BJ", "ZZ:ZZ", "AT:AT", "AC:AC", _
                "ZZ:ZZ", "Z:Z", "ZZ:ZZ", "AD:AD", "ZZ:ZZ", "ZZ:ZZ", "ZZ:ZZ", "ZZ:ZZ", "ZZ:ZZ", "AA:AA")
LastRow = Sheet9.Cells.Find("*", , xlValues, , xlRows, xlPrevious).Row

'this does the copying
i = 0
Do While i <= UBound(Columns)
    For Each Column In Columns
        Intersect(Sheet9.Rows("2:" & LastRow), Sheet9.Range(Column)).Copy Sheet10.Range("B13").Offset(0, i)
        i = i + 1
    Next Column
Loop
   
Call PLOppClearFormatting
Call PLOppConvertData
Call PLOppCleanseData

Call PerformanceEnd

End Sub


Sub PLOppConvertData()
'Sets Opportunity Status field, converts usernames/initials and broker name to ID's for DB etc

    Dim Rw As Integer
  
     For Rw = 1 To Range("PLOpportunityView_Table").Rows.Count
        Range("PLOpportunityView_Table[U/W]")(Rw) = GetUserID(Range("PLOpportunityView_Table[U/W]")(Rw))
        Range("PLOpportunityView_Table[Checked By]")(Rw) = GetUserID(Range("PLOpportunityView_Table[Checked By]")(Rw))
        Range("PLOpportunityView_Table[Logged By]")(Rw) = GetUserID(Range("PLOpportunityView_Table[Logged By]")(Rw))
        Range("PLOpportunityView_Table[Opportunity Status]")(Rw) = PLOppGetOpportunityStatus(Rw)
        Range("PLOpportunityView_Table[Broker - format to match FCA list]")(Rw) = GetBrokerID(Range("PLOpportunityView_Table[Broker - format to match FCA list]")(Rw))
        'calculates number of quotes for that opportunity
        Range("PLOpportunityView_Table[Quote Count]")(Rw) = "=IF(COUNTIF(C$13:C$600,C" & 12 + Rw & ")=COUNTIF(C:C,C" & 12 + Rw & "),COUNTIF(C:C,C" & 12 + Rw & "),"""")"
        Call OtherActiveOppInfo(Rw)
        Range("PLOpportunityView_Table[Has Information To Quote]")(Rw) = HasInformationToQuote(Rw)
        Range("PLOpportunityView_Table[Allocated Chaser ID]")(Rw) = SetChaserID(Rw)

    Next
             
End Sub


Function PLOppGetOpportunityStatus(Row As Integer) As String
' figure out status of opportunity from postlist and enter it in Opportunity View table

    ' Cancelled - doesnt get set
    If Range("Postlist_Table[Are we proceeding to quote]")(Row) = "No" Then
        PLOppGetOpportunityStatus = "Declined"
    ElseIf Range("Postlist_Table[Won / Open / Closed / Retained]")(Row) = "closed" Then
        PLOppGetOpportunityStatus = "Closed"
    ElseIf Range("Postlist_Table[Won / Open / Closed / Retained]")(Row) = "won" Then
    
    ElseIf Range("Postlist_Table[Won / Open / Closed / Retained]")(Row) = "retained" Then
    
    ElseIf Range("Postlist_Table[Position]")(Row) = "issued" Then
        PLOppGetOpportunityStatus = "Issued"
    ElseIf Range("Postlist_Table[Checked / Peer Reviewed by]")(Row) <> "" Then
        PLOppGetOpportunityStatus = "Awaiting Quote Pack Sending"
    ElseIf Range("Postlist_Table[Initial review completed]")(Row) <> "" Then
        PLOppGetOpportunityStatus = "Awaiting Quote(s) Production"
    ElseIf Range("Postlist_Table[Checked by]")(Row) <> "" Then
        PLOppGetOpportunityStatus = "Awaiting Initial Checks Review"
    ElseIf Range("Postlist_Table[Logged By]")(Row) = "" Then
        PLOppGetOpportunityStatus = "Awaiting Initial Checks"
    Else
        PLOppGetOpportunityStatus = "Unknown"
    End If

End Function


Sub PLOppCleanseData()
' remove unwanted characters from postlist data

Dim listObj As ListObject
Set listObj = Sheets("PLOpportunity").ListObjects("PLOpportunityView_Table")
Dim Cell As Range

For Each Cell In listObj.DataBodyRange
    If Cell.Value = "\" Then
        Cell.Value = ""
    End If
Next

For Each Cell In listObj.ListColumns("Opportunity Type").Range
    If Cell.Value = "Rebroke" Then
        Cell.Value = "ReBroke"
    End If
    If Cell.Value = "New business" Then
        Cell.Value = "New Business"
    End If
Next

For Each Cell In listObj.ListColumns("Deadline Type").Range
    If Cell.Value = "Generali" Then
        Cell.Value = "Standard"
    End If
Next

For Each Cell In listObj.ListColumns("Product Type").Range
    If Cell.Value = "Critical Illness" Then
        Cell.Value = "CI"
    End If
Next

For Each Cell In listObj.ListColumns("Current Insurer if known").Range
    If Cell.Value = "New to market" Then
        Cell.Value = ""
    End If
Next

For Each Cell In listObj.ListColumns("Broker - format to match FCA list").Range
    If Cell.Value = "0" Then
        Cell.Value = "FIX ME"
        Cell.Interior.Color = RGB(255, 0, 0)
    End If
Next

For Each Cell In listObj.ListColumns("Head Office Country").Range
    If Cell.Value = "UK" Then
        Cell.Value = "United Kingdom"
    End If
Next

'Dates
For Each Cell In listObj.ListColumns("GEB Spec Sent Date").Range.Offset(1, 0)
    If Not IsDate(Cell.Value) Then
        Cell.Value = ""
    End If
Next

For Each Cell In listObj.ListColumns("Opportunity Date Received").Range
    Cell.NumberFormat = "m/d/yyyy"
Next

For Each Cell In listObj.ListColumns("Deadline").Range
    Cell.NumberFormat = "m/d/yyyy"
Next

For Each Cell In listObj.ListColumns("EBCS Data Date").Range
    Cell.NumberFormat = "m/d/yyyy"
Next

For Each Cell In listObj.ListColumns("GEB Spec Sent Date").Range
    Cell.NumberFormat = "m/d/yyyy"
Next

For Each Cell In listObj.ListColumns("Effective Date").Range
    Cell.NumberFormat = "m/d/yyyy"
Next

For Each Cell In listObj.ListColumns("Initial review completed").Range
    Cell.NumberFormat = "m/d/yyyy"
Next

End Sub


Sub PLOppClearFormatting()
' removes cell formatting from postlist

Call ClearTableFormatting("PLOpportunityView_Table")
    
End Sub


Function HasInformationToQuote(Row As Integer) As String
' check postlist for whether initial review is completed
    
    If Range("Postlist_Table[Initial review completed]")(Row) <> "" Then
        HasInformationToQuote = "Yes"
    Else
        HasInformationToQuote = "No"
    End If

End Function


Sub OtherActiveOppInfo(Row As Integer)
' Determine if other active Opportunities, get Opp info

Dim Rw As Integer

For Rw = 1 To Range("PLOpportunityView_Table").Rows.Count
    If Range("PLOpportunityView_Table[Opportunity Name]")(Row) = Range("PLOpportunityView_Table[Opportunity Name]")(Rw) And _
       Range("PLOpportunityView_Table[Opportunity Number]")(Row) <> Range("PLOpportunityView_Table[Opportunity Number]")(Rw) And _
       Row <> Rw And _
       Range("PLOpportunityView_Table[Opportunity Name]")(Row) <> "" Then
        If InStr(1, Range("PLOpportunityView_Table[Other Active Quotes Info]")(Row), CStr(Range("PLOpportunityView_Table[Opportunity Number]")(Rw))) = 0 Then
            Range("PLOpportunityView_Table[Other Active Quotes Info]")(Row) = Range("PLOpportunityView_Table[Other Active Quotes Info]")(Row) & IIf(Range("PLOpportunityView_Table[Other Active Quotes Info]")(Row) <> "", " / ", "") & Range("PLOpportunityView_Table[Opportunity Number]")(Rw) & " - " & Range("PLOpportunityView_Table[Product Type]")(Rw)
        End If
    End If
Next

If Range("PLOpportunityView_Table[Opportunity Name]")(Row) <> "" Then
    Range("PLOpportunityView_Table[Has Other Active Quotes]")(Row) = IIf(Range("PLOpportunityView_Table[Other Active Quotes Info]")(Row) <> "", "Yes", "No")
End If
      
End Sub


Function SetChaserID(Row As Integer) As Integer
' set AllocatedChaserID based on Opportunity status

    If Range("PLOpportunityView_Table[Opportunity Status]")(Row) = "Issued" Then
        SetChaserID = Range("PLOpportunityView_Table[U/W]")(Row)
    Else
        SetChaserID = 0
    End If

End Function


Sub PLOppClearTable()
' Removes data rows from table

Call DeleteTableRows("PLOpportunityView_Table")

End Sub
```
