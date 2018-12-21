When we run jobsl scripts from a pipeline, it asks for script approval for every jobdsl script. And if you are creating multiple jobs through it, it will ask for every job to approve it. 
So to prevent it and run jobdsl without approvals, Go to Manage Jenkins -> Configure Global Security and uncheck 		
Enable script security for Job DSL scripts.
