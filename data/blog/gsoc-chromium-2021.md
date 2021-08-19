---
title: 'GSoC 2021 with Chromium'
date: '2021-08-19'
tags: ['gsoc', 'development', 'open-source', 'chromium', 'golang']
draft: false
layout: PostSimple
images: ['/static/images/gsoc.png', '/static/images/chromium.png']
summary: 'My experience and Work Summary as a Chromium GSoC Intern, 2021.'
---

<div className="flex flex-wrap justify-center	 -mx-2 overflow-hidden xl:-mx-2">
  <div className="my-1 px-2 overflow-hidden xl:my-1 xl:px-2 xl:w-1/2">
    <Image alt="gsoc" src="/static/images/gsoc.png" width={80} height={80} />
  </div>
  <div className="my-1 px-2 overflow-hidden xl:my-1 xl:px-2 xl:w-1/2 ml-6">
    <Image alt="chromium" src="/static/images/chromium.png" width={80} height={80} />
  </div>
</div>

This post is about my GSoC project, **porting ChromeOS power policy end-to-end tests**, that I worked on during the summer, 2021. It gives a brief summary of my overall contribution to chromium from pre-GSoC to the end of GSoC as a requirement of the final evaluation. It all started in May 2021 with the community bonding. The sweet voyage comes to an end in August 2021 with an incredible learning experience, a lot of fun and some tricky challenges.

## Project Details

**Idea** : `Porting ChromeOS Power Policy End-to-End Tests`

