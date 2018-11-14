# How to restore a RDS database?

Before the restoration following should be done:

- Announce downtime; as no changes should happen during the restoration period; as they will be lost.
- Reduce number of pods to zero for the app whose database is going to be restored; to ensure that database is being changed

Lets say we are resuming a database called: `keycloak-dev`

- (By default a daily snapshot is kept for last 30 days)
- Click on the database you want to restore (go to an older version)
- Go to snapshots section
- Find the snapshot you want to restore to
- Click on the snapshot
- Once snapshot page is open; click Actions and select Restore Snapshot
- Specify the DB Instance Identifier; e.g. `keycloak-dev-restored`
- Ensure both VPC & Subnet are same as the main db (which is being restored)
- Then click Restore DB Instance
- (Note: this will create a new database instance with new name/identifier)
- Once its created; then modify it and set the password & security groups to the same as the db being restored
- Change the database name of old db; e.g. `keycloak-dev-broken`
- Change the databse name of new restored db to `keycloak-dev`
- (one option could be to use 
- Scale up the pods again; and verify everything is up and running
- If all looks good then the older instance of database i.e. `keycloak-dev-broken` can be deleted
