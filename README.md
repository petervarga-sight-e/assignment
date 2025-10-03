# Programming exercise

## Introduction
Modern power grids are becoming increasingly smart: they juggle energy assets, trying to keep the energy supply flowing efficiently. At the core of such systems sits an energy manager, a piece of software that tells energy assets what to do (for example "produce 50 kW" or "stay idle"), constantly checks whether they are behaving correctly and decides how to mitigate losses if necessary. In practice, such systems have an optimized power schedule for each asset, and they are responsible to ensure assets are running according this schedule.

What truly matters is not only sending out the power targets but also adapting intelligently when things go wrong, while keeping the system stable and predictable.

In this exercise, you will create a simplified energy manager that:
- Dispatches power targets to energy assets according to a given schedule.
- Monitors device statuses.
- Adapts gracefully when schedules or devices fail, ensuring there is always a safe, fallback behavior.

The exercise is split into two parts. The first part is about creating a simple foundation of an Energy Manager. The second part is about implementing a basic loss-mitigation dispatch logic into it.

## Before you start

The task has been prepared so that you don't need to use third party services, such as databases or real asset controller interfaces, just Python. Instead, you are given a `DeviceController` and a `ScheduleProvider` class to work with. Please find them in the shared Python module and read their docstrings for additional information. You are not allowed to change the logic in this module.

## Part 1

In this part, you will create an application that does following:
- Initializes at least 3 device controllers and schedules for them (look at the shared module for examples).
- Every 2 seconds, it sets the current `power_target` on each `DeviceController` based on the schedule. It should use 0 as a fallback option if there is an error.
- Every 10 seconds, it reads the status of each device and logs them.
- The application should run indefinitely unless interrupted, in which case it terminates immediately.

Key points:
- Make sure timing is always respected. The application must not lag.
- The control behaviour cannot be impaired by errors.
- The application should not get stuck if things go out of sync.

## Part 2

Sometimes there are physical limitations that an energy manager should take care of. Energy assets may not be able to produce the scheduled power target; just think of a solar panel that suddenly found itself in a storm that was missed by the forecast. However, when controlling multiple devices, an Energy Manager can dispatch the power targets so that the sum of them is still achieved or at least partially achieved. Your task in this part is to implement such a dispatch logic.

Adjust your code from part 1 by doing the following:
- Give each controller a list of potential power outputs. These values determine how much power a device can produce at a certain time block.
- Use the device statuses (which now contain the potential power output) to check if there are assets not outputting the scheduled targets.
- Implement a naive dispatch logic that distributes the difference between power targets and the current outputs across all assets if and only if there is at least one asset not capable of run according to the schedule. There is no need come up with an optimized dispatch strategy, the most trivial algorithm you can think of is enough.
- You should use the most recent status messages for the dispatch logic. A missing status message is the same as if the potential was 0. You can start the application by reading statuses to ensure the dispatch logic can run from the start.
- The frequency of reading statuses and setting power targets is the same as in Part 1. You shouldn't run these operations outside the timed executions. 

Examples:
```python
targets: {'asset_1': 100, 'asset_2': 100, 'asset_3': 300}
potentials: {'asset_1': 200, 'asset_2': 300, 'asset_3': 50}
dispatched: {'asset_1': 200, 'asset_2': 250, 'asset_3': 50}
```

```python
targets: {'asset_1': 100, 'asset_2': 200, 'asset_3': 300}
potentials: {'asset_1': 0, 'asset_2': None, 'asset_3': 500}
dispatched: {'asset_1': 0, 'asset_2': 0, 'asset_3': 500}
