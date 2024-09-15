<small>Dirty Read Possible,	Non Repeatable Read Possible,	Phantom Read Possible are issues.<br>
<i>(Below isolation level shows which issue can occur if you choose which isolation level):</i>
</small>

| Isolation Level   |<small><i>(Can Problem occur?)</small></i><br> Dirty Read Possible            |<small><i>(Can Problem occur?)</small></i><br> Non Repeatable Read Possible          |<small><i>(Can Problem occur?)</small></i><br> Phantom Read Possible
|----------|---------------------------------------------------------------------------|----------------------------------------------------------------------|-----------------|
|READ_UNCOMMITTED|Yes|Yes|Yes|
|READ_COMMITTED|No|Yes|Yes|
|REPEATABLE_COMMITTED|No|No|Yes|
|SERIALIZABLE|No|No|No|

## ***Dirty Read Problem***

| Time | Transaction A | Transaction B    | DB Status
|----------|---------------------------------------------------------------------------|----------------------------------------------------------------------|-----------------|
|T1 |BEGIN TRANSACTION |BEGIN TRANSACTION |<small>Id:123,</br>Status:Free</small>
|T2 | |Update row set <small>Status:Booked</small> |<small> Id:123,</br>Status: Booked</br></small><i><small>(Not committed By Transaction B yet)</i></small>|
|T3 |<i><small>(Read Row)</i></small></br><small>Id:123,</br>Status:Booked</small></br><i><small>(Got status=Booked) </i></small>||<small>Id:123,</br>Status: Booked</br></small><i><small>(Not committed By Transaction B yet)<i><small>|
|T4| |Rollback |<small>Id:123,</br>Status:Free</small></br><i><small>(Uncommited Changes of Transaction B got rolled back)</i></small>

>

1. ***Transaction B (Not Committed):***
    - Updates Status to "Booked".
    - Change stays in memory or a log; it hasn't been persisted to the actual database storage yet.
2. ***Transaction A Reads:***
    - In Read Uncommitted mode/DirtyRead, Transaction A can read this temporary, uncommitted value (Status: Booked), which is risky because it might not be the final state.
3. ***Transaction B Commits:***
    - The change (Status: Booked) is written to the actual    database storage and is visible to all subsequent transactions.
4. ***Transaction B Rolls Back:***
    - The change is discarded, and the status remains Status: Free in the main storage. Transaction A, however, might have already read the incorrect (dirty) value.
  
>Note:As dirty read voilates the ACID property it is only used in case where there is read only data.(No write should be done)


## ***Non Repeatable Read Problem***


|Time|	Transaction A|	Transaction B|	DB Status </br><i><small>(initial: Free)</i></small>|
|------|-------|-----|-----|
|T1|	BEGIN TRANSACTION|		|<small>Id:123, </br>Status: Free</small>
|T2|	<i><small>Read Row</i></small></br><small> Id:123,</br> Status: Free	|BEGIN TRANSACTION|	<small>Id:123, Status: Free</small>|
|T3	| |<i><small>Update Row<br></i></small> <small>	Id:123, <br>Status: Booked,</small><br> then COMMIT	|<small>Id:123,</br> Status: Booked</small>
|T4|	<small><i>Read Row</i><br> Id:123,<br> Status: Booked</small|	|	<small>Id:123,<br> Status: Booked</small>

>Note:Regarding whether Transaction A can read status:booked if Transaction B had not committed is rely on Dirty Read Possible.

***Breakdown of the Scenario:***
- T1: Transaction A starts.
- T2: Transaction A reads the status of Id:123, which is "Free."
- T3: Transaction B starts and updates Id:123 to Status: Booked. It then commits the change, making it visible to all other transactions.
- T4: Transaction A reads the same row again, but now the status has changed to "Booked" because Transaction B committed the update.

Here, Transaction A reads the same row (Id:123) twice but gets different results:

   - In T2, it reads Status: Free.
   - In T4, it reads Status: Booked due to the committed changes from Transaction B.
  

## ***Phantom Read Problem***

- It is same as non repeatable read but the query is targeting rows instead of data