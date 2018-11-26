---
layout: tutorial
title: Step 11. Run the Source Selection Algorithm
subtitle: Order processing with Inventory Management
menu_title: Step 11. Run the Source Selection Algorithm
menu_order: 110
level3_subgroup: msi-tutorial
return_to:
  title: REST Tutorials
  url: rest/tutorials/index.html
redirect_from: /guides/v2.3/rest/tutorials/msi-order-processing/run-ssa.html
functional_areas:
  - Integration
---

One of the most significant parts of Inventory Management is the Source Selection Algorithm (SSA). SSA analyzes and determines the best match for sources and shipping based on the priorities you specified in [Step 4. Link stocks and sources
]({{ page.baseurl }}/rest/tutorials/inventory/assign-source-to-stock.html). The algorithm also provides a list of source items with quantities to deduct per each source item.

For more information about shipping and SSAs, see the Wiki topic [Shipment and Order Management](https://github.com/magento-engcom/msi/wiki/Shipment-and-Order-Management).

## Get the list of algorithms

Currently, Magento supports only the default SSA for priority. Third-party developers and future releases may add support for additional algorithms.

**Endpoint**

`GET http://<host>/rest/us/V1/inventory/source-selection-algorithm-list`

**Scope**

`us` store view

**Headers**

`Content-Type` `application/json`

`Authorization` `Bearer <admin token>`

**Payload**

Not applicable

**Response**

``` json
[
    {
        "code": "priority",
        "title": "Source Priority",
        "description": "Algorithm which provides Source Selections based on predefined priority of Source"
    }
]
```

## Run the SSA

The `POST V1/inventory/source-selection-algorithm-result` endpoint uses the algorithm defined by the `algorithmCode` attribute to calculate the recommended sources and quantities for each item defined in the `items` array.

This tutorial does not consider complications such selling out of products or back ordering. We can ask the SSA to determine the best way to immediately ship all the items ordered (20 items of product `24-WB01` and 50 items of product `24-WB03`). If the `shippable` attribute in the response is `false`, there are not enough salable items to complete a full shipment, but the merchant can still perform a partial shipment.


**Endpoint**

`POST http://<host>/rest/us/V1/inventory/source-selection-algorithm-result`

**Scope**

`us` store view

**Headers**

`Content-Type`: `application/json`

`Authorization`: `Bearer <admin token>`

**Payload**

``` json
{
    "inventoryRequest": {
        "stockId": 2,
        "items": [{
            "sku": "24-WB01",
            "qty": 20
        },
        {
            "sku": "24-WB03",
            "qty": 50
        }]
    },
    "algorithmCode": "priority"
}
```

**Response**

The SSA recommends shipping from the following sources:

Product | Source | Quantity
--- | --- | ---
`24-WB01` | Baltimore | 20
`24-WB03` | Baltimore | 19
`24-WB03` | Reno | 31
{:style="table-layout:auto;"}


``` json
{
    "source_selection_items": [
        {
            "source_code": "baltimore_wh",
            "sku": "24-WB01",
            "qty_to_deduct": 20,
            "qty_available": 35
        },
        {
            "source_code": "austin_wh",
            "sku": "24-WB01",
            "qty_to_deduct": 0,
            "qty_available": 10
        },
        {
            "source_code": "reno_wh",
            "sku": "24-WB01",
            "qty_to_deduct": 0,
            "qty_available": 25
        },
        {
            "source_code": "baltimore_wh",
            "sku": "24-WB03",
            "qty_to_deduct": 19,
            "qty_available": 19
        },
        {
            "source_code": "reno_wh",
            "sku": "24-WB03",
            "qty_to_deduct": 31,
            "qty_available": 42
        }
    ],
    "shippable": true
}
```

## Verify this step {#verify-step}

1. Click **Sales** > **Orders**.
2. Click the **View** link in the Action column for the order.
3. Click **Ship**.
4. Select different sources from the **Sources** menu.