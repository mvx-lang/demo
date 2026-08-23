# demo — a MultiValue demo account

A small trading company: clients, an inventory, orders with line items, staff,
and the states they are in. Modelled on the shape of the demo account UniData
ships (the data is our own), and small enough to read every record.

**This repository is the account.** It is not a package and nothing installs
it: you clone it and what you get is a working account.

```sh
mvx-git clone https://github.com/mvx-lang/demo.git
mvx -a demo
:LIST ORDERS PRODUCT_NO PROD_NAME QTY PRICE
```

It is committed in the **open account format**, so the same repository builds
an account on UniData (`udt-git clone`) and UniVerse (`uv-git clone`) as well
as on MVX. That is the point of it: the dictionaries — conversions, formats,
associations, I-types — have to survive the crossing, and here they can be
watched doing it.

## What is in it

| file | keyed by | about |
| --- | --- | --- |
| `CLIENTS` | `C1001…` | who buys; multivalued phone numbers |
| `INVENTORY` | `P100…` | what is sold; multivalued features |
| `ORDERS` | `1001…` | orders, with multivalued line items |
| `STAFF` | `S01…` | who sells; each reports to another staff member |
| `STATES` | `NSW`… | the lookup every other file points at |

## What it demonstrates

**Conversions.** `SINCE` and `ORD_DATE` are internal day numbers shown through
`D4/`; `ORD_TIME` is seconds through `MTH`; money is the classic scaled integer
— `PRICE` holds `24900` and `MD2$` renders `$249.00`. Read a record raw and you
see the stored form; that is the MV bargain, and the dictionary is the half of
it that travels.

**Associations.** An order's `PRODUCT_NO`, `QTY` and `PRICE` are one
association (`LINE_ITEMS`), so they line up as rows under the order:

```
@ID          Product   Description                  Qty      Price
1001         P100      Cordless Drill 18V             4    $249.00
             P107      Extension Lead 20m            10     $89.90
```

**I-types**, four kinds, deliberately:

- a plain lookup — `ORDERS CLIENT_NAME` is `TRANS(CLIENTS,1,NAME,X)`;
- a lookup on a *multivalued* key — `ORDERS PROD_NAME` translates every line
  item at once;
- a **nested** lookup — `ORDERS REGION` reaches the client, whose own `REGION`
  is itself an I-type into `STATES`: two hops, resolved through the target's
  dictionary;
- a **self-referential** one — `STAFF MANAGER_NAME` looks staff up in `STAFF`.

Try them:

```
:LIST CLIENTS NAME COMPANY CITY STATE_NAME REGION SINCE
:LIST ORDERS CLIENT_NAME CITY REGION ORD_DATE ORD_TIME STATUS
:LIST ORDERS CLIENT_NAME STATUS WITH REGION = "East" WITH STATUS = "open"
:LIST STAFF NAME TITLE MANAGER_NAME SALARY BY NAME
```

## The two that do not evaluate on MVX yet

`INVENTORY STOCK_VALUE` (`PRICE * QTY_ON_HAND`), `ORDERS EPRICE` (`QTY *
PRICE`) and `ORDERS ORDER_TOTAL` (`SUM(EPRICE)`) are arithmetic I-types. They
are ordinary on UniData and UniVerse; MVX's evaluator handles `TRANS` and
`DOCTAG` and returns empty for anything else, so those columns come out blank
here today.

They are in the account on purpose. An account that only contains what the
current implementation understands would prove nothing about moving between
systems — these are exactly the items that have to survive the crossing, and
leaving them out would hide the gap rather than close it.

## How it is stored

Committed in the open account format: one blob per record at `<file>/<id>`, one
per dictionary item at `<file>.DICT/<id>`, the portable file class in
`<file>.DICT/%FILE%`, and `.mv-account` at the root. The backend store is never
committed — the records are the source of truth.

So the diff of a price change is the price, and a dictionary is something you
can read on GitHub:

```
$ cat ORDERS.DICT/PRICE
D
6
MD2$
Price
10R
LINE_ITEMS
```
