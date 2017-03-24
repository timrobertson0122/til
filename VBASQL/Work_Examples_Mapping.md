```
'
' M01_Import
'
' This module contains all the code for updating the DATA and PLAN tabs from CSV data (JIRA)
'
' Written by Tim Robertson - 10/03/2017
'
' Amendments:
'
' Modules:
'   AddProjectToDataTab() - Manages importing (insert/update) of CSV data to DATA tab row by row
'   AddProjectToPlanTab() - Manages importing (insert/update) of CSV data to PLAN tab row by row
'   AddNewDataRow()       - gets data, inserts a new row to a table and then inserts data on that row
'   AddNewPlanEntry()     - inserts new entry on PLAN tab
'   UpdateDataRow()       - updates entry on DATA row
'   UpdatePlanEntry()     - update entry on PLAN tab
'   GetBusinessUnit       - gets DATA Sponsor from CSV Business Unit
'   GetBusinessUnit2      - gets PLAN Business Unit from DATA Sponsor
'   Function GetName      - Converts Owner name from CSV format to DATA tab format
'   Function FirstColumn  - PLAN tab gets calendar column letter from estimated start date entry
'   Function LastColumn   - PLAN tab gets calendar column letter from estimated end date entry
'   UpdateGanttChart()    - updates Gantt Chart from dates in T_PLAN table
'   ClearFormatting()     - remove colour formatting from row
'   BUCase()              - case statement for formatting entries on Gantt chart

Option Explicit

Sub AddProjectToDataTab()
' Manages importing (insert/update) of CSV data to DATA tab row by row

    Dim Row As Integer
    Dim RgFound As Range
    Dim ProjectID As Integer
    Dim ProjectName As String
    Dim Owner As String
    Dim SponsorName As String
    Dim StartDate As String
    Dim EndDate As String
    Dim Business_Unit As String

    'start at row 2 of CSV data (ignore header row)
    Row = 2
    
    'loop through CSV data
    Do While Sheet9.Cells(Row, 1) <> ""
        
        ' insert/update DATA tab - look for existing record by JIRA Reference
        Set RgFound = Range("T_PROJ[JIRA Reference]").Find(Sheet9.Cells(Row, 1).Value)
        
        If RgFound Is Nothing Then
        'not found, add new row
            Call AddNewDataRow(Row, ProjectID, ProjectName, Owner, SponsorName, StartDate, EndDate, Business_Unit)
        Else
        'existing entry, update with data from CSV
            Call UpdateDataRow(Row, RgFound)
        End If
        
        Row = Row + 1
        
    Loop

End Sub


Sub AddProjectToPlanTab()
' Manages importing (insert/update) of CSV data to PLAN tab row by row

    Dim Row As Integer
    Dim RgFound As Range
    Dim ProjectID As Integer
    Dim ProjectName As String
    Dim Owner As String
    Dim StartDate As String
    Dim EndDate As String
    Dim Business_Unit As String
    
    For Row = 1 To Sheet1.Range("T_PROJ").Rows.Count
        If Sheet1.Range("T_PROJ[Add to PLAN?]")(Row) = "Yes" Then
             'if it also exists in PLAN tab then update it there too
            Set RgFound = Sheet6.Range("T_PLAN[ID]").Find(Sheet1.Range("T_PROJ[PROJECT ID]")(Row))
                If RgFound Is Nothing Then
                    'not found in PLAN tab, add
                    Call AddNewPlanEntry(Row)
                Else
                    'existing entry, update with data from DATA tab
                    Call UpdatePlanEntry(Row, RgFound)
                End If
        Else
            'Set to 'No', do nothing
        End If
        
    Next

End Sub


Sub AddNewDataRow(Rw As Integer, Project_ID As Integer, Name As String, Owner As String, _
              SponsorName As String, Start_Date As String, End_Date As String, Business_Unit As String)
' gets data, inserts a new row to a table and then inserts data on that row

    Dim JIRA_Ref As String
    Dim STAGE_1DT As String
    Dim STAGE_2DT As String
    Dim STAGE_3DT As String
    Dim STAGE_4DT As String
    Dim STAGE_5DT As String
    Dim STAGE_6DT As String
    Dim STAGE_7DT As String
    Dim Current_Stage As String
    Dim Completion_Date As String
    Dim Duration As String
    Dim Validation As String
    
    Project_ID = Sheets("Data").Cells(Rows.Count, "A").End(xlUp).Value + 1
    JIRA_Ref = Sheet9.Cells(Rw, 1).Value
    Name = Sheet9.Cells(Rw, 4).Value
    Start_Date = Format(Sheet9.Cells(Rw, 5).Value, "dd-MMM-yyyy")
    End_Date = Format(Sheet9.Cells(Rw, 6).Value, "dd-MMM-yyyy")
    STAGE_1DT = Format(Sheet9.Cells(Rw, 15).Value, "dd-MMM-yyyy")
    STAGE_2DT = Format(Sheet9.Cells(Rw, 16).Value, "dd-MMM-yyyy")
    STAGE_3DT = Format(Sheet9.Cells(Rw, 17).Value, "dd-MMM-yyyy")
    STAGE_4DT = Format(Sheet9.Cells(Rw, 18).Value, "dd-MMM-yyyy")
    STAGE_5DT = Format(Sheet9.Cells(Rw, 19).Value, "dd-MMM-yyyy")
    STAGE_6DT = Format(Sheet9.Cells(Rw, 20).Value, "dd-MMM-yyyy")
    STAGE_7DT = Format(Sheet9.Cells(Rw, 21).Value, "dd-MMM-yyyy")
    Business_Unit = GetBusinessUnit(Sheet9.Cells(Rw, 24).Value)
    Owner = GetName(Sheet9.Cells(Rw, 13).Value)
    SponsorName = GetSponsorName(Sheet9.Cells(Rw, 12).Value)
    Current_Stage = "=IFERROR(IF(INDEX(T_PROJ,ROW([@[PROJECT ID]])-ROW(T_PROJ[[#Headers],[PROJECT ID]]),MATCH(""STAGE ""&N_ST&"" DT"",T_PROJ[#Headers],0))>0,""COMPLETED"",INDEX(L_ST,IF([@[STAGE 6 DT]]>0,7,IF([@[STAGE 5 DT]]>0,6,IF([@[STAGE 4 DT]]>0,5,IF([@[STAGE 3 DT]]>0,4,IF([@[STAGE 2 DT]]>0,3,IF([@[STAGE 1 DT]]>0,2,IF([@[START DATE]]>0,1,0))))))))),"""")"
    Completion_Date = "=IF([@[CURRENT STAGE]]=""COMPLETED"",INDEX(T_PROJ,ROW([@[PROJECT ID]])-ROW(T_PROJ[[#Headers],[PROJECT ID]]),MATCH(""STAGE ""&N_ST&"" DT"",T_PROJ[#Headers],0)),"""")"
    Duration = "=IFERROR(IF([@VALIDATION]=""NO ERROR"",IF([@[CURRENT STAGE]]=""COMPLETED"",[@[COMPLETION DATE]]-[@[START DATE]]+1,TD-[@[START DATE]]+1),""""),"""")"
    Validation = "=IFERROR(IF(AND(AND(IF([@[STAGE 1 DT]]>0,[@[START DATE]]<=[@[STAGE 1 DT]],TRUE),IF([@[STAGE 2 DT]]>0,[@[STAGE 1 DT]]<=[@[STAGE 2 DT]],TRUE),IF([@[STAGE 3 DT]]>0,[@[STAGE 2 DT]]<=[@[STAGE 3 DT]],TRUE),IF([@[STAGE 4 DT]]>0,[@[STAGE 3 DT]]<=[@[STAGE 4 DT]],TRUE),IF([@[STAGE 5 DT]]>0,[@[STAGE 4 DT]]<=[@[STAGE 5 DT]],TRUE),IF([@[STAGE 6 DT]]>0,[@[STAGE 5 DT]]<=[@[STAGE 6 DT]],TRUE)),IF([@[CURRENT STAGE]]=""COMPLETED"",COUNTIF(OFFSET([@[START DATE]],0,0,1,N_ST),"""")=0,COUNTIF(OFFSET([@[START DATE]],0,0,1,IFERROR(MATCH([@[CURRENT STAGE]],L_ST,0),0)),"""")=0)),""NO ERROR"",""ERROR""),""ERROR"")"

    AddTableRow "T_PROJ", Array(Project_ID, JIRA_Ref, Name, Name, "No", End_Date, Start_Date, STAGE_1DT, STAGE_2DT, _
                                STAGE_3DT, STAGE_4DT, STAGE_5DT, STAGE_6DT, STAGE_7DT, Business_Unit, _
                                Owner, SponsorName, Current_Stage, Completion_Date, Duration, Validation)
                                
End Sub


Sub AddNewPlanEntry(Rw As Integer)
' inserts new entry on PLAN tab

    Dim RgFound As Range
    Dim CalRange As Range
    Dim NextFreeRow As Integer
    Dim FirstRowForSection
    Dim Project_ID As Integer
    Dim Name As String
    Dim Owner As String
    Dim Start_Date As String
    Dim End_Date As String
    Dim Business_Unit As String
    
    Project_ID = Sheet1.Range("T_PROJ[PROJECT ID]")(Rw)
    Name = Sheet1.Range("T_PROJ[NAME]")(Rw)
    Owner = Sheet1.Range("T_PROJ[OWNER]")(Rw)
    Start_Date = Format(Sheet1.Range("T_PROJ[START DATE]")(Rw), "dd-MMM-yyyy")
    End_Date = Format(Sheet1.Range("T_PROJ[End Date]")(Rw), "dd-MMM-yyyy")
    
    ' lookup business unit
    Business_Unit = GetBusinessUnit2(Sheet1.Range("T_PROJ[SPONSOR]")(Rw))
    
    ' search for relevant business unit section on PLAN tab
    Set RgFound = Range("T_PLAN[Project Name]").Find(Business_Unit)
    
    ' move to first ID row for relevant business unit section
    FirstRowForSection = RgFound.Offset(1, -1).Address
    
    ' find and set where to insert new row
    NextFreeRow = Sheets("PLAN").Range(FirstRowForSection & ":A" & Rows.Count).Cells.SpecialCells(xlCellTypeBlanks).Row
    
    'insert new row with row id
    AddTableRow "T_PLAN", Array(Project_ID, Name, Owner, Start_Date, End_Date, Business_Unit), NextFreeRow
    
    'logic for colouring calendar for row
    If Start_Date = "" And End_Date = "" Or Start_Date > End_Date Then
        'do nothing
    Else
        If LastColumn(End_Date) = "" Then
            'do nothing
        Else
            If FirstColumn(Start_Date) = "" And LastColumn(End_Date) <> "" Then
                'start date preceeds calendar so set to start of calendar
                Set CalRange = Sheets("PLAN").Range("L" & NextFreeRow & ":" & LastColumn(End_Date) & NextFreeRow)
            Else
                Set CalRange = Sheets("PLAN").Range(FirstColumn(Start_Date) & NextFreeRow & ":" & LastColumn(End_Date) & NextFreeRow)
            End If
          
          Call BUCase(Business_Unit, CalRange)
      
        End If
    End If
    
End Sub


Sub UpdateDataRow(Rw As Integer, JIRARef As Range)
' updates entry on DATA row

    Dim DataRowNumber As Integer

    DataRowNumber = JIRARef.Row
    
    Sheet1.Cells(DataRowNumber, 7).Value = Format(Sheet9.Cells(Rw, 5).Value, "dd-MMM-yyyy")
    Sheet1.Cells(DataRowNumber, 8).Value = Format(Sheet9.Cells(Rw, 15).Value, "dd-MMM-yyyy")
    Sheet1.Cells(DataRowNumber, 9).Value = Format(Sheet9.Cells(Rw, 16).Value, "dd-MMM-yyyy")
    Sheet1.Cells(DataRowNumber, 10).Value = Format(Sheet9.Cells(Rw, 17).Value, "dd-MMM-yyyy")
    Sheet1.Cells(DataRowNumber, 11).Value = Format(Sheet9.Cells(Rw, 18).Value, "dd-MMM-yyyy")
    Sheet1.Cells(DataRowNumber, 12).Value = Format(Sheet9.Cells(Rw, 19).Value, "dd-MMM-yyyy")
    Sheet1.Cells(DataRowNumber, 13).Value = Format(Sheet9.Cells(Rw, 20).Value, "dd-MMM-yyyy")
    Sheet1.Cells(DataRowNumber, 14).Value = Format(Sheet9.Cells(Rw, 21).Value, "dd-MMM-yyyy")
    Sheet1.Cells(DataRowNumber, 15).Value = GetBusinessUnit(Sheet9.Cells(Rw, 24).Value)
    Sheet1.Cells(DataRowNumber, 16).Value = GetName(Sheet9.Cells(Rw, 13).Value)
    Sheet1.Cells(DataRowNumber, 17).Value = GetSponsorName(Sheet9.Cells(Rw, 12).Value)

End Sub


Sub UpdatePlanEntry(Rw As Integer, Project_ID As Range)
' update entry on PLAN tab

    Dim DataRowNumber As Integer
    Dim Start_Date As String
    Dim End_Date As String
    Dim Business_Unit As String
    Dim CalRange As Range
    
    DataRowNumber = Project_ID.Row
    
    Sheet6.Cells(DataRowNumber, 3).Value = Sheet1.Range("T_PROJ[OWNER]")(Rw)
    Sheet6.Cells(DataRowNumber, 4).Value = Sheet1.Range("T_PROJ[START DATE]")(Rw)
    Sheet6.Cells(DataRowNumber, 5).Value = Sheet1.Range("T_PROJ[End Date]")(Rw)
    
    Start_Date = Sheet6.Cells(DataRowNumber, 4).Value
    End_Date = Sheet6.Cells(DataRowNumber, 5).Value
    Business_Unit = GetBusinessUnit2(Sheet1.Range("T_PROJ[SPONSOR]")(Rw))
     
    Sheet6.Cells(DataRowNumber, 6).Value = Business_Unit
    
    'clear colour formatting
    Call ClearFormatting(DataRowNumber)
            
    'logic for colouring calendar for row
    If Start_Date = "" And End_Date = "" Or Start_Date > End_Date Then
        'do nothing
    Else
        If LastColumn(End_Date) = "" Then
            'do nothing
        Else
            If FirstColumn(Start_Date) = "" And LastColumn(End_Date) <> "" Then
                'start date preceeds calendar so set to start of calendar
                Set CalRange = Sheets("PLAN").Range("L" & DataRowNumber & ":" & LastColumn(End_Date) & DataRowNumber)
            Else
                Set CalRange = Sheets("PLAN").Range(FirstColumn(Start_Date) & DataRowNumber & ":" & LastColumn(End_Date) & DataRowNumber)
            End If

            Call BUCase(Business_Unit, CalRange)

        End If
    End If

End Sub


Function GetBusinessUnit(BusinessDivision As String) As String
' gets DATA Sponsor from CSV Business Unit

    GetBusinessUnit = Application.WorksheetFunction.VLookup(BusinessDivision, [Sponsor_Table].Resize(, 2), 2, 0)

End Function


Function GetBusinessUnit2(BusinessDivision As String) As String
' gets PLAN Business Unit from DATA Sponsor

    GetBusinessUnit2 = Application.WorksheetFunction.VLookup(BusinessDivision, [BU_Table].Resize(, 2), 2, 0)

End Function


Function GetName(BusinessOwner As String) As String
' Converts Owner name from CSV format to DATA tab format

    GetName = Application.WorksheetFunction.VLookup(BusinessOwner, [Owner_Table].Resize(, 2), 2, 0)

End Function


Function GetSponsorName(Sponsor As String) As String
' Converts Sponsor name from CSV format to DATA tab format

    GetSponsorName = Application.WorksheetFunction.VLookup(Sponsor, [SponsorName_Table].Resize(, 2), 2, 0)
    
End Function

Sub AddTableRow(ByVal strTableName As String, ByRef arrData As Variant, Optional ByRef RowNumber As Integer)
' This function adds a row to a table. If the table has only one row, data will be added to that row.
    
    Dim LastRowID As Long
    Dim tbl As ListObject
    Dim NewRow As ListRow
    
    Set tbl = Range(strTableName).ListObject
        
    If Range(strTableName).Rows.Count = 1 And Range(strTableName)(1, 1) = "" Then
        If TypeName(arrData) = "Range" Then
            tbl.ListRows(1).Range = arrData.Value
        Else
            tbl.ListRows(1).Range = arrData
        End If
    Else
        ' RowNumber only relevant on PLAN tab as not sequential
        If RowNumber = 0 Then
            Set NewRow = tbl.ListRows.Add(AlwaysInsert:=True)
        Else
        ' Need to offset for table
            Set NewRow = tbl.ListRows.Add(RowNumber - 3, AlwaysInsert:=True)
            
        ' insert new calendar row and remove formatting
        With Sheets("PLAN").Range("L" & RowNumber & ":BK" & RowNumber)
            .Insert Shift:=xlDown, CopyOrigin:=xlFormatFromLeftOrAbove
        End With
        Call ClearFormatting(RowNumber)
              
        End If
        ' Handle Arrays and Ranges
        If TypeName(arrData) = "Range" Then
            NewRow.Range = arrData.Value
        Else
            NewRow.Range = arrData
        End If
        
        Set tbl = Nothing
        
    End If

End Sub


Function FirstColumn(SDate As String) As String
' PLAN tab gets calendar column letter from estimated start date entry

    Dim LastFriDate As String
    Dim ColumnNumber As Integer
    Dim MyDate As Date
    Dim WeekdayNumber As Integer
    Dim LookupResult As Object
    
    'convert to date
    MyDate = CDate(SDate)
    
    'get weekday number
    WeekdayNumber = Weekday(MyDate)
    
    'if older than calendar set to first column of calendar(11)
    If MyDate < "06/01/2017" Then
        FirstColumn = Split(Cells(1, 12).Address, "$")(1)
    Else
        If WeekdayNumber = 6 Then
            'do nothing
        ElseIf WeekdayNumber = 7 Then
            MyDate = MyDate - 1
        Else
            MyDate = MyDate - 1 - WeekdayNumber
        End If
    
    'format as string to match calendar
    LastFriDate = Format(MyDate, "dd/mm")
    
    'search for date in calendar (PLAN row 2)
    Set LookupResult = Sheets("PLAN").Cells(4, 1).EntireRow.Find(What:=LastFriDate _
                                                       , LookIn:=xlValues _
                                                       , LookAt:=xlPart _
                                                       , SearchOrder:=xlByColumns _
                                                       , SearchDirection:=xlPrevious _
                                                       , MatchCase:=False)
    
        If LookupResult Is Nothing Then
        'not found
            ColumnNumber = 0
            FirstColumn = ""
        Else
        'get column letter from number
            ColumnNumber = LookupResult.Column
            FirstColumn = Split(Cells(1, ColumnNumber).Address, "$")(1)
        End If
               
    End If
    
End Function


Function LastColumn(SDate As String) As String
' PLAN tab gets calendar column letter from estimated end date entry

    Dim NextFriDate As String
    Dim ColumnNumber As Integer
    Dim MyDate As Date
    Dim WeekdayNumber As Integer
    Dim LookupResult As Object
    
    'convert to date
    MyDate = CDate(SDate)
    
    'get weekday number
    WeekdayNumber = Weekday(MyDate)
    
     'if later than end of calendar set to last column of calendar(63)
    If MyDate > "29/12/2017" Then
        LastColumn = Split(Cells(1, 63).Address, "$")(1)
    Else
        If WeekdayNumber = 6 Then
        'do nothing
        ElseIf WeekdayNumber = 7 Then
            MyDate = MyDate + WeekdayNumber - 1
        Else
            MyDate = MyDate + 6 - WeekdayNumber
        End If
        
    'format as string to match calendar
    NextFriDate = Format(MyDate, "dd/mm")

    'search for date in calendar (PLAN row 2)
    Set LookupResult = Sheets("PLAN").Cells(4, 1).EntireRow.Find(What:=NextFriDate _
                                                       , LookIn:=xlValues _
                                                       , LookAt:=xlPart _
                                                       , SearchOrder:=xlByColumns _
                                                       , SearchDirection:=xlPrevious _
                                                       , MatchCase:=False)
       If LookupResult Is Nothing Then
        'not found
            ColumnNumber = 0
            LastColumn = ""
        Else
        'get column letter from number
            ColumnNumber = LookupResult.Column
            LastColumn = Split(Cells(1, ColumnNumber).Address, "$")(1)
        End If

    End If
    
End Function


Sub UpdateGanttChart()
' updates Gantt Chart from dates in T_PLAN table

    Dim Rw As Integer
    Dim CalRow As Integer
    Dim Start_Date As String
    Dim End_Date As String
    Dim CalRange As Range
    Dim Business_Unit As String
    
    For Rw = 2 To Range("T_PLAN").Rows.Count
     
        Start_Date = Range("T_PLAN[Est. Start]")(Rw)
        
        If IsDate(Range("T_PLAN[Revised End date]")(Rw)) Then
            End_Date = Range("T_PLAN[Revised End date]")(Rw)
        ElseIf IsDate(Range("T_PLAN[Est.End]")(Rw)) Then
            End_Date = Range("T_PLAN[Est.End]")(Rw)
        Else
            End_Date = ""
        End If
        
        Business_Unit = Range("T_PLAN[Business Unit]")(Rw)
         
        CalRow = Rw + 3
             
        'clear colour formatting
        Call ClearFormatting(CalRow)

        'logic for colouring calendar for row
        If Start_Date = "" Or End_Date = "" Then
            'do nothing
        Else
            If LastColumn(End_Date) = "" Then
                'do nothing
            Else
                If FirstColumn(Start_Date) = "" And LastColumn(End_Date) <> "" Then
                    'start date preceeds calendar so set to start of calendar
                    Set CalRange = Sheets("PLAN").Range("L" & CalRow & ":" & LastColumn(End_Date) & CalRow)
                Else
                    Set CalRange = Sheets("PLAN").Range(FirstColumn(Start_Date) & CalRow & ":" & LastColumn(End_Date) & CalRow)
                End If
                Call BUCase(Business_Unit, CalRange)

            End If
        End If
    
    Next

End Sub

Sub ClearFormatting(Rw)
' remove colour formatting from row

    With Sheets("PLAN").Range("K" & Rw & ":BK" & Rw).Interior
        .Pattern = xlLightUp
        .PatternThemeColor = xlThemeColorAccent5
        .ThemeColor = xlThemeColorDark1
        .TintAndShade = 0
        .PatternTintAndShade = 0.399914548173467
    End With
    
End Sub

Sub BUCase(Business_Unit As String, CalRange As Range)
' case statement for formatting entries on Gantt chart

    Select Case Business_Unit
        Case "EMPLOYEE BENEFITS"
            With CalRange.Interior
                .Pattern = xlLightUp
                .PatternThemeColor = xlThemeColorAccent5
                .ThemeColor = xlThemeColorAccent6
                .TintAndShade = 0.399945066682943
                .PatternTintAndShade = 0.399914548173467
            End With
        Case "GLOBAL CORPORATE & COMMERCIAL"
            With CalRange.Interior
                .Pattern = xlLightUp
                .PatternThemeColor = xlThemeColorAccent5
                .ThemeColor = xlThemeColorAccent5
                .TintAndShade = 0.399945066682943
                .PatternTintAndShade = 0.599963377788629
            End With
        Case "GENERALI GLOBAL HEALTH"
            With CalRange.Interior
                .Pattern = xlLightUp
                .PatternThemeColor = xlThemeColorAccent5
                .ThemeColor = xlThemeColorAccent4
                .TintAndShade = 0.399945066682943
                .PatternTintAndShade = 0.599963377788629
            End With
        Case "UK CORPORATE & COMMERCIAL"
            With CalRange.Interior
                .Pattern = xlLightUp
                .PatternThemeColor = xlThemeColorAccent5
                .ThemeColor = xlThemeColorAccent3
                .TintAndShade = 0.399945066682943
                .PatternTintAndShade = 0.599963377788629
            End With
        Case "UK BRANCH"
            With CalRange.Interior
                .Pattern = xlLightUp
                .PatternThemeColor = xlThemeColorAccent5
                .ThemeColor = xlThemeColorAccent2
                .TintAndShade = 0.399945066682943
                .PatternTintAndShade = 0.599963377788629
            End With
    End Select
End Sub
```
