    SQLSyntax = " SELECT TransactionDesc, IsCancelled, QuoteLevel, QuoteReference," & _
                    " TA.UserID as UN, TA.LastUpdated as LU, Notes," & _
                    " (SELECT COUNT(*) FROM dbo.TransactionAudit as CT" & _
                    " WHERE TA.OpportunityID = CT.OpportunityID" & _
                    IIf(QuoteLevel, " AND TA.QuoteLevel = CT.QuoteLevel)", ")") & " as counter" & _
                    " FROM dbo.TransactionAudit as TA " & _
                    " LEFT JOIN Quote as QT " & _
                    " On TA.OpportunityID = QT.OpportunityID " & _
                    " AND TA.QuoteLevel = QT.QuoteNumber " & _
                    " WHERE TA.OpportunityID = " & [HDOpportunity]
                
                
Discovered I can use a VBA inline If statement within a SQL query string.
