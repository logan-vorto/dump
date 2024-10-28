# TONU

TONU or "Truck Ordered, Not Used" are paid out when a load is unable to be completed after a driver/truck has done some amount of work towards completing that load.

## Causes

The main way a TONU will arise is directly from the Driver interacting with the app.

If the driver is unable to pickup the trailer for a load, a workflow in the app will guide them
through submitting a TONU request. This is typically due to one of the following reasons:

1. The required trailer cannot be found 
1. The required trailer is inaccessible, i.e. still at the dock
1. The facility where the trailer is located is inaccessible

A TONU may also arise if the system is unable to create an Empty Asset Pickup for a load.

## Visibility

The load that incurred the TONU is unassigned from the driver. A clone of the load is created, assigned to the driver, and marked as completed in our system.
This clone load is identifiable by a `-TONU` suffix on the load's reference number. If multiple TONUs occur for the same load, the associated clone loads will reflect that in the reference number (e.g. `-TONU2`, `-TONU3`, etc)
By cloning the load, this provides the visibility so that drivers can see their TONUs easily and are not reliant on payout requests / support interactions. This allows more visibility into the calculation and the charges associated with a TONU.

The cloned load will not have linehaul associated with it as it is only intended to act as a record of the TONU occuring, thereby only having charges associated with the TONU payment.

## Payment

### The Old Way

These used to be paid out as a flat $150 charge

### The New Way

These are now paid out as potentially multiple charges:

1. A `Truck Ordered Not Used (TONU)` charge which will be in the amount of the deadhead associated with the work done before the TONU occurred
1. A Fuel Surcharge on the load, if applicable

The two of these together cannot exceed $150. On the load, the Fuel Surcharge will be prioritized over the TONU payment. For example, a load that accrued deadhead
of $100 and a fuel surcharge of $100, the charges on the load will show:

1. TONU: $50
1. FSC: $100

## Technical Details

### Database

TONU charges will show up in the `lohiloop_load_broker_charge` table as any other load charges would.

### Feature Flag

This is currently controlled within Launch Darkly via the `tonu_deadhead_pricing_enabled_bool` flag.

### Gaps

We are not populating the `calcData` field on the TONU charges themselves. Ideally, we would have this populated to aid in troubleshooting and understanding of the charge calculation.