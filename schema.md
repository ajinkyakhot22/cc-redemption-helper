# Credit Card Redemption Helper — Data Model Schema

## Overview

The data model is split into five JSON files under the `/data` directory. Together they form the knowledge base that powers redemption recommendations.

```
/data
├── cards.json                  # Credit card definitions
├── transfer_partners.json      # Airline and hotel loyalty programs
├── card_transfer_mappings.json # Which cards transfer to which partners, at what ratio
├── airline_award_charts.json   # Miles required per route and cabin class
└── hotel_award_charts.json     # Points required per hotel tier/property
```

---

## File: `cards.json`

Defines each supported credit card.

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique identifier (e.g. `hdfc_infinia`) |
| `bank` | string | Issuing bank name |
| `name` | string | Short card name |
| `full_name` | string | Official full card name |
| `tier` | enum | `super_premium` or `premium` |
| `network` | string | Card network (Visa Infinite, Mastercard, Amex, Diners) |
| `points_currency.name` | string | Name of the points currency (e.g. "Membership Rewards Points") |
| `points_currency.abbreviation` | string | Short form (e.g. "MR", "RP", "EM") |
| `earn_rates[]` | array | One entry per spend category |
| `earn_rates[].category` | string | Category slug (e.g. `default`, `dining`, `smartbuy_portal`) |
| `earn_rates[].points_per_100_inr` | number | Points earned per ₹100 spent |
| `earn_rates[].multiplier` | number | Optional multiplier vs. base rate |
| `earn_rates[].notes` | string | Human-readable explanation |
| `baseline_redemption.value_inr_per_point` | number | Value when redeemed at base rate (e.g. portal vouchers) |
| `baseline_redemption.method` | string | How to redeem at this rate |
| `fees.joining_fee_inr` | number | One-time joining fee |
| `fees.annual_fee_inr` | number | Annual renewal fee |
| `fees.fee_waiver` | string | Fee waiver condition |
| `key_benefits[]` | array | Highlight perks (lounge, golf, forex, etc.) |

### Cards in v1.0

| Card ID | Bank | Tier |
|---------|------|------|
| `hdfc_infinia` | HDFC Bank | Super Premium |
| `hdfc_diners_black` | HDFC Bank | Super Premium |
| `icici_emeralde` | ICICI Bank | Super Premium |
| `amex_platinum` | American Express | Super Premium |
| `amex_gold` | American Express | Premium |
| `hsbc_premier` | HSBC Bank | Premium |
| `axis_magnus` | Axis Bank | Super Premium |

---

## File: `transfer_partners.json`

Defines airline and hotel loyalty programs that cards can transfer points to.

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique identifier (e.g. `sq_krisflyer`) |
| `name` | string | Program name |
| `type` | enum | `airline` or `hotel` |
| `currency.name` | string | Points/miles currency name |
| `currency.abbreviation` | string | Short form |
| **Airline-specific** | | |
| `airline_code` | string | IATA code |
| `alliance` | string | Star Alliance, oneworld, SkyTeam, or None |
| `hub` | string | Primary hub airport(s) |
| `connectivity_from_india.gateways` | array | Indian airports served |
| `connectivity_from_india.key_destinations` | array | Popular destinations reachable |
| `partner_airlines` | array | Key codeshare/partner airlines |
| `avg_value_inr_per_mile` | object | Estimated value by cabin class |
| **Hotel-specific** | | |
| `brands[]` | array | Hotel brands under this program |
| `india_presence` | string | Strength of presence in India |
| `vietnam_presence` | string | Strength of presence in Vietnam |
| `avg_value_inr_per_point` | object | Estimated value per point |
| `program_notes` | string | Key program rules (expiry, special benefits) |

### Partners in v1.0

| Partner ID | Type | Alliance/Group |
|-----------|------|---------------|
| `sq_krisflyer` | Airline | Star Alliance |
| `air_india_flying_returns` | Airline | Star Alliance |
| `etihad_guest` | Airline | Independent |
| `ba_avios` | Airline | oneworld |
| `qatar_privilege_club` | Airline | oneworld |
| `thai_orchid_plus` | Airline | Star Alliance |
| `marriott_bonvoy` | Hotel | Marriott |
| `ihg_rewards` | Hotel | IHG |
| `accor_live_limitless` | Hotel | Accor |
| `hilton_honors` | Hotel | Hilton |

---

## File: `card_transfer_mappings.json`

Links cards to partners with transfer ratios and operational details.

| Field | Type | Description |
|-------|------|-------------|
| `card_id` | string | References `cards.json[].id` |
| `partner_id` | string | References `transfer_partners.json[].id` |
| `transfer_ratio.card_points` | number | Card points given up |
| `transfer_ratio.partner_points` | number | Partner miles/points received |
| `transfer_ratio.description` | string | Human-readable ratio |
| `minimum_transfer_card_points` | number | Minimum points to initiate a transfer |
| `transfer_in_multiples_of` | number | Transfer must be in this increment |
| `processing_time` | string | Estimated transfer time |
| `transfer_fee` | number or null | Fee if any |
| `notes` | string or null | Tips or caveats |

