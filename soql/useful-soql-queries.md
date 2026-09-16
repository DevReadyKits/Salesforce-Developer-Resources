# Useful SOQL Queries

A small collection of practical SOQL examples for everyday Salesforce development.

---

## Basic Query

```sql
SELECT Id, Name
FROM Account
LIMIT 50
```

## LIKE

Use `LIKE` when you need to search for partial text matches.

```sql
SELECT Id, Name
FROM Account
WHERE Name LIKE '%Cloud%'
```

## IN

Use `IN` when filtering against a collection of values or IDs.

```sql
SELECT Id, Name
FROM Account
WHERE Id IN :accountIds
```

## Date Literal

Salesforce provides useful date literals for relative date filtering.

```sql
SELECT Id, Name, CreatedDate
FROM Opportunity
WHERE CreatedDate = LAST_N_DAYS:30
```

Other useful date literals include:

- `TODAY`
- `YESTERDAY`
- `THIS_WEEK`
- `THIS_MONTH`
- `LAST_N_DAYS:30`
- `NEXT_N_DAYS:30`

## Child to Parent

Use dot notation to access fields from a parent record.

```sql
SELECT Id, FirstName, LastName, Account.Name
FROM Contact
```

## Parent to Child

Use a subquery to retrieve related child records.

```sql
SELECT Id, Name,
    (
        SELECT Id, FirstName, LastName
        FROM Contacts
    )
FROM Account
```

## Aggregate Query

Aggregate functions can help summarize records directly in SOQL.

```sql
SELECT StageName, COUNT(Id)
FROM Opportunity
GROUP BY StageName
```

Common aggregate functions include:

- `COUNT()`
- `COUNT_DISTINCT()`
- `SUM()`
- `AVG()`
- `MIN()`
- `MAX()`

## Active Users

A useful query for finding active Salesforce users.

```sql
SELECT Id, Name, Username, Profile.Name
FROM User
WHERE IsActive = true
ORDER BY Name
```

## Multiple Conditions

Use `AND` and `OR` to combine filters.

```sql
SELECT Id, Name, StageName, Amount
FROM Opportunity
WHERE IsClosed = false
AND Amount > 10000
ORDER BY Amount DESC
```

## Null Values

You can filter records based on whether a field contains a value.

```sql
SELECT Id, Name, Email
FROM Contact
WHERE Email != null
```

## Limit Results

Use `LIMIT` when you only need a specific number of records.

```sql
SELECT Id, Name, CreatedDate
FROM Case
ORDER BY CreatedDate DESC
LIMIT 100
```

---

## DevReady Tip

Query only the fields and records you actually need.

Avoid using SOQL inside loops when the same query can be executed once for an entire collection.

Instead of this:

```apex
for (Account acc : accounts) {
    List<Contact> contacts = [
        SELECT Id, Email
        FROM Contact
        WHERE AccountId = :acc.Id
    ];
}
```

Prefer collecting the IDs first:

```apex
Set<Id> accountIds = new Set<Id>();

for (Account acc : accounts) {
    accountIds.add(acc.Id);
}

List<Contact> contacts = [
    SELECT Id, Email, AccountId
    FROM Contact
    WHERE AccountId IN :accountIds
];
```

The general pattern is:

```text
Collect IDs
    ↓
Query once
    ↓
Process the collection
```

---

## Want the Complete Toolkit?

The **Salesforce Developer Survival Kit** includes:

- Salesforce Developer Survival Guide
- SOQL Cheat Sheet
- Apex Bulkification Checklist
- Salesforce Debugging Checklist
- Salesforce Deployment Checklist
- Git + Salesforce CLI Cheat Sheet
- Practical Apex & SOQL Code Examples
- Downloadable source files

[Get the Salesforce Developer Survival Kit](https://payhip.com/DevReadyKits)

---

## About DevReady Kits

**DevReady Kits** creates practical resources for real-world software development.

**Code. Debug. Test. Deploy. Learn.**

DevReady Kits is an independent resource provider and is not affiliated with, endorsed by, or sponsored by Salesforce.
