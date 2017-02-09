    Data = "COUNT(*) AS total_opportunities, " & _
            "SUM(OP.NumberOfLives) AS total_lives, " & _
            "SUM(CASE WHEN QT.QuoteNumber = 1 THEN QT.TotalPremium ELSE NULL END) AS total_premium, " & _
            " COUNT(CASE WHEN OP.OpportunityStatus = 'Awaiting Initial Checks' THEN 1 else NULL END) AS aic_count, " & _
            " COUNT(CASE WHEN OP.OpportunityStatus = 'Awaiting Initial Checks Review' THEN 1 else NULL END) AS aicr_count, " & _
            " COUNT(CASE WHEN OP.OpportunityStatus = 'Awaiting Quote(s) Production' THEN 1 else NULL END) AS aqp_count, " & _
            " COUNT(CASE WHEN OP.OpportunityStatus = 'Awaiting Revised Quote(s) Production' THEN 1 else NULL END) AS arqp_count, " & _
            " COUNT(CASE WHEN OP.OpportunityStatus = 'Awaiting Best Rates Quote(s) Production' THEN 1 else NULL END) AS abrqp_count, " & _
            " COUNT(CASE WHEN OP.OpportunityStatus = 'Awaiting Quote Pack Sending' THEN 1 else NULL END) AS aqps_count, " & _
            " COUNT(CASE WHEN OP.OpportunityStatus = 'Issued' THEN 1 else NULL END) AS issued_count, " & _
            " COUNT(CASE WHEN OP.OpportunityStatus = 'Awaiting Entry on Phoenix' THEN 1 else NULL END) AS aeop_count, " & _
            " COUNT(CASE WHEN OP.OpportunityStatus = 'Closed' THEN 1 else NULL END) AS closed_count, " & _
            " COUNT(CASE WHEN OP.OpportunityStatus = 'Awaiting Decline Authorisation' THEN 1 else NULL END) AS ada_count, " & _
            " COUNT(CASE WHEN OP.OpportunityStatus = 'Declined' THEN 1 else NULL END) AS declined_count, " & _
            " COUNT(CASE WHEN OP.OpportunityStatus = 'Cancelled' THEN 1 else NULL END) AS cancelled_count " & _
            "FROM dbo.Opportunity AS OP " & _
            "LEFT JOIN dbo.Quote AS QT " & _
            "ON OP.OpportunityID = QT.OpportunityID "

      DateRange = "WHERE OP.ReceivedDate >= " & VBAtoSQLUpdateFormatDate(StartDate) & "" & _
                  "AND OP.ReceivedDate <= " & VBAtoSQLUpdateFormatDate(endDate) & ""

      Select Case [FRFilterBy]

          Case "Allocated UW"

              SQLSyntax = "SELECT US.Username, " & _
                  Data & _
                  "LEFT JOIN dbo.Users AS US " & _
                  "ON OP.AllocatedUnderwriterID = US.UserID " & _
                  DateRange & _
                  "GROUP BY US.Username " & _
                  "ORDER BY US.Username"
