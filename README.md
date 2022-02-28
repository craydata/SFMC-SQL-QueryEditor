## Query subscribers who opened emails sent from the Marketing Cloud in the previous 30 days.

This query finds subscribers who opened an email in the last 30 days, regardless of how many sending jobs used that EmailID. Setting up this query starts with creating a target data extension to store the query's data output. Then, create the query, selecting the data extension you created as the target data extension.

```
Select
j.JobID,
j.EmailName,
j.DeliveredTime as SendTime,
o.EventDate as OpenTime,
s.EmailAddress,
s.SubscriberKey
    from [_Job] j
    join [_Open] o on j.JobID = o.JobID
    join [_Subscribers] s on o.SubscriberID = s.SubscriberID
        where
        o.IsUnique = 1 and
        o.EventDate > dateadd(d,-30,getdate()) and
        j.EmailID = <EmailID> 
```
## Query: Find Subscriber Status

Retrieve subscribers' statuses.

Use this query to return subscribers in a data extension and their status for a specified list or the All Subscribers list. Setting up this query starts with creating a target data extension to store the query's data output. Then, create the query, selecting the data extension you created as the target data extension.
```
Select
l.ListName,
de.SubscriberKey,
l.[Status]
from [<DE_Name>] de
    join [_listsubscribers] l
    on de.SubscriberKey = l.SubscriberKey
        where
        l.ListName = '<List Name>'
```


## Query: Query Subscribers in a Publication or Suppression List

Considerations

Use this query to pull statuses for any list or group.
The available status values are “active”, “bounced”, “held”, and “unsubscribed.”

```
Select
l.SubscriberKey,
l.EmailAddress,
l.ListName,
l.ListType,
l. [Status],
l.CreatedDate as DateAdded,
l.DateUnsubscribed
    from [_ListSubscribers] l
        where ListName = '<List Name>'
```


## Query Subscribers by Date or Time Frame

Write a query using Automation Studio’s query activity to find subscribers according to the date or time frame.

This query pulls a list of Subscribers on a list and the date they were added, or that were added in a particular time frame. Setting up this query starts with creating a target data extension to store the query's data output. Then, create the query, selecting the data extension you created as the target data extension.

```
Select
    l.SubscriberKey,
    l.EmailAddress,
    l.ListID,
    l.ListName,
    l.CreatedDate as DateAdded
        from [_ListSubscribers] l
            where
             l.ListID = <ListID> and CreatedDate between '2015-11-01' and '2015-12-01'
```

## Query Subscribers with No Opens or Clicks

This query's results show all subscribers that were sent a job, but did not click or open the email tied to a particular JobID. Setting up this query starts with creating a target data extension to store the query's data output. Then, create the query, selecting the data extension you created as the target data extension.

```
Select distinct
s.SubscriberKey,
s.JobID,
s.BatchID,
convert(char(19),s.EventDate,20) as SendDate
    from [_sent] s
    left join [_open] o
    on s.JobID = o.JobID and s.ListID = o.ListID and s.BatchID = o.BatchID and s.SubscriberID = o.SubscriberID and o.IsUnique = 1
    left join [_click] c
    on s.JobID = c.JobID and s.ListID = c.ListID and s.BatchID = c.BatchID and s.SubscriberID = c.SubscriberID and c.IsUnique = 1
        where
        s.JobID = JobID
        and (o.SubscriberID is NULL and c.SubscriberID is NULL)
```
Considerations
This query runs well for jobs under 500,000 subscribers. For larger jobs, consider using Intermediate Tables to ensure optimal query performance.


## Query Top Bounces for a Job
Write a query using Automation Studio’s query activity to find the top bounces by percentage.

This query shows the top 25 bounces, as a percentage, for a particular job. Setting up this query starts with creating a target data extension to store the query's data output. Then, create the query, selecting the data extension you created as the target data extension. This complex query uses multiple queries and intermediate data extensions. This query runs optimally for jobs that include fewer than 5,000,000 subscribers.