**Official Requirement Docs** : [Port power policies end-to-end tests](https://docs.google.com/document/d/1CS1gCt8DmtGX39mFuTs2m306mK_vTT6T_VUHjg3Wo4w/edit?usp=sharing&resourcekey=0-_7vfr0BYbkzGOGvh8ToYmQ)

**Project Page** : [summerofcode](https://summerofcode.withgoogle.com/projects/#6687401885302784)

**Summary** : Integration tests of power policies in `ChromeOS` utilize `autotest` for automated testing. There are several problems with autotest.

- The power policy tests are not stable and fail randomly.
- It's super tedious to find the exact error from its vast log trails.
- These policy testing requires a particular debugging device, `servo` connected to the `DUT` (Device Under Test), a Chromebook, to perform various hardware-level debugging stuff, including operating DUT charging state, power state etc. There is no documentation in autotest on the requisite setup to replicate the intended behaviour for these policy tests.
- Also, the autotest framework is old and still uses `Python2` that has already reached the end of life officially in January 2020.

Instead of making a complete overhaul by migrating the whole autotest to `Python3`, the ChromeOS team has decided to port the policy tests in a fast golang based automated testing framework [Tast](https://chromium.googlesource.com/chromiumos/platform/tast) gradually and decommission the autotest eventually.  
As a Chromium GSoC intern, my work is to migrate five power policy tests by understanding one of the possible test setups, port and document them and make them stable. The policies are `PowerPeakShift`, `BatteryChargeMode`, `USBPowershare`, `BootOnAC` and `AdvancedBatteryChargeMode`.

**Mentors** 👨

- Oleh Lamzin
- Mahmoud Gawd

**Technology Used** : `golang`, `Tast`, `python`, `autotest`, `gRPC`

**My Gerrit Profile URL** : [Chromium Gerrit](https://chromium-review.googlesource.com/q/owner:bisakhmondal00%2540gmail.com)

## Pre-GSoC Work

As my project requires access to `chromebooks` & `servos`, so the starter work that had been asked was a bit different for the initial evaluation. I was responsible for creating a `Gerrit` (Google Git) scraper in `golang` and content parser-analyser to index the commits and reviews made by the chromium authors using the chrome `CDP` (chrome devtools protocol). Without fetching the pages in generic way through `HTTP.GET`, in a nutshell, the program loads the page in headless chrome instance and communicates with it directly with devtools protocol (CDP). `Tast` heavily relies on CDP websocket connection for the required communication. Finally, it provides a CLI to fetch and parse commit messages, per author commit-review counts and a dockerfile with the build steps to run everything in an isolated containerized environment.

**Full Problem Details** : [GSoC - Port power policies end-to-end tests: starter bug](https://docs.google.com/document/d/1dxkJFTJ4RCUfmzl8AbNLtJPtH39Y8QOnnE4SVlRhwAY/edit?usp=sharing)

**Solution Implemented** : [bisakhmondal00/cdp-go](https://github.com/bisakhmondal/cdp-go)

## Community Bonding Progress

I had spent the whole three weeks setting up the ChromeOS development setup, creating a new chroot, building images, exploring `Tast` framework, completing tast codelabs and also understanding the Chromium git-flow on a single monolithic repository. My mentor, Oleh, shared some bugs and I picked one ([chromium:1142132](https://bugs.chromium.org/p/chromium/issues/detail?id=1142132)) and started working on the fix to have a hands-on with the chromium Gerrit. Along the way, I also worked on implementing a `TODO` feature on `tast-lint`.

✅️ [CL:2919074](https://chromium-review.googlesource.com/c/chromiumos/platform/tast/+/2919074) : Better Handling of `fmt.Errorf` in `tast-lint`( Improvement over existing implementation by prunning the entire sub parse-tree for `ast` selector expression with an ability to deal with complex scenarios. )

✅️ [CL:2919426](https://chromium-review.googlesource.com/c/chromiumos/platform/tast/+/2919426) : **Allow Linting from `git` Subdirectories** where previously `tast-lint` could only be run from repository git root.

## Change Logs

Here is the list of CLs with brief info that I worked on during the GSoC coding period from June 7 to August 16.

✅️ [CL:3056633](https://chromium-review.googlesource.com/c/chromiumos/platform/tast-tests/+/3056633) : **`servo` package migration from remote to common**. Remote tests relies on gRPC to communicate with DUT for policy testing but certainly there are some cases which only require a servo connection in local tests, e.g. test that doesn't require a restart. In my case, with this change, _PeakShift_, _BatterCharge_ & _AdvancedBatterCharge_ has been made possible to write as local tests.

✅️ [CL:2987941](https://chromium-review.googlesource.com/c/chromiumos/platform/tast-tests/+/2987941) : **Battery Charge - Drain Utilities**. Implements the required functionality to ensure the DUT is within a required batter range to satisfy the requirements of certain power policies.

✅️ [CL:3058137](https://chromium-review.googlesource.com/c/chromiumos/platform/tast-tests/+/3058137) : **Peak Shift Local Policy Tests**. Test the behaviour of the `DevicePowerPeakShiftEnabled` power management policy that if enabled reduces AC usage in peak hours.

✅️ [CL:3064937](https://chromium-review.googlesource.com/c/chromiumos/platform/tast-tests/+/3064937) : **Battery Charge Mode Local Policy Test**. Test the behaviour of the `DeviceBatteryChargeMode` power management policy that if enabled minimizes battery stress and wear-out by using _standard charge/ fast charge/ adaptive charge_ depending upon the policy enrollment value.

✅️ [CL:3071878](https://chromium-review.googlesource.com/c/chromiumos/platform/tast-tests/+/3071878) : **Advanced Battery Charge Mode Local Policy Test**. Test the behaviour of the `DeviceAdvanced BatteryChargeModeEnabled` power management policy that if enabled maximizes the battery health by using a standard charging algorithm and other techniques during non-working hours.

✅️ [CL:2973093](https://chromium-review.googlesource.com/c/chromiumos/platform/tast-tests/+/2973093) : **Boot on AC Remote Policy Test**. Test the behaviour of the `DeviceBootOnAcEnabled` policy that if enabled reboots the DUT from power off state when connected to an AC power supply.

✅️ [CL:3075461](https://chromium-review.googlesource.com/c/chromiumos/platform/tast-tests/+/3075461) : **USB Power Share Remote Policy Test**. Test the behaviour of the `DeviceUsbPowerShareEnabled` policy that if enabled shares power through `USB VBUS` in power off state.

🚧 [CL:3090465](https://chromium-review.googlesource.com/c/chromiumos/platform/tast-tests/+/3090465) : **Wilco Device Setup Documentation**. It provides the elaborate details of the test lab setup and summarizes important facts of servo devices required for writing and debugging power management policies. Effectively, it fills the gap as a setup guide that we are lacking in autotest.

---

The planned GSoC milestones have been achieved with the aforelisted CLs. Out of curiosity, I spent some time implementing `DeviceRebootOnShutdown` policy tests and migrating Wilco DTC remote tests to local in tast with the required changes.

🚧 [CL:3080647](https://chromium-review.googlesource.com/c/chromiumos/platform/tast-tests/+/3080647) : **Reboot on Shutdown Remote Policy Tests**. If enabled, the policy replaces all shutdown buttons in the ui with restart buttons.

🚧 [CL:3088795](https://chromium-review.googlesource.com/c/chromiumos/platform/tast-tests/+/3088795) : **Wilco DTC Enrolled Fixtures**. Provides a fixture with `Wilco DTC VM` & `Supportd` daemon running returns chrome & fakedms object for policy enrollment.

🚧 [CL:3085093](https://chromium-review.googlesource.com/c/chromiumos/platform/tast-tests/+/3085093) : Local test migration of Wilco DTC `GetStatefulPartitionAvailableCapacity` gRPC method.

🚧 [CL:3089877](https://chromium-review.googlesource.com/c/chromiumos/platform/tast-tests/+/3089877) : Local test migration of Wilco DTC `PerformWebRequest` gRPC method.

🚧 [CL:3097629](https://chromium-review.googlesource.com/c/chromiumos/platform/tast-tests/+/3097629) : Local test migration of Wilco DTC `RunRoutineRequest` and `GetRoutineUpdate` gRPC methods.

( **Notion Used** : ✅️ &rarr; CL has been Committed to `cros/main` Head, 🚧 &rarr; CL In-Review )

## End Note

So, in the end, a big thanks to Google and Chromium for this opportunity. I want to thank particularly my mentor, Oleh, for guiding me throughout the whole process, igniting cool ideas and for the awesome in-depth code reviews. This project itself was an interesting experience for me since it was entirely new. I've never worked at such an os and hardware level. Exploring and understanding two separate codebases was exhilarating. I got to learn how to write efficient, self-contained Go code.

This GSoC is ending but Chromium has won a new contributor. I hope it's just the beginning of a new story.