### Key Transfer Ratios Summary

| Card | Partner | Ratio | Effective miles per RP |
|------|---------|-------|------------------------|
| HDFC Infinia / Diners Black | Any airline | 2 RP → 1 mile | 0.5 miles/RP |
| Amex Platinum / Gold | Any airline | 1 MR → 1 mile | 1.0 miles/MR |
| ICICI Emeralde | Any airline | 4 RP → 1 mile | 0.25 miles/RP |
| Axis Magnus | KrisFlyer / AI | 5 EM → 4 miles | 0.8 miles/EM |
| HSBC Premier | KrisFlyer | 5 RP → 2 miles | 0.4 miles/RP |

---

## File: `airline_award_charts.json`

Defines the miles required to redeem flights by route zone and cabin class.

| Field | Type | Description |
|-------|------|-------------|
| `partner_id` | string | References `transfer_partners.json[].id` |
| `program_name` | string | Display name |
| `chart_type` | enum | `saver`, `standard`, `distance_based`, `zone_based` |
| `chart_notes` | string | Program-level notes on pricing model |
| `routes[]` | array | Award rate entries |
| `routes[].origin_region` | string | Origin zone (matches `region_definitions`) |
| `routes[].destination_region` | string | Destination zone |
| `routes[].cabin` | enum | `economy`, `business`, `first` |
| `routes[].miles_one_way` | number | Miles for one-way award |
| `routes[].miles_round_trip` | number | Miles for round-trip award |
| `routes[].example_routes` | array | Concrete route examples |
| `routes[].notes` | string | Tips for this specific route |

### Region Codes

| Region Key | Covers |
|-----------|--------|
| `India` | All major Indian airports |
| `Southeast_Asia` | Vietnam, Thailand, Singapore, Malaysia, etc. |
| `East_Asia` | Japan, Korea, China, Hong Kong, Taiwan |
| `Middle_East` | UAE, Qatar, Saudi, Bahrain, Oman |
| `Europe` | UK, France, Germany, Netherlands, etc. |
| `North_America` | USA and Canada |
| `Australia_NZ` | Australia and New Zealand |

---

## File: `hotel_award_charts.json`

Defines points required per night at hotel partner properties.

| Field | Type | Description |
|-------|------|-------------|
| `partner_id` | string | References `transfer_partners.json[].id` |
| `pricing_model` | enum | `dynamic` or `points_based` |
| `pricing_notes` | string | Explanation of how pricing works |
| `fifth_night_free` | boolean | Whether 5th night is free on award stays |
| `fourth_night_free` | boolean | Whether 4th night is free on award stays |
| `categories[]` | array | Tiers from budget to ultra-luxury |
| `categories[].tier` | string | Tier name |
| `categories[].typical_points_per_night_range` | array | `[min, max]` range |
| `categories[].example_brands` | array | Brands in this tier |
| `categories[].example_properties_vietnam` | array | Specific Vietnam properties |
| `vietnam_spotlight.best_redemptions[]` | array | Curated high-value Vietnam redemptions |
| `vietnam_spotlight.best_redemptions[].property` | string | Property name |
| `vietnam_spotlight.best_redemptions[].location` | string | City/area |
| `vietnam_spotlight.typical_cash_rate_usd_per_night` | number | Cash price benchmark |
| `vietnam_spotlight.typical_points_per_night` | number | Points cost benchmark |

---

## How the Recommendation Engine Will Use This Data

For a query like "HDFC Infinia, 150,000 points, Vietnam trip, 5 days":

1. **Look up card** → `cards.json` → `hdfc_infinia`
2. **Find transfer partners** → `card_transfer_mappings.json` → filter by `card_id = hdfc_infinia`
3. **For each airline partner:**
   - Apply transfer ratio → 150,000 RP ÷ 2 = 75,000 KrisFlyer miles
   - Look up `airline_award_charts.json` for `India → Southeast_Asia` routes
   - Check if 75,000 miles covers a round-trip (e.g. economy RT = 35,000 miles ✓, business RT = 80,000 miles ✗ for one person)
   - Calculate value: miles × avg_value_per_mile vs. cash ticket price
4. **For each hotel partner:**
   - Apply transfer ratio → 150,000 RP ÷ 2 = 75,000 Marriott points (if Marriott)
   - Check `hotel_award_charts.json` for Vietnam properties
   - Estimate nights coverable at standard rates (e.g. 75,000 ÷ 35,000/night = ~2 nights at Sheraton)
5. **Rank all options by value per original card point (INR value / card points spent)**
6. **Return top recommendations** with explanation

---

## Data Quality Notes

- Transfer ratios and award rates are sourced from publicly available program information as of June 2026.
- Hotel award costs are indicative; dynamic programs require live lookup for exact costs.
- Always verify on the issuing bank's portal and loyalty program website before transferring points — rates can change without notice.
- Taxes, fuel surcharges, and carrier-imposed fees on awards are not included in mile counts and must be paid separately.
