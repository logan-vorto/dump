# Bonus Points / Bonus Coins

Bonus Points are intended to act as a reward for drivers helping out the Vorto / 5F platform where the benefit of the action is not directly related to the driver themselves. This is in contrast to Tier Points, which are designed to be a reflection of how well a driver performs their duties while participating on the platform.

## Earning

As of writing, the only thing in our system that rewards drivers with Bonus Points are trailer status updates. Each status update rewards a driver with 4 Bonus Points. To prevent abuse, each trailer can only reward a driver with points once per 2 hour period.

## Redemption

After earning 100 Bonus Points, they are automatically redeemed or 1 Bonus Coin.

Bonus Coins (called Crank Coin in the codebase) can be used to "crank" a load and earn 10% additional revenue on that load, capping at $50.

###  Configuration 

* `vpf.bonus_coin_v1_event` - Information and point values of the bonus point events
* `vpf.bonus_coin_v1_driver` - Tracks drivers' current bonus point balances
* `vpf.bonus_coin_v1_driver_event` - A log of all the bonus point events that have occurred for drivers

###  Events

As of writing, there are only two events that can occur:

* Trailer Updates (called `yard_check` in the DB, somewhat of a misnomer) that award 4 points
* Award Bonus Coin, which deducts 100 Bonus Points from the driver and gives them 1 Bonus Coin

Which events are visible to drivers is configurable. Currently drivers cannot see the "Award Bonus Coin" event, as seeing an event worth -100 points would likely just be confusing.