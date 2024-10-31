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
* Accrued time is not cumulative across the entire load - **Each stop resets tracks time independently**

Recently, CMC Detention rules have been added that allow emails to be sent out when a load hits the detention threshold. They require this before they will pay Detention out for a load.

## Notifications

When a stop is updated and the system finds that the driver has earned Detention for the current load, the driver will receive a SMS message informing them that they are eligible for detention for that stop.

### Notification Technicals

The chain of events that cause notifications to be sent:

1. The stop is updated, generally due to being completed
1. The `stop_changed_for_billing_calc` DB trigger fires when a stop is updated in the DB
1. A listener causes VPF Accessorials to be updated for a load
1. Billing finds that Detention has accrued for a given load
1. If a notification hasn't been sent out for this stop yet:
    1. An SMS is sent to the driver
    1. An entry is created in `vpf.carrier_contract_detention_notification` to track that we told the driver about this stop.

### CMC 

To accommodate some CMC features, a few more pieces have been added to allow for sending emails as part of the Detention process:

* `vpf.detention_notification_configuration` for rules around the notifications themselves
* `vpf.detention_stop_notified` for keeping track of which stops have triggered emails

Along with a few more columns on the `vpf.carrier_contract_detention` table:

* `on_pickup` / `on_dropoff` to allow control over which stops we trigger on, if desired
* `after_appointment_window` to control if the detention timing is based off of the time of arrival, or the appointment window

There are some minor aspects that are custom-built for CMC needs, but it should be fairly easy to go through and make these parts more generic should other customers want similar behavior.