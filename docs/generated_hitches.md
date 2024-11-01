# Generated Hitches / Historical Driver Demand

1. [The Problem](#the-problem)
1. [The Follow-Up](#the-follow-up)
1. [The Data](#the-data)
1. [Building Hitches](#building-hitches)
1. [Subregions](#subregions)
1. [Booking Hitches](#booking-hitches)
1. [Technical Details](#technical-details)

## The Problem

There is a fine balance between Turn Rate and OTP. There were doubts that the predictive model for driver demand was
striking the right balance and was instead erring on the side of OTP performance. This means we may be overtrucked
and failing to capture as much money for our drivers as we'd like to. We wanted to attempt to find the point at which we were undertrucked. 

The first attempt was to control daily driver demand as a single number. This was approached by looking at the
overall idle time of our drivers in each region, identifying "high idle" drivers as individuals with more than two
hours of idle time over the course of their shift. By eliminating a portion of these shifts, we hoped to see our turn rate increase, ideally finding the point at which we start brokering loads.

## The Follow-Up

This has gone over fairly successfully, and prompted a second phase which is to actually build a driver shift schedule
that plans out drivers in a way that better matches the expected distribution of loads on a given day.

## The Data

To determine where/when we want shifts, the system compiles the previous three instances of that day (Monday, Tuesday, etc) to build up the average dipsatchability data of the loads over each day.

We define "dispatchability" as a load that is all of the following:

1. Ready
1. Able to be carried out (based on arrival window)
1. Involved facilities are open

From this, we can build a curve of the average load distribution for the given day of the week.

## Building Hitches

A cron runs once per week to generate the data for the upcoming week. As of writing this, the cron is intended to run on Friday and generate one week worth of data data starting from the following Monday.

From all the estimated shift times, the system will perform a greedy algorithm to build the longest hitches with the following restrictions:

* Hitches can have one day without a shift (e.g. a hitch that spans 5 days, but only has shifts on Monday, Tuesday, Thursday, and Friday)
* Hitches can have varied start times, but will be restricted to a 2-hour window across the hitch (e.g. all shift start times will fall between 8am and 10am)

These rules are intended to maximize the number of 5 day hitches available to the drivers. There will be hitches of various lengths from 1-5 shifts that can be as long as 6 days in the case of a 5-shift hitch with a day off in the middle.

## Subregions

If configured, facilities can be used as subregion hubs to build driver demand around. An example of this is in Atlanta (pool 52), where we have 3 major service centers acting as our subregions in 3 distinct parts of the overall region. Configuring subregions will cause the historical load data to be crunched based on proximity to each subregion before being built into hitches. Using subregions can _dramatically change what drivers see when booking hitches_. See [Driver Hitch Booking](#driver-booking) for details.

## Booking Hitches

The old hitch booking has two notable flaws:

* Drivers booked hitches purely based off of the availability of a shift on the _first day of the hitch_. After that, they chose their own length of hitch, regardless of how much driver demand our system may have expected.
* Drivers' homebases are not taken into consideration when demand is being booked. This means that it is up to the planners and AMs to try to put drivers in the correct places in the region as a whole.

This meant that it was very common for drivers to be put on standby as their hitch progressed, as the loads available in the region did not match the number of drivers that were signed up to drive that day.

The new system tries to build hitches around historical data to allow us to only let drivers book hitches where _all_ shifts in the hitch are expected to be needed. It attempts to take both spatial and temporal data into consideration when building the hitches.

### Driver Booking

The app workflow has been updated to reflect the new hitch process, and drivers are free to book multiple hitches, as long as they match the following criteria:

* Drivers can only see/book hitches starting the next day
* Drivers can only see hitches for their closest subregion, if their region has subregions configured
* Desired hitch does not overlap with their existing hitches
    * Hitches cannot be nested inside of eachother, i.e. a Driver cannot book a 1-day hitch to fill an off-day in an existing hitch
    * Desired hitches shifts do not fall within a 10 hour buffer around the shifts of their existing hitches, assuming 14 hour shifts

### Carrier Booking

Carriers can use the web to book hitches for their drivers. Their ability to book hitches follows the same rules as the drivers in the app. Their UI varies slightly, but provides the same information

### Internal Booking

Internal folks are able to see more than the drivers/carriers. They are not limited to the specific subregion that a driver is closest to, and they are also able to see same-day hitches.

## Technical Details

### Autodispatch pools

Each pool can have this demand model enabled via the `use_shift_demand_generation` bool column on the `vpf.autodispatch_pool` table.

### Cron

The `generate_historic_driver_demand_slots` cron controls when this process runs. As of writing, this runs hourly from 12pm-8pm Mountain Time on Friday.

### Database

The generated hitch data is stored in the following tables

* `vpf.shift_demand_generation` - The base run information
* `vpf.shift_demand_generated_hitch` - The generated hitches, including subregion information and a reference to any booking that may have come from this generated hitch
* `vpf.shift_demand_generated_shift` - The generated shifts, each referencing which hitch they are a part of
* `vpf.shift_demand_subregion` - Subregions for a pool to build hitches around
* `vpf.shift_demand_generation_raw_output` - Intended to house raw data for troubleshooting/investigation. Currently unused

### Endpoints

There are only two endpoints available:

* `/v2/vpf/driver/hitch/booking_v3/available_hitches` - Allows drivers to view hitches that are available to them, or admin/internal users to see what hitches are available for a given driver
* `/v2/vpf/driver/hitch/booking_v3/book` - Allows drivers to book their own hitches, or admin/internal to book hitches on behalf of drivers


### Odds and Ends

We allow swapping hitches to other drivers. This is at least one place in the code that maniupulates hitches in a less-than-ideal manner for keeping our Generated Hitches aligned with the rest of the system. There is a function to handle this specific scenario, but there may be others we haven't uncovered yet. Be careful when trying to move hitches around and deleting/recreating hitches.
