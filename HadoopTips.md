Hadoop Tips

The symptom here is that your code in your reduce phase is "stuck", either because of an infinite loop or just a ludicrous amount of data received, or something else (maybe post your reduce code?).

Here are the way that percentages work in the reducer:

0-33% is the shuffle. This is data moving from the mappers to the reducers (see how it starts before the mappers are finished).
33%-67% is the sort. This can only start when the mappers are finished (see how it goes from 30% to 67% after map is at 100%).
67%-100% is the actual reduce code you are running. This percentage goes up every time a reduce task completes. None of your reduce tasks are completing.
In the JobTracker interface, look at your job and see how much data the reducers are getting in. If the number of records in the reducer is going up, that means you probably have too much data going to the reducers. If that number stays still, you might have an infinite loop of some sort.