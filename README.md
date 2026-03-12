# sp_DBPermissions
This is a free tool from **[DataPaws](https://datapawsconsulting.com)** for SQL Server Database Administrators to gain visibility into permissions at the database-level, as well as quickly drop/create them.

This is a continuation of [sp_DBPermissions by Kenneth Fisher](https://github.com/sqlstudent144/SQL-Server-Scripts/blob/main/sp_DBPermissions.sql)

# What does sp_DBPermissions do?
This stored procedure returns 3 data sets. The first dataset is the list of database
principals, the second is role membership, and the third is object and database level
permissions.

You can specify the following for @Output:
```
		CreateOnly 	- Returns scripts to create both the database principal and grant database permissions
		DropOnly 	- Returns scripts to drop database permissions
		ScriptsOnly - Returns drop and create scripts
		Report 		- Returns one output with one row per principal and a comma delimited list of roles the principal is a member of and a comma delimited list of the individual permissions they have.
```
```
Parameters:
    @DBName
        If NULL use the current database, otherwise give permissions based on the parameter.
    
        There is a special case where you pass in ALL to the @DBName.  In this case the SP
        will loop through all of the DBs in sys.databases and run the queries into temp tables
		before returning the results.  WARNINGS: If you use this option and have a large number
		of databases it will be SLOW.  If you use this option and don't specify any other parameters
		(say a specific @Principal) and have even a medium number of databases it will be SLOW. Also
		the undo/do scripts do not have USE statements in them so please take that into account.
    @Principal
        If NOT NULL then all three queries only pull for that database principal.  @Principal
        is a pattern check.  The queries check for any row where the passed in value exists.
        It uses the pattern '%' + @Principal + '%'
    @Role
        If NOT NULL then the roles query will pull members of the role.  If it is NOT NULL and
        @DBName is NULL then DB principal and permissions query will pull the principal row for
        the role and the permissions for the role.  @Role is a pattern check.  The queries 
        check for any row where the passed in value exists.  It uses the pattern '%' + @Role +
        '%'
    @Type
        If NOT NULL then all three queries will only pull principals of that type.  
        S = SQL login
        U = Windows login
        G = Windows group
        R = Server role
        C = Login mapped to a certificate
        K = Login mapped to an asymmetric key
    @ObjectName
        If NOT NULL then the third query will display permissions specific to the object 
        specified and the first two queries will display only those users with those specific
        permissions. This parameter uses the pattern '%' + @ObjectName + '%'
    @Permission
        If NOT NULL then the third query will display only permissions that match what is in
        the parameter.  The first two queries will display only those users with that specific
        permission.
    @LoginName
        If NOT NULL then each of the queries will only pull back database principals that
        have the same SID as a login that matches the pattern '%' + @LoginName + '%'
	@UseLikeSearch
		When this is set to 1 (the default) then the search parameters will use LIKE (and 
		%'s will be added around the @Principal, @Role, @ObjectName, and @LoginName parameters).  
        When set to 0 searchs will use =.
	@IncludeMSShipped
		When this is set to 1 (the default) then all principals will be included.  When set 
		to 0 the fixed server roles and SA and Public principals will be excluded.
	@DropTempTables
		When this is set to 1 (the default) the temp tables used are dropped.  If it's 0
		then the temp ables are kept for references after the code has finished.
		The temp tables are:
			##DBPrincipals
			##DBRoles 
			##DBPermissions
	@Output
		Type of output desired
		Default 	- Either 'Default' or it doesn't match any of the allowed values then the SP will return the standard 3 outputs.
		None		- No output at all.  Usually used if you keeping the temp tables to do your own reporting.
		CreateOnly 	- Returns scripts to create both the database principal and grant database permissions
		DropOnly 	- Returns scripts to drop database permissions
		ScriptsOnly - Returns drop and create scripts
		Report 		- Returns one output with one row per principal and a comma delimited list of roles the principal is a member of and a comma delimited list of the individual permissions they have.
    @Print
        Defaults to 0, but if a 1 is passed in then the queries are not run but printed
        out instead.  This is primarily for debugging.
    
Data is ordered as follows
    1st result set: DBPrincipal
    2nd result set: RoleName, UserName if the parameter @Role is used else
                    UserName, RoleName
    3rd result set: ObjectName then Grantee_Name if the parameter @ObjectName
                    is used otherwise Grantee_Name, ObjectName
```
