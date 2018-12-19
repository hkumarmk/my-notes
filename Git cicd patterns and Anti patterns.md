# Git development and cicd patterns and Anti patterns

## MS visual studio usecase
MS Visual studio team was using different long running branches -

* There is one master/main branch
* Each team create a branch from it - and on and off they merge to master and back from master to that branch
* There may be other subgroups branch the code from team branch and they also do merge back and forth
* Subgroups may create feature branches
* Then there may be release branches etc

This make a "merge hell".

### They moved to trunk based developement where
* Only single long running branch that is master branch
* Everyone who work on fixes/bugs/features, fork a feature branch, make smallest change possible, PR back to master
* Any incomplete features should be masked from endusers using feature flags, rather than hiding the code from other
team members

#### This improves the situation by 

* Less merge conflicts
* More visibility within the team - any changes that are made by one dev is visible to other team and thus they can
reused updated
code asap
* Easy to review the code because now its very small change to be reviewed

## Github flow on continuous deployment

* There is only one long running branch - master
* Every dev fork from master for their own feature branches - features/bug fixes
* Once done with the dev (as small change as possible), he create a PR to master
* All tests etc would be performed, and code is reviewed
* Now he request "hubot" to deploy code from PR branch to **production**
* hubot queues his request - ONLY ONE DEV CAN DEPLOY THE CHANGE IN PRODUCTION AT ONCE
* on his turn, hubot does:
  * Lock master, so that noone can merge to it
  * merge master to PR branch
  * run build, test, deploy to canary, wait for any failures, deploy to production wait for failures
  * Once all done, hubot report back to the user "Done with prod deployment, if you satisfied with the change,
  you may merge this to master; for rollback, deploy master
* Unlock the master
NOTE: BUT I think hubot should merge the changes to master before unlocking master because, what if PR1 get passed,
then PR2 deployment is done, then PR1 get merged to master, and then PR2 get merged to master - probably the combinnation of
PR1 and PR2 may created a bug which never get tested untill another PR get tested, i.e PR3. now PR3 failed on prod, and
I wanted to rollback, but rollback on master also get failed, because the failed/buggy code is in master.

This one perfectly work - because every change is going to prod, rollback is easy etc etc.
BUT This method can affect scaling - because only one change can go production at once.

## Visual studio team take a different approach
* one long running branch i.e master
* every dev fork from master for their own feature branches - features/bug fixes
* Once done with the dev (as small change as possible), he create a PR to master
* All tests etc would be performed, and code is reviewed
* Code then merged to master
* After end of every sprint (3 or 4 weeks sprint), a release/sprint branch is forked from master
* Test release/sprint branch, and deploy it to prod
* The reason why release branch because if there any issue they see in the deployment, can be fixed in the release branch
rather than getting all changes in master to prod

### Rollback problem if you have data/schema change
* No conflicting changes in the data layer
* If you want to remove/change any column/data, you have to copy the data to new place, with appropriate change, and let
column/table/data intact, but mark that one as obsolete somewhere
* Deployment system can look for any obsolete data and delete the data that are at least 2 releases old
* This enable rollback in ncase of any issue
  
## Reference
https://medius.studios.ms/Embed/Video/THR2017?sid=THR2017
https://guides.github.com/introduction/flow/
