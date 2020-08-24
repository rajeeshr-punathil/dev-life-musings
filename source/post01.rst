Day 1: Distance in days between Dates in Swift
===============================================

*Date* : **22 Aug 2020**

*Topics* : ``Swift``, ``macOS``, ``iOS``


Recently, my phone has developed a display issue and I wanted to continue using a couple of my essential apps from the laptop. Unfortunately, there weren't any desktop or web versions available. At the same time, I have been trying my hands at developing apps for apple ecosystem using Swift. So I thought like, my essential features aren't a big deal, so why not try porting them to MacOS?

One of the apps I planned just needs to track a specific habit for now. It will save the date each time the user performs that action. User needs to perform the action at least once everyday, no matter which exact time it is. What matters is the date only. Each time user opens the app, our code needs to compare current date with the date of their previous activity. As long as the user performs the action at least once every day, a streak is maintained. If the distance between the dates of 2 consecutive activities is of more than 1 day, that means that the streak is broken. So in summary, this is all about finding the absolute day-distance between two given dates.

A (short) Ode to Python before we start ;)
----------------------------------------------

If we were using Python, we could store the Dates as ``date`` objects and finding the distance between two dates would be as simple as follows (Since we are going to discuss Swift, let's make the function look as much Swift as it can, using function annotation): 

.. code-block:: python

    """An example implementation using Python"""

    from datetime import date

    def distance_in_days(date1: date, date2: date) -> int :
        """Return absolute distance in days between given dates."""
        return abs((date2 - date1).days)

The above function can be tested as follows:

.. code-block:: python

   from datetime import date

    date1 = date(2020, 8, 13)
    date2 = date(2020, 8, 14)

    assert distance_in_days(date1, date2) == 1, "Calendar went wrong!"
    # Both must be same since we are finding the absolute distance.
    assert distance_in_days(date2, date1) == 1, "Calendar went wrong!"


Back to the Swift land 
------------------------

There's no date-only class in Swift. Swift's ``Date`` is more like Python's ``datetime``. An equivalent solution in Swift to the above would be,

.. code-block:: swift

    func distanceInDays(from date1: Date, to date2: Date) -> Int {
        return abs(Calendar.current.dateComponents(
            [.day], from: date1, to: date2
        ).day ?? 0)
    }

Since I needed to use this function so often in the app in different modules, I decided to extend the builtin ``Date`` and make this available everywhere.

.. code-block:: swift

    extenstion Date {
        func distanceInDays(from other: Date) -> Int {
            return abs(Calendar.current.dateComponents(
                [.day], from: self, to: other
            ).day ?? 0)
        }
    }

Now let's test it.

.. _test_code_zero:

Testing sample 01
~~~~~~~~~~~~~~~~~~~

.. code-block:: swift

    let comp1 = DateComponents(year: 2020, month: 08, day: 13)
    let comp2 = DateComponents(year: 2020, month: 08, day: 14)
    let d1 = Calendar.current.date(from: comp1)
    let d2 = Calendar.current.date(from: comp2)

    print("Date 1: \(d1!) and Date 2: \(d2!)")
    print("Distance in days: \(d1!.distanceInDays(from: d2!))")

Output is,

.. code-block:: sh

    Date 1: 2020-08-12 18:30:00 +0000 and Date 2: 2020-08-13 18:30:00 +0000
    Distance in days: 1

Now, we think that our solution works fine, but it does not. Since swift doesn't support Date-only data types as said before, Our app would be storing timestamps of when the user perfomed the activity. That includes time components and timezone also.  Let's try another example with time components as well.

.. _test_code_one:

Testing sample 02
~~~~~~~~~~~~~~~~~~~

.. code-block:: swift

    let comps1 = DateComponents(year: 2020, month: 08, day: 21, hour: 23, minute: 55)
    let comps2 = DateComponents(year: 2020, month: 08, day: 23, hour: 23, minute: 54)
    let comps3 = DateComponents(year: 2020, month: 08, day: 24, hour: 23, minute: 53)
    let date1 = Calendar.current.date(from: comps1)
    let date2 = Calendar.current.date(from: comps2)
    let date3 = Calendar.current.date(from: comps3)
    
    print("Date 1: \(date1!) and Date 2: \(date2!)")
    print("Distance in days: \(date1!.distanceInDays(from: date2!))")
    print()
    print("Date 2: \(date2!) and Date 3: \(date3!)")
    print("Distance in days: \(date2!.distanceInDays(from: date3!))")

Date1 is on Aug 21 and Date2 is on Aug 23. So we expect the distance between them to be 2 days. And distance between Date2 (Aug23) and Date3 (Aug 24) must be 1 day. Let's check the otput:

.. code-block:: sh

    Date 1: 2020-08-21 18:25:00 +0000 and Date 2: 2020-08-23 18:24:00 +0000
    Distance in days: 1

    Date 2: 2020-08-23 18:24:00 +0000 and Date 3: 2020-08-24 18:23:00 +0000
    Distance in days: 0

Results are 1 less than what we expected. Why? Because technically, the difference between ``date1`` and ``date2`` is less than 2 days (``1 day 23 hours and 59 minutes``). Similarly distance between ``date2`` and ``date3`` is ``0 days 23 hours and 59 minutes``, 1 minute short of 1 day. So Swift is just returning the day component of the absolute difference between the given timestamps. To fix this, Our function needs to cancel out the time components of both timestamps before finding the distance. Let's also cancel out the timezone components as well. Our improved function will look like this:

.. code-block:: swift

    extension Date {
        // Return distance between corresponding days, ignoring the time components.
        func distanceInActualDays(from other: Date) -> Int {
            let tz = TimeZone.init(abbreviation: "GMT")
            var myComponents = Calendar.current.dateComponents(in: tz!, from: self)
            myComponents.hour = 12
            myComponents.minute = 0
            myComponents.second = 0
            var otherComponents = Calendar.current.dateComponents(in: tz!, from: other)
            otherComponents.hour = 12
            otherComponents.minute = 0
            otherComponents.second = 0
            return abs(
                Calendar.current.dateComponents(
                    [.day],
                    from: otherComponents,
                    to: myComponents
                ).day!
            )
        }
    }

We normalized both timestamps by setting the timezone to GMT and time components to 12 Noon of the given dates. Let's test this improved code with :ref:`test_code_one` again: 

Now, the output is,

.. code-block:: sh

    Date 1: 2020-08-21 18:25:00 +0000 and Date 2: 2020-08-23 18:24:00 +0000
    Distance in days: 2

    Date 2: 2020-08-23 18:24:00 +0000 and Date 3: 2020-08-24 18:23:00 +0000
    Distance in days: 1

We have made it right this time? Seems so. But wait, let's check again with another example:

.. _test_code_two:

Testing sample 03
~~~~~~~~~~~~~~~~~~~

.. code-block:: swift

    let date4 = Date(timeIntervalSince1970: 1597257913.450566)
    let date5 = Date(timeIntervalSince1970: 1597346178.289075)

    print("Distance between date 4(\(date4)) & date 5 (\(date5)): \(date4.distanceInDays(from: date5))")

Output is,

.. code-block:: sh

    Distance between date 4 (2020-08-12 18:45:13 +0000) & date 5 (2020-08-13 19:16:18 +0000): 0

Given dates are on Aug 12 and Aug 13, we expect the distance to be 1 day, but our function returns 0. This error may occur, even when you are not initiating dates as in the last example. Swift stores ``Date`` instances internally as a float that represents the seconds elapsed since Jan 01 2001 00:00:00 GMT. So even a difference of a few milliseconds may result in this unexpected behaviour. One solution to this is to add one second to the bigger normalized timestamp. We update our code again as follows:

.. code-block:: swift

    extension Date {
        // Return distance between corresponding days, ignoring the time components.
        func distanceInActualDaysWithBugfix(from other: Date) -> Int {
            let tz = TimeZone.init(abbreviation: "GMT")
            var myComponents = Calendar.current.dateComponents(in: tz!, from: self)
            myComponents.hour = 12
            myComponents.minute = 0
            myComponents.second = 0
            var otherComponents = Calendar.current.dateComponents(in: tz!, from: other)
            otherComponents.hour = 12
            otherComponents.minute = 0
            otherComponents.second = 0
            // Always add 1 second to the bigger date to compensate for possible loss of milliseconds.
            if (self > other) {
                myComponents.second = 1
            } else if (self < other) {
                otherComponents.second = 1
            }
            return abs(
                Calendar.current.dateComponents(
                    [.day],
                    from: otherComponents,
                    to: myComponents
                ).day!
            )
        }
    }

Run above testing samples, :ref:`test_code_one` and :ref:`test_code_two` again and you can see that we get the expected results now.

Extra serving
--------------

If we want to get timestamps in local time, we can add the following function also to the above ``Date`` extension:

.. code-block:: swift

    func localDate() -> Date {
        return self.addingTimeInterval(
            TimeInterval(TimeZone.current.secondsFromGMT())
        )
    }
