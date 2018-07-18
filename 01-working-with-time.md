# Working with Time

Time is the singlemost important part of this entire framework. It is quite complicated
so we need to settle some concepts before we can schedule to it.

## The Time Cycle Concept

A time cycle is a finite duration of time that events may be scheduled on. Since it is a
cycle, it is meant to be repeated, thus, rolled onto an actual global absolute timeline. For
example, a TV channel may line up shows on a weekly schedule. That weekly schedule is the
time cycle, and will play week after week.

## The Smallest Unit Of Time in the Abstract

Ideally, a continuous timeline should be infinitely precise, meaning, that the smallest unit
time interval is infinitely short. We are working with a machine, and we can only get so
precise. Even with the most precise allowances of a machine, though, we will get unwanted
errors.

In this system, though, we shall define, in abstract code, that the smallest unit of time
will be one second. In most applications, though, scheduling to the minute is sufficient,
however, we will further refine it to seconds just in case.

Realistically, most time cycles will not exceed 1 week. Sometimes, a cycle is 2 weeks. The
integer is sufficient enough to hold values upto a little over 2 billion. Suppose we have
a time cycle that is 52 weeks long (approximately 1 year). That is only 31,449,600 seconds,
which an integer can readily represent. So, one or two weeks (the most common cases), will
suffice.

## Time Conversion

Since time data (start time and duration) will be stored in seconds, we will need to convert
that to real time for display and user input purposes. For flexibility, we will create an 
interface whose responsibility is to convert back and forth to and from a desired time object:

```csharp
interface ITimeConverter<TTime>
   where TTime : class
{
   TTime ConvertToTime(int secondMark);
   int ConvertToSecondMark(TTime currentTime);
}
```

The implementer can, if they want, decide the `TTime` type parameter as they wish in case
they want to reinterpret what our 1 time unit should be. We will provide several
time converters out of the box, with the type parameters of `DateTime`, `TimeSpan`,
and `WeekScheduleTime`. `WeekScheduleTime` is a structure we will create that has week
(0 to max integer), day (0 - 6), hour (0 - 23), minute (0 - 59), and second (0 - 59) -
convenient for our purposes.

## Determinining a Time Cycle for which to Schedule
We roughly defined a time cycle, but we haven't yet established exactly how an implementer
determines it. As mentioned, a time cycle is a duration of time, and our smallest time unit
will be 1 seconds. So, a time cycle really is a single integer - the total number of seconds
there are in that single time cycle.

We provide the following enumaration that includes the most common time cycle durations:

```csharp
public enum TimeCycleDuration
{
   Hourly = 3600,
   Daily = 86400,
   Bidaily = 172800,
   Weekly = 604800,
   Biweekly = 1209600
}
```