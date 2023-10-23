# Captain's coffee (1 point)

Ahoy, deck cadet,

your task is to prepare good coffee for captain. As a cadet, you are prohibited from going to the captain's cabin, so
you will have to solve the task remotely. Good news is that the coffee maker in captain's cabin is online and can be
managed via API.

May you have fair winds and following seas!

Coffee maker API is available at http://coffee-maker.cns-jv.tcc.

## Hints

* Use VPN to get access to the coffee maker.
* API description si available at http://coffee-maker.cns-jv.tcc/docs.

## Solution

We see that the coffee maker provides one API method to list available drinks and the other one to make coffee (which
requires drink ID).

Let's take a look at what's on the menu. (_Note: `jq` is used for pretty-printing JSON and is not really required_ )

```console
$ curl -s http://coffee-maker.cns-jv.tcc/coffeeMenu | jq
{
  "Menu": [
    {
      "drink_name": "Espresso",
      "drink_id": 456597044
    },
    {
      "drink_name": "Lungo",
      "drink_id": 354005463
    },
    {
      "drink_name": "Capuccino",
      "drink_id": 234357596
    },
    {
      "drink_name": "Naval Espresso with rum",
      "drink_id": 501176144
    }
  ]
}
```

We can prepare any coffee but since proper sailors drink rum, let's try `Naval Espresso with rum` (drink
ID: `501176144`).

```console
$ curl -s -H "Content-Type: application/json" -d '{"drink_id":501176144}'  http://coffee-maker.cns-jv.tcc/makeCoffee/
| jq
{
  "message": "Your Naval Espresso with rum is ready for pickup",
  "validation_code": "Use this validation code FLAG{ccLH-dsaz-4kFA-P7GC}"
}
```

The desired flag is included in the response (in the `validation_code` field).
