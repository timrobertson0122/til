M2 - Timed

This module manages production of the reports on the Timed sheet of the workbook.


Written by Tim Robertson - 16/03/2017

Amendments:

Modules

FRGetTimedData()    - Download Timed opportunity report data and place into Excel table
TRHowLong()         - Report showing average time between transactions in hours per user
 
Option Explicit

```
Sub TRGetTimedData()
Download Timed opportunity report data and place into Excel table

    DeleteTableRows ("TR_Table")
    
    Dim LinkToServer As New ADODB.Connection
    Dim LoginAccess As String
    Dim SQLSyntax As String
    Dim Basket As New ADODB.Recordset ' recordset is fetched in.
    Dim Data As String
    Dim HowLong As String
    Dim TransactionFilters As String
    Dim StartDate As Date
    Dim SQLStartDate As String
    Dim Transaction As String
    Dim AllocatedUnderwriter As String
    Dim Username As String
    Dim BrokerName As String
    Dim PoolingCode As String
    Dim GroupBrokerName As String
    Dim ParentCountry As String
    
    [TRError] = ""

    SQLStartDate = VBAtoSQLUpdateFormatDate([TRStartDate])
    
     Data = " COUNT(DISTINCT TA.OpportunityID) LastUpdated, " & _
            " COUNT(DISTINCT(CASE WHEN DATEDIFF(week, " & SQLStartDate & ", TA.LastUpdated) = 0 THEN TA.OpportunityID END)) as Week_1, " & _
            " COUNT(DISTINCT(CASE WHEN DATEDIFF(week, " & SQLStartDate & ", TA.LastUpdated) = 1 THEN TA.OpportunityID END)) as Week_2, " & _
            " COUNT(DISTINCT(CASE WHEN DATEDIFF(week, " & SQLStartDate & ", TA.LastUpdated) = 2 THEN TA.OpportunityID END)) as Week_3, " & _
            " COUNT(DISTINCT(CASE WHEN DATEDIFF(week, " & SQLStartDate & ", TA.LastUpdated) = 3 THEN TA.OpportunityID END)) as Week_4, " & _
            " COUNT(DISTINCT(CASE WHEN DATEDIFF(week, " & SQLStartDate & ", TA.LastUpdated) = 4 THEN TA.OpportunityID END)) as Week_5, " & _
            " COUNT(DISTINCT(CASE WHEN DATEDIFF(week, " & SQLStartDate & ", TA.LastUpdated) = 5 THEN TA.OpportunityID END)) as Week_6, " & _
            " COUNT(DISTINCT(CASE WHEN DATEDIFF(week, " & SQLStartDate & ", TA.LastUpdated) = 6 THEN TA.OpportunityID END)) as Week_7, " & _
            " COUNT(DISTINCT(CASE WHEN DATEDIFF(week, " & SQLStartDate & ", TA.LastUpdated) = 7 THEN TA.OpportunityID END)) as Week_8, " & _
            " COUNT(DISTINCT(CASE WHEN DATEDIFF(week, " & SQLStartDate & ", TA.LastUpdated) = 8 THEN TA.OpportunityID END)) as Week_9, " & _
            " COUNT(DISTINCT(CASE WHEN DATEDIFF(week, " & SQLStartDate & ", TA.LastUpdated) = 9 THEN TA.OpportunityID END)) as Week_10, " & _
            " COUNT(DISTINCT(CASE WHEN DATEDIFF(week, " & SQLStartDate & ", TA.LastUpdated) = 10 THEN TA.OpportunityID END)) as Week_11, " & _
            " COUNT(DISTINCT(CASE WHEN DATEDIFF(week, " & SQLStartDate & ", TA.LastUpdated) = 11 THEN TA.OpportunityID END)) as Week_12 " & _
            "FROM dbo.TransactionAudit AS TA "
                
    TransactionFilters = "WHERE TA.IsCancelled = '' " & _
                         "AND TransactionDesc = '" & [TRTransaction2] & "'" & _
                         " AND TA.QuoteLevel = 0 "
                         
    LoginAccess = [HDConnectionString]
    
    Select Case [TRFilteredBy]
    
        Case "User"
        
            SQLSyntax = "SELECT US.Username, " & _
                        Data & _
                        "LEFT JOIN dbo.Users AS US " & _
                        "ON TA.UserID = US.UserID " & _
                        TransactionFilters & _
                        "GROUP BY US.Username " & _
                        "ORDER BY US.Username"
                
        Case "Allocated UW"

            SQLSyntax = "SELECT US.Username, " & _
                        Data & _
                        "LEFT JOIN dbo.Opportunity AS OP " & _
                        "ON TA.OpportunityID = OP.OpportunityID " & _
                        "LEFT JOIN dbo.Users AS US " & _
                        "ON OP.AllocatedUnderwriterID = US.UserID " & _
                        TransactionFilters & _
                        "GROUP BY US.Username " & _
                        "ORDER BY US.Username"

        Case "Broker"

            SQLSyntax = "SELECT BR.BrokerName, " & _
                        Data & _
                        "LEFT JOIN dbo.Opportunity AS OP " & _
                        "ON TA.OpportunityID = OP.OpportunityID " & _
                        "LEFT JOIN dbo.Broker AS BR " & _
                        "ON OP.BrokerID = BR.BrokerID " & _
                        TransactionFilters & _
                        "GROUP BY BR.BrokerName " & _
                        "ORDER BY CASE WHEN BR.BrokerName = 'nullbroker' THEN 1 ELSE 2 END, BR.BrokerName"

        Case "Broker Group"

            SQLSyntax = "SELECT BR.GroupBrokerName, " & _
                        Data & _
                        "LEFT JOIN dbo.Opportunity AS OP " & _
                        "ON TA.OpportunityID = OP.OpportunityID " & _
                        "LEFT JOIN dbo.Broker AS BR " & _
                        "ON OP.BrokerID = BR.BrokerID " & _
                        TransactionFilters & _
                        "GROUP BY BR.GroupBrokerName " & _
                        "ORDER BY BR.GroupBrokerName"

        Case "Country"

            SQLSyntax = "SELECT OP.ParentCountry, " & _
                        Data & _
                        "LEFT JOIN dbo.Opportunity AS OP " & _
                        "ON TA.OpportunityID = OP.OpportunityID " & _
                        TransactionFilters & _
                        "GROUP BY OP.ParentCountry " & _
                        "ORDER BY OP.ParentCountry"

        Case "Pooling Code"

            SQLSyntax = "SELECT OP.PoolingCode, " & _
                        Data & _
                        "LEFT JOIN dbo.Opportunity AS OP " & _
                        "ON TA.OpportunityID = OP.OpportunityID " & _
                        TransactionFilters & _
                        "GROUP BY OP.PoolingCode " & _
                        "ORDER BY OP.PoolingCode"
             
    End Select
    
    LinkToServer.ConnectionTimeout = 30
    LinkToServer.Open LoginAccess
    
    Basket.Open SQLSyntax, LinkToServer
     
    Select Case [TRFilteredBy]
        Case "User"
            If Basket.EOF Then
                [TRError] = "No records for filter or period"
            Else
                While Not Basket.EOF
                    [TRError] = ""
                        ' change 'nulluser' to 'Unallocated'
                        Username = IIf((Basket.Fields("Username").Value) = "nulluser", "Unassigned", Basket.Fields("Username").Value)
                        AddTableRow "TR_Table", _
                            Array(Username, _
                            Basket.Fields("Week_1").Value, "", _
                            Basket.Fields("Week_2").Value, "", _
                            Basket.Fields("Week_3").Value, "", _
                            Basket.Fields("Week_4").Value, "", _
                            Basket.Fields("Week_5").Value, "", _
                            Basket.Fields("Week_6").Value, "", _
                            Basket.Fields("Week_7").Value, "", _
                            Basket.Fields("Week_8").Value, "", _
                            Basket.Fields("Week_9").Value, "", _
                            Basket.Fields("Week_10").Value, "", _
                            Basket.Fields("Week_11").Value, "", _
                            Basket.Fields("Week_12").Value, "")
                    Basket.MoveNext
                Wend
                TRHowLong
            End If
        Case "Allocated UW"
            If Basket.EOF Then
                [TRError] = "No records for filter or period"
            Else
                While Not Basket.EOF
                    [TRError] = ""
                        ' change 'nulluser' to 'Unallocated'
                        Username = IIf((Basket.Fields("Username").Value) = "nulluser", "Unassigned", Basket.Fields("Username").Value)
                        AddTableRow "TR_Table", _
                            Array(Username, _
                            Basket.Fields("Week_1").Value, "", _
                            Basket.Fields("Week_2").Value, "", _
                            Basket.Fields("Week_3").Value, "", _
                            Basket.Fields("Week_4").Value, "", _
                            Basket.Fields("Week_5").Value, "", _
                            Basket.Fields("Week_6").Value, "", _
                            Basket.Fields("Week_7").Value, "", _
                            Basket.Fields("Week_8").Value, "", _
                            Basket.Fields("Week_9").Value, "", _
                            Basket.Fields("Week_10").Value, "", _
                            Basket.Fields("Week_11").Value, "", _
                            Basket.Fields("Week_12").Value, "")
                    Basket.MoveNext
                Wend
            End If
        Case "Broker"
            If Basket.EOF Then
                [TRError] = "No records for filter or period"
            Else
                While Not Basket.EOF
                    [TRError] = ""
                        ' change 'nullbroker' to 'Unassigned'
                        BrokerName = IIf((Basket.Fields("BrokerName").Value) = "nullbroker", "Unassigned", Basket.Fields("BrokerName").Value)
                        AddTableRow "TR_Table", _
                            Array(BrokerName, _
                            Basket.Fields("Week_1").Value, "", _
                            Basket.Fields("Week_2").Value, "", _
                            Basket.Fields("Week_3").Value, "", _
                            Basket.Fields("Week_4").Value, "", _
                            Basket.Fields("Week_5").Value, "", _
                            Basket.Fields("Week_6").Value, "", _
                            Basket.Fields("Week_7").Value, "", _
                            Basket.Fields("Week_8").Value, "", _
                            Basket.Fields("Week_9").Value, "", _
                            Basket.Fields("Week_10").Value, "", _
                            Basket.Fields("Week_11").Value, "", _
                            Basket.Fields("Week_12").Value, "")
                    Basket.MoveNext
                Wend
            End If
        Case "Broker Group"
            If Basket.EOF Then
                [TRError] = "No records for filter or period"
            Else
                While Not Basket.EOF
                    [TRError] = ""
                        ' change 'nullgroup' to 'Unassigned'
                        GroupBrokerName = IIf((Basket.Fields("GroupBrokerName").Value) = "", "Unassigned", Basket.Fields("GroupBrokerName").Value)
                        AddTableRow "TR_Table", _
                            Array(GroupBrokerName, _
                            Basket.Fields("Week_1").Value, "", _
                            Basket.Fields("Week_2").Value, "", _
                            Basket.Fields("Week_3").Value, "", _
                            Basket.Fields("Week_4").Value, "", _
                            Basket.Fields("Week_5").Value, "", _
                            Basket.Fields("Week_6").Value, "", _
                            Basket.Fields("Week_7").Value, "", _
                            Basket.Fields("Week_8").Value, "", _
                            Basket.Fields("Week_9").Value, "", _
                            Basket.Fields("Week_10").Value, "", _
                            Basket.Fields("Week_11").Value, "", _
                            Basket.Fields("Week_12").Value, "")
                    Basket.MoveNext
                Wend
            End If
        Case "Country"
            If Basket.EOF Then
                [TRError] = "No records for filter or period"
            Else
                While Not Basket.EOF
                ' add data to excel
                    [TRError] = ""
                    ' change 'nullbroker' to 'Unallocated'
                    ParentCountry = IIf((Basket.Fields("ParentCountry").Value) = "", "Unassigned", Basket.Fields("ParentCountry").Value)
                    AddTableRow "TR_Table", _
                        Array(ParentCountry, _
                        Basket.Fields("Week_1").Value, _
                            Basket.Fields("Week_1").Value, "", _
                            Basket.Fields("Week_2").Value, "", _
                            Basket.Fields("Week_3").Value, "", _
                            Basket.Fields("Week_4").Value, "", _
                            Basket.Fields("Week_5").Value, "", _
                            Basket.Fields("Week_6").Value, "", _
                            Basket.Fields("Week_7").Value, "", _
                            Basket.Fields("Week_8").Value, "", _
                            Basket.Fields("Week_9").Value, "", _
                            Basket.Fields("Week_10").Value, "", _
                            Basket.Fields("Week_11").Value, "", _
                            Basket.Fields("Week_12").Value, "")
                    Basket.MoveNext
                Wend
            End If
        Case "Pooling Code"
            If Basket.EOF Then
                [TRError] = "No records for filter or period"
            Else
                While Not Basket.EOF
                    [TRError] = ""
                        ' change '' to 'No Code'
                        PoolingCode = IIf((Basket.Fields("PoolingCode").Value) = "", "No Code", Basket.Fields("PoolingCode").Value)
                        AddTableRow "TR_Table", _
                            Array(PoolingCode, _
                            Basket.Fields("Week_1").Value, "", _
                            Basket.Fields("Week_2").Value, "", _
                            Basket.Fields("Week_3").Value, "", _
                            Basket.Fields("Week_4").Value, "", _
                            Basket.Fields("Week_5").Value, "", _
                            Basket.Fields("Week_6").Value, "", _
                            Basket.Fields("Week_7").Value, "", _
                            Basket.Fields("Week_8").Value, "", _
                            Basket.Fields("Week_9").Value, "", _
                            Basket.Fields("Week_10").Value, "", _
                            Basket.Fields("Week_11").Value, "", _
                            Basket.Fields("Week_12").Value, "")
                        Basket.MoveNext
                Wend
            End If
    End Select
        
    Basket.Close
    LinkToServer.Close
    Set LinkToServer = Nothing
    
End Sub
```

