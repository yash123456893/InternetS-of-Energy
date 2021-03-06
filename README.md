# CitizenWatt for PINE64+
  
![PINE64+ board](/images/PINE64Board.jpeg)

Adaptations for PINE64 board used for the DAISEE prototype. The code is a based on the one of [CitizenWatt](https://github.com/CitoyensCapteurs/CitizenWatt-Base) (GPL3).

## Hardware

- PINE64+ Board
- nRF24L01+ transceiver, to communicate with CitizenWatt Sensor
- Arduino Uno, to link the PINE64+ board and the RF24L01+ (the transceiver needs the use of SPI, but there is no software support for SPI on the PINE64+ GPIO for the moment)


## Prerequisites

* Python 3.4, and the following libraries :  
** sqlalchemy  
** cherrypy  
** numpy  
** pycrypto  
** psycopg2 for communication with the PostgreSQL database  

* NFR24 library (for Arduino)



## API documentation

This is a list of all the endpoints available in the API. You should authenticate yourself to use the API. For this purpose, use a POST request with `login` and `password` fields.

All the results are in a JSON dict, under the key `data`, for security purpose.

* `/api/sensors`
	* Returns all the available sensors with their associated measure types. If no sensors are found, returns `null`.

* `/api/sensors/<id:int>`
    * Returns all the infos for the specified sensor. If no matching sensor is found, returns `null`.

* `/api/types`
	* Returns all the available measure types. If no types are found, returns `null`.

* `/api/time`
    * Returns the current timestamp of the server.

* `/api/<sensor:int>/get/watts/by_id/<nb:int>`
    * Returns measure with id `<id1>` associated to sensor `<sensor>`, in watts.
    * If `<id1>` < 0, counts from the last measure, as in Python lists.
    * If no matching data is found, returns `null`.

* `/api/<sensor:int>/get/<watt_euros:watts|kwatthours|euros>/by_id/<nb1:int>/<nb2:int>`
    * Returns measures between ids `<id1>` and `<id2>` from sensor `<sensor>` in watts or euros.
    * If `<id1>` and `<id2>` are negative, counts from the end of the measures.
    * Depending on `<watt_euros>`:
        * If it is `watts`, returns the list of measures.
        * If it is `kwatthours`, returns the total energy for all the measures (dict).
        * If it is `euros`, returns the cost of all the measures (dict).
    * Returns measure in ASC order of timestamp.
    * Returns `null` if no measures were found.

* `/api/<sensor:int>/get/<[watt_euros:watts|kwatthours|euros>/by_id/<id1:int>/<id2:int>/<step:int>`
    * Returns all the measures of sensor `sensor` between ids `<id1>` and `<id2>`, grouped by `<step>`, as a list of the number of steps element.`<step>` should be positive.
    * Each item is `null` if no matching measures are found.
    * Depending on `<watt_euros>`:
        * If it is `watts`, returns the mean power for each group.
        * If it is `kwatthours`, returns the total energy for each group.
        * If it is `euros`, returns the cost of each group.
    * Returns measure in ASC order of timestamp.
    * Returns `null` if no measures were found.

* `/api/<sensor:int>/get/<watt_euros:watts|kwatthours|euros>/by_time/<time1:float>/<time2:float>`
    * Returns measures between timestamps `<time1>` and `<time2>` from sensor `<sensor>` in watts or euros.
    * Depending on `<watt_euros>`:
        * If it is `watts`, returns the list of measures.
        * If it is `kwatthours`, returns the total energy for all the measures (dict).
        * If it is `euros`, returns the cost of all the measures (dict).
    * Returns measure in ASC order of timestamp.
    * Returns `null` if no matching measures are found.

* `/api/<sensor:int>/get/<watt_euros:watts|kwatthours|euros>/by_time/<time1:float>/<time2:float>/<step:float>`
    * Returns all the measures of sensor `sensor` between timestamps `time1` and `time2`, grouped by step, as a list of the number of steps element.
    * Each item is `null` if no matching measures are found.
    * Depending on `<watt_euros>`:
        * If it is `watts`, returns the mean power for each group.
        * If it is `kwatthours`, returns the total energy for each group.
        * If it is `euros`, returns the cost of each group.
    * Returns measure in ASC order of timestamp.

* `/api/<sensor:int>/delete/by_id/<id1:int>`
    * Deletes measure with id `<id1>` associated to sensor `<sensor>`.
    * If `<id1> < 0`, counts from the last measure, as in Python lists.
    * If no matching data is found, returns `null`. Else, returns the number of deleted measures (1).

* `/api/<sensor:int>/delete/by_id/<id1:int>/<id2:int>`
    * Deletes measures between ids `<id1>` and `<id2>` from sensor `<sensor>`.
    * Returns `null` if no matching measures are found. Else, returns the number of deleted measures.

* `/api/<sensor:int>/delete/by_time/<time1:float>`
    * Deletes measure at timestamp `<time1>` for sensor `<sensor>`.
    * Returns null if no measure is found. Else, returns the number of deleted measures (1).

* `/api/<sensor:int>/delete/by_time/<time1:float>/<time2:float>`
    * Deletes measures between timestamps <time1> and <time2> from sensor <sensor>.
    * Returns null if no matching measures are found. Else, returns the number of deleted measures.

* `/api/<sensor:int>/insert/<value:float>/<timestamp:int>/<night_rate:int>`
    * Insert a measure with:
        * Timestamp `<timestamp>`
        * Value `<value>`
        * Tariff "day" if `<night_rate> == 0`, "night" otherwise.
    * Returns `True` if successful, `False` otherwise.

* `/api/energy_providers`
    * Returns all the available energy providers or `null` if none found.

* `/api/energy_providers/<id:current|int>`
    * Returns the current energy provider (if `id == "current"`), or the energy provider with the specified id.

* `/api/<energy_provider:current|int>/watt_to_euros/<tariff:night|day>/<consumption:float>`
    * Returns the cost in euros associated with a certain consumption, in kWh.
    * One should specify the tariff (night or day, day if no such distinction is to be done) and the id of the energy_provider.
    * Returns `null` if no valid result to return.
