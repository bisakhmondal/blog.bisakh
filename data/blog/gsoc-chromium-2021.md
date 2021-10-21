---
title: 'GSoC 2021 with Chromium'
date: '2021-08-20'
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

This post is about my GSoC project, **porting ChromeOS power policy end-to-end tests**, that I worked on during the summer, 2021. It gives a brief summary of my overall contribution to Chromium from pre-GSoC to the end of GSoC as a requirement of the final evaluation. It all started in May 2021 with the community bonding. The sweet voyage comes to an end in August 2021 with an incredible learning experience, a lot of fun and some tricky challenges.

## Project Details

**Idea** : `Porting ChromeOS Power Policy End-to-End Tests`

**Official Requirement Docs** : [Port power policies end-to-end tests](https://docs.google.com/document/d/1CS1gCt8DmtGX39mFuTs2m306mK_vTT6T_VUHjg3Wo4w/edit?usp=sharing&resourcekey=0-_7vfr0BYbkzGOGvh8ToYmQ)

**Project Page** : [summerofcode](https://summerofcode.withgoogle.com/projects/#6687401885302784)

**Summary** : Integration tests of power policies in `ChromeOS` utilize `autotest` for automated testing. There are several problems with autotest.

- The power policy tests are not stable and fail randomly.
- It's super tedious to find the exact error from its vast log trails.
- This policy testing requires a particular debugging device, `servo` connected to the `DUT` (Device Under Test), a Chromebook, to perform various hardware-level debugging stuff, including operating DUT charging state, power state etc. There is no documentation in autotest on the requisite setup to replicate the intended behaviour for these policy tests.
- Also, the autotest framework is old and still uses `Python2` that has already reached the end of life officially in January 2020.

Instead of making a complete overhaul by migrating the whole autotest to `Python3`, the ChromeOS team has decided to port the policy tests in a fast golang based automated testing framework [Tast](https://chromium.googlesource.com/chromiumos/platform/tast) gradually and decommission the autotest eventually.  
As a Chromium GSoC intern, my work is to migrate five power policy tests by understanding one of the possible test setups, port and document them and make them stable. The policies are `PowerPeakShift`, `BatteryChargeMode`, `USBPowershare`, `BootOnAC` and `AdvancedBatteryChargeMode`.

**Mentors** 👨

- Oleh Lamzin
- Mahmoud Gawad

**Technology Used** : `golang`, `Tast`, `python`, `autotest`, `gRPC`

**My Gerrit Profile URL** : [Chromium Gerrit](https://chromium-review.googlesource.com/q/owner:bisakhmondal00%2540gmail.com)

## Pre-GSoC Work

As my project requires access to `Chromebooks` & `servos`, so the starter work that had been asked was a bit different for the initial evaluation. I was responsible for creating a `Gerrit` (Google Git) scraper in `golang` and content parser-analyser to index the commits and reviews made by the Chromium authors using the chrome `CDP` (chrome devtools protocol). Without generically fetching the pages through `HTTP.GET`, in a nutshell, the program loads the page in a headless Chrome instance and communicates with it directly with devtools protocol (CDP). `Tast` heavily relies on CDP WebSocket connection for the required communication. Finally, it provides a CLI to fetch and parse commit messages, per author commit-review counts and a dockerfile with the build steps to run everything in an isolated containerized environment.

**Full Problem Details** : [GSoC - Port power policies end-to-end tests: starter bug](https://docs.google.com/document/d/1dxkJFTJ4RCUfmzl8AbNLtJPtH39Y8QOnnE4SVlRhwAY/edit?usp=sharing)

**Solution Implemented** : [bisakhmondal00/cdp-go](https://github.com/bisakhmondal/cdp-go)

## Community Bonding Progress

I had spent the whole three weeks setting up the ChromeOS development setup, creating a new chroot, building images, exploring the `Tast` framework, completing tast [codelabs](https://chromium.googlesource.com/chromiumos/platform/tast/+/HEAD/docs/README.md#first-steps) and also understanding the Chromium git-flow on a single monolithic repository. My mentor, Oleh, shared some bugs and I picked one ([chromium:1142132](https://bugs.chromium.org/p/chromium/issues/detail?id=1142132)) and started working on the fix to have a hands-on with the Chromium Gerrit. Along the way, I also worked on implementing a `TODO` feature on `tast-lint`.

✅️ [CL:2919074](https://chromium-review.googlesource.com/c/chromiumos/platform/tast/+/2919074) : Better Handling of `fmt.Errorf` in `tast-lint`( Improvement over existing implementation by pruning the entire sub-parse-tree for `ast` selector expression with an ability to deal with complex scenarios. )

✅️ [CL:2919426](https://chromium-review.googlesource.com/c/chromiumos/platform/tast/+/2919426) : **Allow Linting from `git` Subdirectories** where previously `tast-lint` could only be run from repository git root.

## Change Logs

Here is the list of CLs with brief info that I worked on during the coding period from June 7 to August 16.

✅️ [CL:3056633](https://chromium-review.googlesource.com/c/chromiumos/platform/tast-tests/+/3056633) : **`servo` package migration from remote to common**. Remote tests rely on gRPC to communicate with DUT for policy testing but certainly, some cases only require a servo connection in local tests, e.g. test that doesn't require a restart. In my case, with this change, _PeakShift_, _BatterCharge_ & _AdvancedBatterCharge_ has been made possible to write as local tests.

✅️ [CL:2987941](https://chromium-review.googlesource.com/c/chromiumos/platform/tast-tests/+/2987941) : **Battery Charge - Drain Utilities**. Implements the required functionality to ensure the DUT is within the required batter range to satisfy the requirements of certain power policies.

✅️ [CL:3058137](https://chromium-review.googlesource.com/c/chromiumos/platform/tast-tests/+/3058137) : **Peak Shift Local Policy Tests**. It tests the behaviour of the `DevicePowerPeakShiftEnabled` power management policy that if enabled, reduces AC usage in peak hours.

✅️ [CL:3064937](https://chromium-review.googlesource.com/c/chromiumos/platform/tast-tests/+/3064937) : **Battery Charge Mode Local Policy Test**. It tests the behaviour of the `DeviceBatteryChargeMode` power management policy that if enabled, minimizes battery stress and wear-out by using _standard charge/ fast charge/ adaptive charge_ depending upon the policy enrollment value.

✅️ [CL:3071878](https://chromium-review.googlesource.com/c/chromiumos/platform/tast-tests/+/3071878) : **Advanced Battery Charge Mode Local Policy Test**. It tests the behaviour of the `DeviceAdvanced BatteryChargeModeEnabled` power management policy that if enabled, maximizes the battery health by using a standard charging algorithm and other techniques during non-working hours.

✅️ [CL:2973093](https://chromium-review.googlesource.com/c/chromiumos/platform/tast-tests/+/2973093) : **Boot on AC Remote Policy Test**. It tests the behaviour of the `DeviceBootOnAcEnabled` policy that if enabled, reboots the DUT from a power-off state when connected to an AC power supply.

✅️ [CL:3075461](https://chromium-review.googlesource.com/c/chromiumos/platform/tast-tests/+/3075461) : **USB Power Share Remote Policy Test**. It tests the behaviour of the `DeviceUsbPowerShareEnabled` policy that if enabled, shares power through `USB VBUS` in a power-off state.

🚧 [CL:3090465](https://chromium-review.googlesource.com/c/chromiumos/platform/tast-tests/+/3090465) : **Wilco Device Setup Documentation**. It provides elaborate details of the test lab setup and summarizes important facts of servo devices required for writing and debugging power management policies. Effectively, it fills the gap as a setup guide that we lack in autotest.

---

The planned GSoC milestones have been achieved with the aforelisted CLs. Out of curiosity, I spent some time implementing `DeviceRebootOnShutdown` policy tests and migrating all of the Wilco DTC remote tests (that involves interacting with `DTC VM` and `Supportd`) to local in tast with the required changes. Further, I would like to continue contributing more to tast on this migration process from autotest.

🚧 [CL:3080647](https://chromium-review.googlesource.com/c/chromiumos/platform/tast-tests/+/3080647) : **Reboot on Shutdown Remote Policy Tests**. If enabled, the policy replaces all shutdown buttons in the UI with restart buttons.

✅️ [CL:3088795](https://chromium-review.googlesource.com/c/chromiumos/platform/tast-tests/+/3088795) : **Wilco DTC Enrolled Fixtures**. Provides a fixture with `Wilco DTC VM` & `Supportd` daemon running returns chrome & fakedms object for policy enrollment.

✅️ [CL:3085093](https://chromium-review.googlesource.com/c/chromiumos/platform/tast-tests/+/3085093) : Remote test migration to local of Wilco DTC `GetStatefulPartitionAvailableCapacity` gRPC method.

✅️ [CL:3089877](https://chromium-review.googlesource.com/c/chromiumos/platform/tast-tests/+/3089877) : Remote test migration to local of Wilco DTC `PerformWebRequest` gRPC method.

✅️ [CL:3097629](https://chromium-review.googlesource.com/c/chromiumos/platform/tast-tests/+/3097629) : Remote test migration to local of Wilco DTC `RunRoutineRequest` and `GetRoutineUpdate` gRPC methods.

✅️ [CL:3113185](https://chromium-review.googlesource.com/c/chromiumos/platform/tast-tests/+/3113185) &nbsp;: Remote test migration to local of Wilco DTC `SendMessageToUi` gRPC methods new `wilcoDTCEnrolledExtensionSupport` fixture to perform enrollment along with wilco test extension support.

✅️ [CL:3113192](https://chromium-review.googlesource.com/c/chromiumos/platform/tast-tests/+/3113192) &nbsp;: Remote test migration to local of Wilco DTC `HandleMessageFromUi` gRPC method.

[ **Notion Used** : ✅️ &rarr; CL has been Committed to `cros/main` Head, 🚧 &rarr; CL In-Review ]

## Project Challenges

!! 🚨 _This section contains intricate details about the blockers and might be boring. Feel free to skip/skim the details._ 🚨 !!

I had faced several challenges, uncertainties throughout the GSoC and this section explain some of them briefly. My project was on power policy integration tests so a test setup is a must to inspect the intended behaviour. And the policies being the power policy in nature, are dependent on battery charge drain utilities, so a physical device is mandatory as we can't run those tests in a `VM` (virtual machine) due to the battery being one of the hardware dependencies.  
My test setup comprised of two Chromebooks where one acts as `Servo Host` and the other is the actual `DUT`, two V4 Servos (USB Type-A and USB Type-C) and a Micro Servo that is attached to the DUT logic board debug header to perform various hardware-level operations.  
Regarding **my development environment**, I always used a `GCP`, Google Cloud Platform, instance (2 cores, 8 GB RAM, 300 GB SSD) for all the development. Initially, I tried everything locally, but due to network, hardware bottlenecks, thermal throttling, lack of enough storage space, it turned out to be a disaster during the ChromeOS dev setup. 🤡

- **Remote Setup** : Due to the nationwide lockdown in June, the ChromeOS team couldn't drop the physical devices at my doorstep. My mentor, Oleh, gave me access to the required setup that I have mentioned earlier. All I had to do is borrow another GCP instance that acts as a proxy where the resources were available through `Reverse SSH Tunnelling`. I could use them from my dev instance through another `Local SSH Tunnelling` (an extra network hop).

  - I allocated an instance at `asia-south1-c` (Mumbai, India), closest to my location for the proxy instance, thinking that the network latency would be lesser however it turns out it's quite the opposite later. I was getting painful `context.DeadlineExceeded` error repeatedly 😢 for all the code instruction that waits for DUT to reconnect after a reboot. The existing stable tests were also failing for the same issue.
  - I, along with my mentor, knew something is wrong. We performed various debugging, changes in the way we are establishing `SSH`. Meanwhile, I was writing **"hacky"**, non-publication ready code to continue testing with the issue to minimize time wastage. It took _a month_ to figure out that the concern is with the GCP instance itself 😶. Later, I switched my proxy server location from Mumbai to `europe-west3-c` (Frankfurt, Germany), nearest to my mentor's location to mitigate the very same issue.
  - Lately, I was facing massive connection drops. The resources were inaccessible through the middle proxy. Instead of exposing DUT and Servo Host to the proxy server, Oleh took some time and exposed his `raspberry-pi` into the proxy server. Due to this, I was able to do remote port forwarding directly into my dev setup, exposing all the required resources and avoiding an extra connection hop.

- **Servo** : Three servos offer a different set of commands to communicate through `servod` via `xmlrpc`.

  - There was no documentation on available commands exposed by the `micro` servo and servo `v4`. Also, autotest lacks documentation on device setup and I didn't have physical access to the servos. So it was a bit of a challenge to identify the commands based upon the requirements and figure out the potential combination of servos connected to the DUT. Later, the servo setup that I used looks similar to the figure shown below. The _"Labstation or Workstation"_ is the `Servo Host`.
  <div className="flex flex-wrap justify-center	 -mx-2 overflow-hidden xl:-mx-2 py-8">
    <div className="my-1 px-2 overflow-hidden xl:my-1 xl:px-2 xl:w-1/2">
      <Image alt="gsoc" src="/static/images/v4.png" width={400} height={300 } />
    </div>
    <div className="my-1 px-2 overflow-hidden xl:my-1 xl:px-2 xl:w-1/2 ml-6">
      <Image alt="chromium" src="/static/images/v4p1.png" width={400} height={300} />
    </div>
  </div>

  <h6 className="text-center py-4 text-xs	italic"> Source: [FAFT - Automated Firmware Testing](https://chromium.googlesource.com/chromiumos/third_party/autotest/+/HEAD/docs/faft-how-to-run-doc.md) </h6>

  - Servo `v4p1` (Type-C) has the ability to act as a replacement of DUT charger and fiddle with servo power delivery (PD) through `servo_pd_role` command with two modes `src` for charging and `snk` for discharging. Initially, a bug was encountered where I wasn't able to flip PD role from _snk_ to _src_ neither via `servo package` nor through `dut-control`. It costs us a prolonged, troublesome firmware update on the servos.

- **Dev Setup** : There was an instance where to resolve a merge conflict I performed a `repo sync` and an update on my chroot. It shattered my entire dev setup💥. Suddenly I was not able to apply any policies due to the enrollment fixture was failing with an error :

  ```log
  2021-08-01T14:54:03.360004Z [14:54:03.359] Error at enrolled_fixture.go:110: Failed to enroll using Chrome: rpc error: code = Unknown desc = failed to start chrome: login failed: could not enroll: context deadline exceeded; last error follows: Enterprise Enrollment login screen not found
  2021-08-01T14:54:03.360057Z [14:54:03.359] Stack trace:
  Failed to enroll using Chrome
          at chromiumos/tast/remote/policyutil.(*enrolledFixt).SetUp (enrolled_fixture.go:110)
          at chromiumos/tast/internal/planner.(*statefulFixture).RunSetUp.func1 (fixt.go:396)
          at chromiumos/tast/internal/planner.safeCall.func2 (safe.go:92)
          at runtime.goexit (asm_amd64.s:1374)
  rpc error: code = Unknown desc = failed to start chrome: login failed: could not enroll: context deadline exceeded; last error follows: Enterprise Enrollment login screen not found
  ```

  I rebuilt the new packages and built an image (`R94-14126.0.2`) with the latest changes to flash it into the DUT, `drallion` board. But it didn't work. Later, Oleh flashed a prebuilt stable image `R94-14131.0.0` and solved the issue.

**As it is evident, these blockers cost a significant amount of time during my project. Also, it taught me to explore different solutions to tackle such adversaries. It led me to explore the details of different parts of the tast framework that probably I wouldn't have gone through without these scenarios. Some of the packages are tremendously well written, especially the [testexec](https://source.chromium.org/chromiumos/chromiumos/codesearch/+/main:src/platform/tast-tests/src/chromiumos/tast/common/testexec/testexec.go), [linuxssh](https://source.chromium.org/chromiumos/chromiumos/codesearch/+/main:src/platform/tast/src/chromiumos/tast/internal/linuxssh/;bpv=0), [rpc](https://source.chromium.org/chromiumos/chromiumos/codesearch/+/main:src/platform/tast/src/chromiumos/tast/internal/rpc/;bpv=0), [servo](https://source.chromium.org/chromiumos/chromiumos/codesearch/+/main:src/platform/tast-tests/src/chromiumos/tast/common/servo/) etc.**

## Acknowledgement

I would like to thank my mentors Oleh and Mahmoud for their support and all the help. I am thankful to Oleh for being the person to be reached out anytime for any discussion, guiding me throughout the whole process, igniting cool ideas and for the awesome in-depth code reviews. It never felt like a remote internship. This project itself was an interesting experience for me since it was entirely new. I've never worked before with such kinds of tools interacting with different OS services and hardware. Exploring and understanding two separate codebases was exhilarating. I got to learn how to write efficient, self-contained Go code. Finally, a big thanks to Google Open Source, Google Summer of Code and Chromium for this opportunity.

This GSoC is ending but Chromium has won a new contributor. I hope it's just the beginning of a new story.
