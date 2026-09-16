# Useful SOQL Queries

A small collection of practical SOQL examples for everyday Salesforce development.

## Basic Query

```sql
SELECT Id, Name
FROM Account
LIMIT 50

## LIKE

SELECT Id, Name
FROM Account
WHERE Name LIKE '%Cloud%'

## IN

SELECT Id, Name
FROM Account
WHERE Id IN :accountIds

## Date Literal

SELECT Id, Name, CreatedDate
FROM Opportunity
WHERE CreatedDate = LAST_N_DAYS:30

## Child to Parent

SELECT Id, FirstName, LastName, Account.Name
FROM Contact

## Parent to Child

SELECT Id, Name,
    (SELECT Id, FirstName, LastName
     FROM Contacts)
FROM Account

## Aggregate Query

SELECT StageName, COUNT(Id)
FROM Opportunity
GROUP BY StageName

## Active Users
SELECT Id, Name, Username, Profile.Name
FROM User
WHERE IsActive = true
ORDER BY Name

## Tip

Query only the fields and records you actually need.

Avoid using SOQL inside loops when the same query can be executed once for a collection.

## Want the complete toolkit?

The **Salesforce Developer Survival Kit** includes practical guides, checklists, code examples, and downloadable source files.

[Get the Salesforce Developer Survival Kit](https://payhip.com/DevReadyKits)
