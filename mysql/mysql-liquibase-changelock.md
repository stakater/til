# MYSQL liquibase changelock

Applications that are configured to use liquibase for state management of MYSQL might come across the issue where **liquibase
acquires change lock** and as a result locks the database. Firstly, it's a feature not a bug and this usually
occurs if liquibase is stalled/application is restarted while it is verifying the current state of the database, multiple
instances of liquibase are executed against the same database concurrently or due to faulty liquibase changesets. The lock is 
set to ensure that only one instance of Liquibase is running at any given time.

## To fix this issue run the following query:
`UPDATE DATABASECHANGELOGLOCK SET LOCKED=FALSE, LOCKGRANTED=null, LOCKEDBY=null where ID=1;`

## or drop the table, it will be re-created by liquibase
`DROP TABLE DATABASECHANGELOGLOCK;`

Although, this is not a neat or auotmated fix. You can create a side car container that will scrape API logs for the following
string `Could not acquire change log lock` and whenever that string is found it will run the above given queries on MYSQL.
