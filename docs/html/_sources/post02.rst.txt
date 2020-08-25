Day 2: Prevent sleeping of macOS or iOS device in Swift
========================================================

*Date* : **23 Aug 2020**

*Topics* : ``Swift``, ``macOS``, ``iOS``

Our swift app may sometimes need to prevent the idle-sleep behaviour of the device it runs on. To do this in iOS is very simple, we just need to write a single line of code. Then after performing the task, we can restore the original behaviour by running another single line code. This is possible in macOS too, only that our solution is a little longer to write, but still simple. The following example code deals with both macOS and iOS. Code is self-explanatory:

.. code-block:: swift

    /* 1. Place the following import lines where all other imports are. */
    #if os(macOS)
        import IOKit
        import IOKit.pwr_mgt
    #endif

    /* 2. Place the following line where the variable declarations are. */
    #if os(macOS)
        @State var preventSleepID: IOPMAssertionID = 0
    #endif

    /* 3. The sleep-preventing function. */
    func preventSleep() {
        #if os(iOS)
            UIApplication.shared.isIdleTimerDisabled = true
        #else if os(macOS)
            let reasonForActivity = "DESCRIBE WHY APP NEEDS TO PREVENT SLEEP IN SIMPLE WORDS." as CFString
            let preventSleep = IOPMAssertionCreateWithName(
                // To prevent display-sleep, use kIOPMAssertionTypeNoDisplaySleep below.
                kIOPMAssertionTypeNoIdleSleep as CFString,
                IOPMAssertionLevel(kIOPMAssertionLevelOn),
                reasonForActivity,
                &preventSleepID
            )
            if preventSleep != kIOReturnSuccess {
                print("Idle-sleep could not be prevented. System may go to sleep midway.")
            }
        #endif
    }

    /* 4. The function that restores sleep behaviour. */
    func restoreSleep() {
        #if os(iOS)
            UIApplication.shared.isIdleTimerDisabled = false 
        #else if os(macOS)
            if preventSleepID != 0 {
                IOPMAssertionRelease(preventSleepID)
                preventSleepID = 0
            } else {
                print("Idle-sleep Already restored?")
            }
        #endif
    }

    /* 5. A sample function that invokes preventSleep */
    func startTask() {
        // Do other things and then..
        preventSleep()
    } 

    /* 6. A sample function that invokes restoreSleep after finishing the task. */
    func finishTask() {
        // Do things that finishes the task and then..
        restoreSleep()
    }