```
Sub TRHowLong()
' Report showing average time between transactions in hours per user
    
    Dim LinkToServer As New ADODB.Connection
    Dim LoginAccess As String
    Dim SQLSyntax As String
    Dim Basket As New ADODB.Recordset ' recordset is fetched in.
    Dim VBAWeekStartDate As Date
    Dim SQLWeekStartDate As String
    Dim VBAWeekEndDate As Date
    Dim SQLWeekEndDate As String
    Dim UserId As Integer
    Dim WkCount As Integer
    Dim RowCount As Integer
        
    [TRError] = ""
    
    RowCount = 1
    
    LoginAccess = [HDConnectionString]
    
    LinkToServer.ConnectionTimeout = 30
    LinkToServer.Open LoginAccess
    
    Do While Range("TR_Table[Filter]")(RowCount) <> "Total"
    
        UserId = GetUserID(Range("TR_Table[Filter]")(RowCount))
        Range("TR_Table[Filter]")(RowCount).Select
        
        VBAWeekStartDate = CDate([TRStartDate])
        SQLWeekStartDate = VBAtoSQLUpdateFormatDate(VBAWeekStartDate)
        VBAWeekEndDate = DateAdd("d", 6, VBAWeekStartDate)
        SQLWeekEndDate = VBAtoSQLUpdateFormatDate(VBAWeekEndDate)
            
        For WkCount = 1 To 12
            
            SQLSyntax = "SELECT T1,T2, AVG(DATEDIFF) as average_hours, US.Username from " & _
                        "( " & _
                            "SELECT  t1.TransactionDesc as T1, t2.TransactionDesc as T2, " & _
                            "DATEDIFF(hour,MIN(t1.lastupdated), MAX(t2.lastupdated)) as datediff, t2.UserId " & _
                            "FROM dbo.TransactionAudit as t1, dbo.TransactionAudit as t2 " & _
                            "WHERE t1.IsCancelled <> 'Yes' " & _
                            "AND t2.IsCancelled <> 'Yes' " & _
                            "AND t1.QuoteLevel = 0 " & _
                            "AND t2.QuoteLevel = 0 " & _
                            "AND t1.OpportunityID = t2.OpportunityID " & _
                            "AND t1.TransactionDesc = '" & [TRTransaction] & "'" & _
                            " AND t2.TransactionDesc = '" & [TRTransaction2] & "'" & _
                            " AND t2.LastUpdated >= " & SQLWeekStartDate & _
                            " AND t2.LastUpdated <= " & SQLWeekEndDate & _
                            " AND t2.UserID = " & UserId & _
                            " GROUP BY t1.OpportunityID, t1.TransactionDesc, t2.TransactionDesc, t2.UserId " & _
                            ") as tempTable " & _
                        "LEFT JOIN dbo.Users AS US " & _
                        "ON tempTable.UserID = US.UserID " & _
                        "GROUP BY T1, T2, US.Username"

            Basket.Open SQLSyntax, LinkToServer
            
            If Not Basket.EOF Then
                Cells(RowCount + 12, WkCount * 2 + 3) = Basket.Fields("average_hours").Value
            Else
                Cells(RowCount + 12, WkCount * 2 + 3) = 0
            End If
     
            Basket.Close
            
            VBAWeekStartDate = DateAdd("d", 7, VBAWeekStartDate)
            SQLWeekStartDate = VBAtoSQLUpdateFormatDate(VBAWeekStartDate)
            VBAWeekEndDate = DateAdd("d", 7, VBAWeekEndDate)
            SQLWeekEndDate = VBAtoSQLUpdateFormatDate(VBAWeekEndDate)
            
        Next WkCount
    
        RowCount = RowCount + 1
    
    Loop
    
    LinkToServer.Close
    Set LinkToServer = Nothing
    
End Sub
```
