General
o Inconsistent data types for the same column name – which smell like they should be the same:-
 dbo.IFA.[Deleted] vs. dbo.UserName.[Deleted]
 dbo.Opportunity.[IFAContactName] vs. dbo.QuoteVersion.[IFAContactName]
 dbo.Opportunity.[OpportunityName] vs. dbo.QuoteVersion.[OpportunityName]
o Normalisation
 Any entity which is repeatedly re-used/referenced in the data should normally be broken out into separate tables, for example:-
• Users (you have the table, but doesn’t look like you reference the ID)
• Contacts
• Brokers
• Addresses
• Opportunities
o No Foreign Keys to enforce referential Integrity across related tables
 For example UserName
o There are no non-clustered indexes
 Think about which tables will grow significantly and by which column(s) will be searched or joined to
o No Check Constraints to enforce valid data, for example
 Email addresses are ‘something@something.something’
 Ages should be between 0 and 130 (to be fair, I think you’re used TINYINT which allows 0-255)
o No Unique Constraints
 are there any natural keys:- email, policy number, BrokerName, etc. where uniqueness should be enforced in a table?
o Use of reserved words (ideally avoid):-
 Deleted
 Version
 Password
 Application
o There are lots of column names that use business acronyms
 Consider expanding these to avoid confusion for future Support/Development staff
o Use consistent naming
 SomethingNumber v. SomethingNum
 SomethingDate vs. DateSomething
o I don’t see any BIT columns
 these are useful for anything which is yes|no, [has] not, [not-]required, etc.
 For example dbo.UserName.[Protected]
 Also by convention these are normally prefixed ‘Is’ or ‘Has’ such as:-
• IsRequired
• IsDeleted
• HasChildren
• IsCancelled
o The fewer NULL’able columns the better, use defaults where possible
 Some string columns could default to ‘TBC’
 Some monetary columns could default to £0.00
o If you’re not going to exceed 2B+ rows (!) consider changing all INT identify columns to SMALLINT or TINYINT
• dbo.ApplicationControl
o PK spelt wrongly
o You’ve already got a [UserName] table – don’t store the user name here again (store the ID)
o [Version] smells like something which should be an integer or maybe a numeric (not a string)
o If you’re storing passwords how are you encrypting them?
• dbo.Chaser
o You’ve already got a [UserName] table – don’t store the user name here again (store the ID)
• dbo.DocumentProductionControl
o By convention the IDENTITY column PK would normally be called [TableNameId]
• dbo.IFA
o Could this be too specifically named
 Maybe ‘Advisor’ instead
o Avoid acronyms in object (table) names?
o You’ve already got a [UserName] table – don’t store the user name here again (store the ID)
• dbo.Opportunity
o You’ve already got a [UserName] table – don’t store the user name here again (store the ID)
• dbo.Rate
o You’ve already got a [UserName] table – don’t store the user name here again (store the ID)
o Gender VARCHAR(5) (“Female” = 6)
• dbo.TransactionAudit
o You’ve already got a [UserName] table – don’t store the user name here again (store the ID)
• dbo.Username
o [TheUser] – strange name for a column?
• dbo.UWNote
o Avoid acronyms in object (table) names?
