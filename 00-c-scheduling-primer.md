# Continuous Scheduling
In many applications, there are entities that are scheduled to start at a specific point
in time, has duration, and consequently, ends at a specific point in time. The entity can
be someone's shift at work, or when a specific video clip goes on the air for a TV station,
for instance. The aim of this undertaking is to write abstract code that will be able to
schedule anything. When it is complete, any application that needs to schedule something
to occur at some time in the future can utlize this library.

This is NOT a library that schedules tasks to run against operating system time, and therefore,
it is not concurrent process or parallel task management. This system isn't also a block-type
of scheduling system such as one that schedules a teacher to teach a particular course, at a
particular classroom, in a particular timeblock. This is a system that works directly with
continuous time and scheduling things in a timeline.

## Aspects To Tackle
1. **Avoiding Conflicts**. There will need to be a mechanism that should disallow scheduling of
an event when its start time or end time falls inside a time interval of something already scheduled.
2. **Time Cycles**. In the real world, many things are scheduled not on an absolute
point in "global time", but at a specific point within a single time cycle. For example, a TV
show isn't explicitly scheduled to play on a specific date. It is scheduled at a specific time
on which day[s] of the week. In this case, the time cycle is one week. However, an implementer
of this system should get to define their own time cycle - could be one day, two days, one week,
two weeks, etc. When coding this in the abstract, we do not know exactly how the implementor
defines a single time cycle, and we don't know how each time cycle would be broken down. We
will discuss time cycle abstraction in a later article as it entails many considerations and
complexities.
3. **Explicit Scheduling**. There also needs to be a capability to schedule something specific
at a point in absolute "global time". Issues to be addressed include conflict detection against
something scheduled on a time cycle, managing the event when the explicitly-scheduled event
starts or ends within the time slot of something scheduled on a time cycle.
4. **Keeping Saved Data Minimal**. Since we are working with a continuous timeline, we are
free to schedule anything at any time, and it can be as long or short as it needs to be. This
means that it won't start at convenient, nice, and rounded times, like right at the beginning
of the hour, for example. However, if care isn't taken, we could end up associating an event
with thousands, if not, millions, of unique small time blocks, which is overkill.