# My donut factory
This hypothetical use case has two entities a kiosk aka consumer and a producer aka inventory management system. Unfortunately they do not follow same JSON schema so are unable to communicate directly with one another. MuleSoft integration layer helps bridge this gap. Inventory management system is uses IoT to gather inventory in near real time but still takes seconds to minutes to process requests for inventory. Hence, producer only responds asynchronously when a request for inventory arrives.

## Request Inventory

Consumer is requesting today's inventory of donuts. Kiosk needs to update itself with latest from inventory.

### Activity Diagram:

```plantuml
:Consumer;
:New request for inventory in consumer's format;
:Transform request by adding identity of consumer;
:Sends transformed request to producer;
:Start a job to process inventory request;
:Producer acknowledges request;
```

### Sequence Diagram:

```plantuml
actor Consumer as c
participant "Integration Layer" as il
participant "Producer" as p

c -> il: POST (consumer format)
group MuleSoft
    il->il:Transformation logic
end group
il->p: POST (producer format)
p -> il: 202 Accepted
il -> c: 202 Accepted
```


### Consumer's JSON

Consumer creates a request for donut inventory.

```json
{
	"type": "donut",
}
```

### Producer's JSON

Producer donut inventory request JSON.

```json
{
	"type": "donut",
    "consumer-id": 1011,
}
```

## Inventory Response

Once producer's job successfully finished; it responds asynchronously. Producer creates a json with all available donuts and toppings. It also includes today's pricing in the response. Producer sends a JSON to integration layer in its own format. Integration layer performs transformation from producer's JSON format  to consumer's format and sends the response to consumer.

### Activity Diagram:

```plantuml
:Producer;
:Ready with inventory details;
:Sends response to integration layer; 
:Transform request into consumer's format;
:Sends transformed request to consumer;
:Consumer acknowledges response;
```

### Sequence Diagram:

```plantuml
actor Producer as p
participant "Integration Layer" as il
participant "Consumer" as c

p -> il: POST (producer format)
group MuleSoft
    il->il:Transformation logic
end group
il->c: POST (consumer format)
c -> il: 202 Accepted
il -> p: 202 Accepted
```

### Producer's JSON

Producer produces inventory data for all donuts currently available in donut factory. As a wholesale dealer, factory will sells its donuts by weight. If price is different for any donuts JSONstructure allows to overrides it.

```json
{
	"type": "donut",
    "weight-unit": "lb", 
    "price-unit": "$/lb",
    "price": 10.75,
	"batters":
		{
			"batter":
				[
					{ "id": "10011", "type": "Regular","weight": 500},
					{ "id": "10021", "type": "Chocolate","weight": 200, "price": 11.75 },
					{ "id": "10031", "type": "Blueberry", "weight": 250, "price": 11.75  },
					{ "id": "10041", "type": "Devil's Food", "weight": 150}
				]
		},
	"topping":
		[
			{ "id": "50011", "type": "None", "price": 0 },
			{ "id": "50021", "type": "Glazed", "price": 45.23},
			{ "id": "50051", "type": "Sugar", "price": 34.1},
			{ "id": "50071", "type": "Powdered Sugar", "price": 21.11},
			{ "id": "50061", "type": "Chocolate with Sprinkles", "price": 34.43 },
			{ "id": "50031", "type": "Chocolate", "price": 87.40},
			{ "id": "50041", "type": "Maple", "price": 64.11}
		]
}
```

### Consumer's JSON

Consumer only supports three types of donut's. Original, Chocolate and Blueberry. Kiosk is manufactured in UK so it only supports metric system. Note, example below does not actually follow correct unit version. These are fake numbers for demonstration only.


```json
{
	"type": "donut",
	"ChocolateFlavoredGlazedDonut" : {
		"weight": 0.70,
		"unit": "kg",
		"price": 19.25,
		"unit": "$/kg",
	},
	"ChocolateFlavoredSprinklesDonut" : {
		"weight": 0.73,
		"unit": "kg",
		"price": 19.25,
		"unit": "$/kg",
	},
	"BlueberryFlavoredSugarDonut" : {
		"weight": 0.70,
		"unit": "kg",
		"price": 19.25,
		"unit": "$/kg",
	},
	"OriginalGlazedDonut" : {
		"weight": 0.70,
		"unit": "kg",
		"price": 19.25,
		"unit": "$/kg",
	},
    "OriginalMapleDonut" : {

		"weight": 0.70,
		"unit": "kg",
		"price": 19.25,
		"unit": "$/kg",
	},
    "OriginalSugarDonut" : {
		"weight": 0.70,
		"unit": "kg",
		"price": 19.25,
		"unit": "$/kg",
	},
}
```