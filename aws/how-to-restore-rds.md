# How to restore RDS database?

When doing the restoration following you be done:

- announce downtime; as no changes should happen; as they will be lost!
- reduce number of pods to zero

Steps:

Lets say we are resuming a database called: keycloak-dev

- By default a daily snapshot is kept for last 30 days.
- Click on the database you want to restore
- Go to snapshots section
- Find the snapshot you want to restore to
- Click on the snapshot
- Once snapshot page is open; click Actions and select Restore Snapshot
- Specify the DB Instance Identifier; e.g. keycloak-dev-restored
- Ensure both VPC & Subnet are same as the main db
- Then click Restore DB Instance
- (Note: this will create a new database instance)
- Once its created; then modify it and set the password & security groups to the same as the db being restored
- Change the database name of old db; e.g. keycloak-dev-broken
- Change the databse name of new restored db to keycloak-dev
- 
