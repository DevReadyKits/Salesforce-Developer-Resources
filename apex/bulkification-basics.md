# Apex Bulkification Basics

Bulkification means writing Apex so it can safely process multiple records in a single transaction.

This is essential in Salesforce because triggers, APIs, data loads, Flow, and asynchronous processes can all execute against collections of records.

---

## The Core Rule

When you see code processing multiple records, think:

```text
Collect
  ↓
Query once
  ↓
Process in memory
  ↓
DML once
```

The main goal is to avoid performing expensive operations repeatedly for each record.

---

## Avoid SOQL Inside Loops

### Avoid

```apex
for (Account acc : accounts) {
    List<Contact> contacts = [
        SELECT Id, Email
        FROM Contact
        WHERE AccountId = :acc.Id
    ];
}
```

This executes another SOQL query for every Account in the loop.

That may work with a few records, but it does not scale safely.

---

## Collect IDs First

### Prefer

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

Now the query runs once for the complete collection.

---

## Avoid DML Inside Loops

### Avoid

```apex
for (Account acc : accounts) {
    acc.Description = 'Updated';
    update acc;
}
```

This performs one DML operation per iteration.

---

## Perform DML Once

### Prefer

```apex
List<Account> accountsToUpdate = new List<Account>();

for (Account acc : accounts) {
    acc.Description = 'Updated';
    accountsToUpdate.add(acc);
}

if (!accountsToUpdate.isEmpty()) {
    update accountsToUpdate;
}
```

The pattern is:

```text
Build the collection
        ↓
Perform one DML operation
```

---

## Use Sets for IDs

Sets are useful when you need unique values.

```apex
Set<Id> accountIds = new Set<Id>();

for (Contact contactRecord : contacts) {
    if (contactRecord.AccountId != null) {
        accountIds.add(contactRecord.AccountId);
    }
}
```

Because a `Set` keeps values unique, the same Account ID will not be added multiple times.

---

## Use Maps for Fast Record Access

Maps are useful when you need to retrieve a record by ID.

```apex
Map<Id, Account> accountsById = new Map<Id, Account>([
    SELECT Id, Name, Industry
    FROM Account
    WHERE Id IN :accountIds
]);
```

Then you can access a record directly:

```apex
Account acc = accountsById.get(accountId);
```

This is usually cleaner and faster than repeatedly looping through a list looking for a matching record.

---

## Filter Before Querying

In update triggers, do not process every record if only a few actually changed.

Use `Trigger.oldMap` to compare the previous and current values.

```apex
Set<Id> changedAccountIds = new Set<Id>();

for (Account acc : Trigger.new) {
    Account oldAcc = Trigger.oldMap.get(acc.Id);

    if (acc.Industry != oldAcc.Industry) {
        changedAccountIds.add(acc.Id);
    }
}
```

Now downstream logic can work only with the relevant records.

---

## Keep Triggers Thin

A trigger should coordinate the transaction, not contain all business logic.

### Trigger

```apex
trigger AccountTrigger on Account (after update) {
    AccountTriggerHandler.afterUpdate(
        Trigger.new,
        Trigger.oldMap
    );
}
```

### Handler

```apex
public with sharing class AccountTriggerHandler {

    public static void afterUpdate(
        List<Account> newRecords,
        Map<Id, Account> oldMap
    ) {
        Set<Id> changedAccountIds = new Set<Id>();

        for (Account acc : newRecords) {
            Account oldAcc = oldMap.get(acc.Id);

            if (acc.Industry != oldAcc.Industry) {
                changedAccountIds.add(acc.Id);
            }
        }

        if (changedAccountIds.isEmpty()) {
            return;
        }

        // Continue with bulk-safe logic here.
    }
}
```

This keeps the trigger easy to read and makes the business logic easier to test.

---

## Bulkify Helper Methods Too

Moving code into a helper class does not automatically make it bulk-safe.

### Avoid

```apex
public static void processAccount(Id accountId) {

    List<Contact> contacts = [
        SELECT Id
        FROM Contact
        WHERE AccountId = :accountId
    ];

    // Process contacts...
}
```

If that method is called inside a loop, it still creates one query per Account.

---

## Accept Collections Instead

### Prefer

```apex
public static void processAccounts(Set<Id> accountIds) {

    List<Contact> contacts = [
        SELECT Id, AccountId
        FROM Contact
        WHERE AccountId IN :accountIds
    ];

    // Process the entire collection.
}
```

Design helper methods around collections whenever the calling context may process multiple records.

---

## Query Only the Fields You Need

Avoid retrieving unnecessary fields.

### Less efficient

```apex
List<Account> accounts = [
    SELECT Id, Name, Industry, Phone, Website, BillingCity, BillingCountry
    FROM Account
];
```

If the logic only needs `Id` and `Industry`, query only those fields.

### Better

```apex
List<Account> accounts = [
    SELECT Id, Industry
    FROM Account
];
```

Smaller queries are easier to understand and reduce unnecessary data processing.

---

## Guard Against Empty Collections

Before performing work, check whether there is anything to process.

```apex
if (accountIds.isEmpty()) {
    return;
}
```

And before DML:

```apex
if (!accountsToUpdate.isEmpty()) {
    update accountsToUpdate;
}
```

This keeps intent clear and avoids unnecessary operations.

---

## Test With Multiple Records

A test that only inserts one record does not prove that your code is bulk-safe.

Example:

```apex
@IsTest
static void processesMultipleAccounts() {

    List<Account> accounts = new List<Account>();

    for (Integer i = 0; i < 200; i++) {
        accounts.add(
            new Account(
                Name = 'Account ' + i,
                Industry = 'Technology'
            )
        );
    }

    Test.startTest();
    insert accounts;
    Test.stopTest();

    System.assertEquals(200, accounts.size());
}
```

Also test mixed scenarios where:

- some records meet the condition;
- some records do not;
- multiple records reference the same parent;
- fields may contain null values.

---

## Common Bulkification Smells

Watch for these during code review:

- SOQL inside a loop
- DML inside a loop
- helper methods that accept only one record or one ID
- repeated queries for the same data
- nested loops that could use a Map
- processing every Trigger record when only changed records matter
- unnecessary queries before checking whether collections are empty

---

## Quick Checklist

Before committing Apex code, verify:

- [ ] No SOQL inside loops
- [ ] No DML inside loops
- [ ] Use Sets for unique IDs
- [ ] Use Maps for related record lookup
- [ ] Query collections instead of individual records
- [ ] Query only required fields
- [ ] Filter records before expensive operations
- [ ] Use `Trigger.oldMap` when relevant
- [ ] Keep triggers thin
- [ ] Helper methods accept collections
- [ ] Guard against empty collections
- [ ] Test with multiple records
- [ ] Test mixed and edge-case scenarios

---

## DevReady Tip

Whenever you see this:

```text
for each record
    ↓
query
    ↓
update
```

stop and ask whether it can become:

```text
collect everything
        ↓
query once
        ↓
process everything
        ↓
DML once
```

That simple change prevents many governor-limit problems.

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
