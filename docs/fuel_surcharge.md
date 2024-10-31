# Fuel Surcharge

Fuel Surcharge (often abbreviated as FSC) is charge paid out to offset fuel costs to drivers based on how many miles they drove while running a load.

## Payment

For completed loads, FSC is calculated against the Deadhead and Linehaul miles accrued during the load.

Restrictions:

* Spot loads **do not** pay FSC
* For TONU loads, only Deadhead miles are used to calculate FSC

## Configuration

There are two pieces to control if FSC will be calculated and added to loads:

* Pool ID must be added in Consul to the `Key/Value` -> `vpfAutodispatchService` -> `vpf_fuel_surcharge_pool_ids` list
* The `vpf.customer_fuel_surcharge_info` table holds customer rules for calculating FSC charges on loads

## Calculation

As of writing, only Chep loads pay out an FSC to drivers. Due to this, the current system has been built to meet the needs of how Chep pays these out by using the formula:

```
FSC = Qualified Miles * ((Regional Fuel Price + Customer Fuel Adjustment) / Customer Avg. Truck Fuel Economy)
```

### Regional Fuel Price

The `update_eia_fuel_prices` cron runs once ever 12 hours and calls the the US Energy Information Administration API to update the regional fuel prices in the US.

* The cron pulls regions from the `fuel_price_region` table
* It writes the prices for regions into the `fuel_price_region_history` table
* The Regional Fuel Price for a load is currently calculated based on the user's home base as saved in the `user_address` table
* If the user's home base does not align with one of our fuel regional pricings in the DB, the cron also captures the average fuel price for the entire country as a fallback.
