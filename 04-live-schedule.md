# The Live Schedule

Once we have finalized a time cycle (filled), we can now roll it out to a live schedule.

This has some serious implications because we should not be able to schedule on the live
schedule table on date and times where there are already scheduled entries.

Some issues to address:

* What records can be deleted?
* When can a record be deleted?
* How do we pick out on the live schedule which ones to delete? Can we delete in bulk?
* How do we manually put something on a live schedule that is not from a time cycle
  and now do we manage conflicts then?