---
layout: post
title: "Entity Framework Migrations - No logical space left to create more user strings"
date: 2012-07-15 18:41
comments: true
categories: [Entity Framework Migrations]
---
Yesterday we started seeing a compile time exception:

    Unexpected error writing metadata to file 'e:\….\obj\Release\…Repository.EntityFramework.dll'
    -- 'No logical space left to create more user strings'
 
As it turns out, this related to us having too many migration scripts, in our case, > 90.
<!--more-->
The problem is that each time we generate a new script, a snapshot of the model schema is hashed into the migration designer IMigrationMetadata.Target property. 

As we have a lot of developers working on the solution, we reached the limit pretty quickly.  I believe MS know about this issue and are talking about trying to resolve in EF6.

How to resolve?  Well we could use resx files as was outlined here but, for us, this is a non runner for various reasons.  Instead we have had to re-baseline the migration scripts.  Not ideal but that’s what we came up with.

##Environments
We have them in abundance.  Developer local, CI, Test, Stage, Stage Integration, Production.  Each one is in a different state so what to do?

We checked all the environments and looked at the __MigrationHistory table to see the latest script that was common to all.  Then we went to the script prior to that.  That’s our baseline target.

Create the baseline

Next, as we have seeded data in our app, we need to get this data in a usable format.  We rolled back a dev machine to the change set related to our baseline target script.  Dump the local database, then recreate.  Next in SQL Management Studio, right click the database and Tasks –> generate scripts.  Select the Advanced option and set ‘Types of Data to Script’ to Data only.  This will generate a bunch of INSERT statements.

Drop the local database again.

Back in the project, delete all the migration script files.  Add-Migration a new baseline one.  This will create the schema for you.  Add the insert statements as Sql(“INSERT….”); in your script.  We have quite a crude mechanism to prevent any of this script running in on any non-debug environment:

```c#
#if !DEBUG 
  return; 
#endif
```

Hey, it works for us!

We then tweaked the timestamp in the name of the script and also in the `IMigrationMetadata.Id` property of the designer to a little after the target baseline script.  This keeps it looking correct.

Finally, get latest from source control (remember, we were at the change set of the target baseline script up to now).

##Deploy
When we deployed and the migrations kicked in, it ran our new baseline script (which does nothing as we have our  #if !DEBUG in there). But the application fails with:

```
Unable to update database to match the current model because there are pending changes and automatic 
migration is disabled. Either write the pending model changes to a code-based migration or enable 
automatic migration. Set DbMigrationsConfiguration.AutomaticMigrationsEnabled to true to enable 
automatic migration.
```

So another little tweak, this time to the `__MigrationHistory` table.  Find the entry for the baseline script and change the CreatedOn date to a little after the target baseline script.

All good.

Conclusion

While what we did is pretty hacky, it gets us out of a bind.  10+ developers can now get back to work and, until MS resolve the issue, at least we have a ‘resolution’.

EDIT - and they have.  The designer now utilises a resx file to store the hash.