```
Select
s.JobID,
s.Domain,
count (s.SubscriberID) as SendCount
from [_Sent] s
    where 
    s.JobID = <JobID>
    group by s.JobID, s.Domain
```

## Query: Journey Builder Bounced Email Messages
Create a list of contacts to send a direct mailer to based on bounced email messages from Journey Builder.

Use this query to create a list of contacts based on bounced email messages.


```
select
j.JourneyName,
j.VersionNumber,
ja.ActivityName as 'EmailName',
s.EventDate as 'SendTime',
b.EventDate as 'BounceTime',
su.EmailAddress,
su.SubscriberKey as 'ContactKey',
su.SubscriberID as 'ContactID',
s.JobID,
s.ListID,
s.BatchID,
cpd.[Address Line 1],
cpd.[Address Line 2],
cpd.City,
cpd.State,
cpd.Zip,
cpd.HomePhone,
cpd.MobileNumber 
    from [_Sent] s
    join [_JourneyActivity] ja 
    on s.TriggererSendDefinitionObjectID = ja.JourneyActivityObjectID
    join [_Journey] j
    on ja.VersionID = j.VersionID
    join [_Bounce] b
    on s.JobID = b.JobID
    and s.ListID = b.ListID
    and s.BatchID = b.BatchID
    and s.SubscriberID = b.SubscriberID
    join [_Subscribers] su
    on s.SubscriberID = su.SubscriberID
    join ContactProfileData cpd
    on s.SubscriberKey = cpd.ContactKey
        where ja.ActivityType in ('EMAIL','EMAILV2')
        and j.JourneyName = <JourneyName>
        and s.EventDate < cast(cast(dateadd(hh,-72,getdate()) as date) as datetime)
        and b.SubscriberID is not null

```

# Query: Journey Builder Sends by Email Across Versions
Create an aggregate of Journey Builder Email Sends across multiple versions of the same journey.

Use this query to create an aggregate of Journey Builder Email Sends across multiple versions of the same journey.

```
select j.JourneyName,
cast(s.EventDate as date) as [Date],
ja.ActivityName as 'EmailName',
ja.ActivityExternalKey,
count(s.SubscriberID) as [Sends]
from [_Sent] s
join [_JourneyActivity] ja 
on s.TriggererSendDefinitionObjectID = ja.JourneyActivityObjectID
join [_Journey] j
on ja.VersionID = j.VersionID
join [_Subscribers] su
on s.SubscriberID = su.SubscriberID
    where ja.ActivityType in  ('EMAIL','EMAILV2')
    and j.JourneyName = <JourneyName>
    and s.EventDate > dateadd(dd,-7,getdate())
      group by j.JourneyName,j.JourneyID,cast(s.EventDate as date),ja.ActivityName,ja.ActivityExternalKey

```

# Query Journey Builder Sends in Last 24 Hours
Find subscribers who were sent an email from a specified journey in Journey Builder within the last 24 hours.

Use this query to find subscribers who were sent an email from a specified journey within the last 24 hours.

```
select
j.JourneyName,
j.VersionNumber,
ja.ActivityName as 'EmailName',
s.EventDate as 'SendTime',
su.EmailAddress,
su.SubscriberKey as 'ContactKey',
s.SubscriberID as 'ContactID'    ,
s.JobID,
s.ListID,
s.BatchID
    from [_Sent] s
    join [_JourneyActivity] ja 
    on s.TriggererSendDefinitionObjectID = ja.JourneyActivityObjectID
    join [_Journey] j
    on ja.VersionID = j.VersionID
    join [_Subscribers] su
    on s.SubscriberID = su.SubscriberID
        where ja.ActivityType in  ('EMAIL','EMAILV2')
        and j.JourneyName = <JourneyName>
        and s.EventDate > dateadd(hh,-24,getdate())
```

#

```

```

#

```

```
