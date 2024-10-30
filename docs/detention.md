# Automated Detention

Detention is paid out to drivers to compensate for time lost at facilities during a load.

## Configuration

The `vpf.carrier_contract_detention` table holds configuration for adding automated detention rules to a carrier contract.

Detention can be configured with the following values:

* `start_minutes` - How many minutes must pass before a driver becomes eligable for Detention to start being calculated
* `period_minutes` - The length of each interval where Detention is accrued
* `period_cents` - How many cents worth of Detention is accrued each period
* `max_cents` - The maximum number of cents a driver can earn in Detention **per stop**
* `carrier_ratio` / `shipper_ratio` - The respective ratio of the accrued sense that the carrier/shipper will see in the charge. As of writing, this is only used for G3 contracts because we charge the shipper for detention, but do not pay the carrier (i.e. `carrier_ratio 0` and `shipper_ratio 1`)

## Automation

Detention is automatically applied to loads where the contract has Detention configured according to the following rules:

* Only applies to `Pickup` and `Dropoff` stops
* Accrued time is not cumulative across the entire load. **Each stop resets tracks time independently**
