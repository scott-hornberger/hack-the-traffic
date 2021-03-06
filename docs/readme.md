# Austin Bluetooth Travel Sensor Data Documentation

 * [Dataset Documentation](#dataset-documentation)
    * [Individual Address Files (IAF)](#individual-address-files-iaf)
    * [Inidividual Traffic Match Files (ITFM)](#individual-traffic-match-files-itmf)
    * [Traffic Match Summary Records (TMSR)](#traffic-match-summary-records-tmsr)
    * [Travel Sensor Locations](#travel-sensor-locations)
    * [Travel Sensor Route Segments](#travel-sensor-route-segments)
 * [API Documentation](#api-documentation)
    * [Socrata Query Examples](#socrata-api-query-examples)
 * [Route Index](#route-index)
 * [Glossary](#glossary)

## Dataset Documentation

### Individual Address Files (IAF)

Each row in this dataset represents a Bluetooth device that was detected by one of our sensors. Each record contains a detected device’s anonymized Media Access Control (MAC) address along with the time and location the device was detected. These records alone are not traffic data but can be post-processed to measure the movement of detected devices through the roadway network

#### Download

 - [data.austintexas.gov](https://data.austintexas.gov/dataset/Bluetooth-Travel-Sensors-Individual-Addresses/qnpj-zrb9/data)
 - [data.world](https://data.world/ctr/atd-awam-iaf)

#### Column Descriptions
 
*record_id*: The unique record identifer generated as an MD5 hash of the row contents

*host_read_time*: The timestamp on the host server when the device address was received. By default, this is the timestamp used in travel time estimation.

*field_device_read_time*: The timestamp of the field device when the address was received. By default, this is not used in travel time estimation.

*reader_identifier*: The unique identifier of the sensor that the Bluetooth device address record originated from.

*device_address*: The unique address of the device that was read by the sensor. For security, the source MAC address is discarded and replaced with a random address.

### Individual Traffic Match Files (ITMF)

Each row in this dataset represents one Bluetooth enabled device that was detected at two locations in the roadway network. Each record contains a detected device’s anonymized Media Access Control (MAC) address along with information about origin and destination points at which the device was detected, as well the time, date, and distance traveled.

#### Download

 - [data.austintexas.gov]( https://data.austintexas.gov/dataset/Bluetooth-Travel-Sensors-Individual-Traffic-Matche/x44q-icha/data)
 - [data.world](https://data.world/ctr/atd-awam-itmf)

#### Column Descriptions

*record_id*: The unique record identifer generated as an MD5 hash of the row contents

*device_address*: The unique address of the device that was read by the field software. For security, the source MAC address is discarded and replaced with a random address.

*origin_reader_identifier*: The unique identifier assigned to origin sensor that recorded a device address match.

*destination_reader_identifier*: The unique identifier assigned to destination sensor that recorded a device address match.

*start_time*: The time the device address was recorded at the origin sensor.

*end_time*: The time the device address was recorded at the destination sensor.

*day_of_week*: The name of the day of the week at the time the device address was recorded at the origin sensor. 

*travel_time_seconds*: The travel time in seconds from the origin to the destination sensor.

*speed_miles_per_hour*: The speed in miles per hour between the origin and the destination sensors.

*match_validity*: Indicates whether the sensor server classified the traffic data sample as being valid or invalid based on the filtering algorithm and minimum/maximum allowable speeds applied to the roadway segment. Values are valid or invalid.

*filter_identifier*: The numeric code of the filtering algorithm used in the outlier filter for the roadway segment. See the host documentation section titled “Algorithm Configuration” for more information.

### Traffic Match Summary Records (TMSR)

The traffic summary records contain aggregate travel time and speed summaries based on the individual traffic match records. Each row in the dataset summarizes average travel time and speed along a sensor-equipped roadway segment in 15 minute intervals.

#### Download
 - [data.austintexas.gov](https://data.austintexas.gov/dataset/Bluetooth-Travel-Sensors-Match-Summary-Records/v7zg-5jg9)

### Travel Sensor Locations

This dataset contains information about the location of Bluetooth travel sensors in Austin, TX.

#### Download

 - [data.austintexas.gov](https://data.austintexas.gov/dataset/Travel-Sensors/6yd9-yz29)

### Travel Sensor Route Segments

This dataset contains line features which represent travel segments connected by the City of Austin's bluetooth travel sensor network. This dataset can be linked to travel sensor data to visualize travel time and speeds.

See also: [Route Index](#routes)

#### Download

 - [data.austintexas.gov](https://data.austintexas.gov/dataset/Bluetooth-Travel-Sensors-Routes/itsd-rizg)

## API Documentation

- [Data.World API](https://docs.data.world/documentation/api/)
- [Socrata Open Data API (SODA)](https://dev.socrata.com/docs/endpoints.html)

### Socrata API Query Examples
These queries use the [Socrata Open Data API (SODA)](https://dev.socrata.com/consumers/getting-started.html) to fetch data from the City of Austin's [data portal](http://data.austinetxas.gov). We'll use a few of them in the [Map the Traffic workshop](https://github.com/cityofaustin/hack-the-traffic/tree/master/mapping_workshop).

1. Return top 10 most recent rows from weekdays 
    ```sql
    https://data.austintexas.gov/resource/922j-6afw.json?
    $query=SELECT * WHERE UPPER(day_of_week) NOT IN('SATURDAY', 'SUNDAY') ORDER BY start_time DESC LIMIT 10
    ```
    
2. Return the maximum 'valid' speed for each origin reader
    ```sql
    https://data.austintexas.gov/resource/922j-6afw.json?
    $query=SELECT origin_reader_identifier, MAX(speed_miles_per_hour) WHERE match_validity='valid' GROUP BY origin_reader_identifier
    ```

3. Count all unique segments by concatenating orgin and destination reader names
    ```sql
    https://data.austintexas.gov/resource/922j-6afw.json?
    $query=SELECT origin_reader_identifier || "$" || destination_reader_identifier as segment_name, COUNT(segment_name) as count GROUP BY segment_name
    ```

4. Return all rows with start_time at 4:30PM
    ```sql
    https://data.austintexas.gov/resource/922j-6afw.json?
    $query=SELECT origin_reader_identifier, start_time WHERE start_time LIKE('%25T16:30%25')
    ```

5. Compute average speed for all northbound segments where start time is between 4PM and 7PM
    ```sql
    https://data.austintexas.gov/resource/922j-6afw.json?
    $query=SELECT origin_reader_identifier || "$" || destination_reader_identifier as segment_name, start_time, day_of_week, speed_miles_per_hour WHERE match_validity='valid' AND segment_name in ("tx71_silvermine$us290_tx71","us290_tx71$us290_wm_cannon","us290_wm_cannon$lamar_panther","lamar_panther$Lamar_and_Manchca_Barton_skyway","Lamar_and_Manchca_Barton_skyway$Lamar_Blue_Bonnet","Lamar_Blue_Bonnet$lamar_oltorf","lamar_oltorf$lamar_lamar_square","lamar_lamar_square$lamar_barton_springs","lamar_barton_springs$lamar_riverside","lamar_riverside$lamar_5th","lamar_5th$lamar_6th","lamar_6th$lamar_12th","lamar_12th$lamar_mlk","lamar_mlk$lamar_24th","lamar_24th$lamar_29th","lamar_29th$lamar_38th","lamar_38th$lamar_45th","lamar_45th$lamar_51st","lamar_51st$lamar_koenig","lamar_koenig$lamar_airport","lamar_airport$lamar_morrow","lamar_morrow$lamar_payton_gin","lamar_payton_gin$lamar_rundberg","lamar_rundberg$lamar_braker","lamar_braker$lamar_parmer","congress_slaughter$congress_wm_cannon","congress_wm_cannon$congress_stassney","congress_stassney$congress_benwhite","congress_benwhite$congress_oltorf","congress_oltorf$congress_elizabeth","congress_elizabeth$riverside_congress","riverside_congress$congress_cesar_chavez","congress_cesar_chavez$congress_5th","congress_5th$congress_6th","congress_6th$congress_11th","guadalupe_15th$mlk_guadalupe","mlk_guadalupe$guadalupe_21st","guadalupe_21st$guadalupe_24th","guadalupe_24th$guadalupe_26th","guadalupe_26th$guadalupe_27th","south_1st_slaughter$south_1st_wm_cannon","south_1st_wm_cannon$south_1st_stassney","south_1st_stassney$south_1st_st_elmo","south_1st_st_elmo$south_1st_benwhite","south_1st_benwhite$south_1st_oltorf","south_1st_oltorf$south_1st_barton_spring","south_1st_barton_spring$cesar_chavez_lavaca","cesar_chavez_lavaca$lavaca_5th","lavaca_5th$lavaca_6th")
    |> SELECT start_time, day_of_week, speed_miles_per_hour, segment_name where UPPER(day_of_week) NOT IN('SATURDAY', 'SUNDAY')
    |> SELECT start_time, day_of_week, speed_miles_per_hour, segment_name WHERE start_time LIKE('%25T16:%25')  OR start_time LIKE('%25T17:%25') OR start_time LIKE('%25T18:%25') 
    |> SELECT segment_name, AVG(speed_miles_per_hour), COUNT(*) GROUP BY segment_name
    ```

6. Compute percentage of valid matches for all sensors with matches
    ```sql
    https://data.austintexas.gov/resource/922j-6afw.json?
    $query=SELECT origin_reader_identifier, CASE(match_validity='valid', 1, true, 0) AS valid
    |> SELECT origin_reader_identifier, COUNT(origin_reader_identifier) AS total, SUM(valid) AS total_valid GROUP BY origin_reader_identifier
    |> SELECT origin_reader_identifier, total_valid/total AS pct_valid ORDER BY pct_valid ASC
    ```

## Route Index

You can plug these route objects into queries to find traffic matches along specific routes. The `segment_name` field returned in the [example queries](#query-examples) is a concatenation of the origin and destination reader IDs, and can be used to join data to the `segment_name` field in the [route geoJSON](http://github.com/cityofaustin/hack-the-traffic/mapping_workshop/data/austin_bt_routes.csv) file.

#### Guadalupe Northbound
```
("guadalupe_15th$mlk_guadalupe","mlk_guadalupe$guadalupe_21st","guadalupe_21st$guadalupe_24th","guadalupe_24th$guadalupe_26th","guadalupe_26th$guadalupe_27th")
```

#### Guadalupe Southbound
```
("guadalupe_27th$guadalupe_26th","guadalupe_26th$guadalupe_24th","guadalupe_24th$guadalupe_21st","guadalupe_21st$mlk_guadalupe","mlk_guadalupe$guadalupe_15th")
```

#### Congress Northbound
```
("congress_slaughter$congress_wm_cannon","congress_wm_cannon$congress_stassney","congress_stassney$congress_benwhite","congress_benwhite$congress_oltorf","congress_oltorf$congress_elizabeth","congress_elizabeth$riverside_congress","riverside_congress$congress_cesar_chavez","congress_cesar_chavez$congress_5th","congress_5th$congress_6th","congress_6th$congress_11th")
```

#### Congress Southbound
```
("congress_11th$congress_6th","congress_6th$congress_5th","congress_5th$congress_cesar_chavez","congress_cesar_chavez$riverside_congress","riverside_congress$congress_elizabeth","congress_elizabeth$congress_oltorf","congress_oltorf$congress_benwhite","congress_benwhite$congress_stassney","congress_stassney$congress_wm_cannon","congress_wm_cannon$congress_slaughter")
```

#### Lamar Northbound
```
("tx71_silvermine$us290_tx71","us290_tx71$us290_wm_cannon","us290_wm_cannon$lamar_panther","lamar_panther$Lamar_and_Manchca_Barton_skyway","Lamar_and_Manchca_Barton_skyway$Lamar_Blue_Bonnet","Lamar_Blue_Bonnet$lamar_oltorf","lamar_oltorf$lamar_lamar_square","lamar_lamar_square$lamar_barton_springs","lamar_barton_springs$lamar_riverside","lamar_riverside$lamar_5th","lamar_5th$lamar_6th","lamar_6th$lamar_12th","lamar_12th$lamar_mlk","lamar_mlk$lamar_24th","lamar_24th$lamar_29th","lamar_29th$lamar_38th","lamar_38th$lamar_45th","lamar_45th$lamar_51st","lamar_51st$lamar_koenig","lamar_koenig$lamar_airport","lamar_airport$lamar_morrow","lamar_morrow$lamar_payton_gin","lamar_payton_gin$lamar_rundberg","lamar_rundberg$lamar_braker","lamar_braker$lamar_parmer")
```

#### Lamar Southbound
```
("lamar_parmer$lamar_braker","lamar_braker$lamar_rundberg","lamar_rundberg$lamar_payton_gin","lamar_payton_gin$lamar_morrow","lamar_morrow$lamar_airport","lamar_airport$lamar_koenig","lamar_koenig$lamar_51st","lamar_51st$lamar_45th","lamar_45th$lamar_38th","lamar_38th$lamar_29th","lamar_29th$lamar_24th","lamar_24th$lamar_mlk","lamar_mlk$lamar_12th","lamar_12th$lamar_6th","lamar_6th$lamar_5th","lamar_5th$lamar_riverside","lamar_riverside$lamar_barton_springs","lamar_barton_springs$lamar_lamar_square","lamar_lamar_square$lamar_oltorf","lamar_oltorf$Lamar_Blue_Bonnet","Lamar_Blue_Bonnet$Lamar_and_Manchca_Barton_skyway","Lamar_and_Manchca_Barton_skyway$lamar_panther","lamar_panther$us290_wm_cannon","us290_wm_cannon$us290_tx71","us290_tx71$tx71_silvermine")
```

#### South 1st Northbound
```
("south_1st_slaughter$south_1st_wm_cannon","south_1st_wm_cannon$south_1st_stassney","south_1st_stassney$south_1st_st_elmo","south_1st_st_elmo$south_1st_benwhite","south_1st_benwhite$south_1st_oltorf","south_1st_oltorf$south_1st_barton_spring","south_1st_barton_spring$cesar_chavez_lavaca","cesar_chavez_lavaca$lavaca_5th","lavaca_5th$lavaca_6th")
```

#### South 1st Southbound
```
("lavaca_6th$lavaca_5th","lavaca_5th$cesar_chavez_lavaca","cesar_chavez_lavaca$south_1st_barton_spring","south_1st_barton_spring$south_1st_oltorf","south_1st_oltorf$south_1st_benwhite","south_1st_benwhite$south_1st_st_elmo","south_1st_st_elmo$south_1st_stassney","south_1st_stassney$south_1st_wm_cannon","south_1st_wm_cannon$south_1st_slaughter")
```

#### All Northbound Routes
```
("tx71_silvermine$us290_tx71","us290_tx71$us290_wm_cannon","us290_wm_cannon$lamar_panther","lamar_panther$Lamar_and_Manchca_Barton_skyway","Lamar_and_Manchca_Barton_skyway$Lamar_Blue_Bonnet","Lamar_Blue_Bonnet$lamar_oltorf","lamar_oltorf$lamar_lamar_square","lamar_lamar_square$lamar_barton_springs","lamar_barton_springs$lamar_riverside","lamar_riverside$lamar_5th","lamar_5th$lamar_6th","lamar_6th$lamar_12th","lamar_12th$lamar_mlk","lamar_mlk$lamar_24th","lamar_24th$lamar_29th","lamar_29th$lamar_38th","lamar_38th$lamar_45th","lamar_45th$lamar_51st","lamar_51st$lamar_koenig","lamar_koenig$lamar_airport","lamar_airport$lamar_morrow","lamar_morrow$lamar_payton_gin","lamar_payton_gin$lamar_rundberg","lamar_rundberg$lamar_braker","lamar_braker$lamar_parmer","congress_slaughter$congress_wm_cannon","congress_wm_cannon$congress_stassney","congress_stassney$congress_benwhite","congress_benwhite$congress_oltorf","congress_oltorf$congress_elizabeth","congress_elizabeth$riverside_congress","riverside_congress$congress_cesar_chavez","congress_cesar_chavez$congress_5th","congress_5th$congress_6th","congress_6th$congress_11th","guadalupe_15th$mlk_guadalupe","mlk_guadalupe$guadalupe_21st","guadalupe_21st$guadalupe_24th","guadalupe_24th$guadalupe_26th","guadalupe_26th$guadalupe_27th","south_1st_slaughter$south_1st_wm_cannon","south_1st_wm_cannon$south_1st_stassney","south_1st_stassney$south_1st_st_elmo","south_1st_st_elmo$south_1st_benwhite","south_1st_benwhite$south_1st_oltorf","south_1st_oltorf$south_1st_barton_spring","south_1st_barton_spring$cesar_chavez_lavaca","cesar_chavez_lavaca$lavaca_5th","lavaca_5th$lavaca_6th")
```

#### All Southbound Routes
```
("guadalupe_27th$guadalupe_26th","guadalupe_26th$guadalupe_24th","guadalupe_24th$guadalupe_21st","guadalupe_21st$mlk_guadalupe","mlk_guadalupe$guadalupe_15th","congress_11th$congress_6th","congress_6th$congress_5th","congress_5th$congress_cesar_chavez","congress_cesar_chavez$riverside_congress","riverside_congress$congress_elizabeth","congress_elizabeth$congress_oltorf","congress_oltorf$congress_benwhite","congress_benwhite$congress_stassney","congress_stassney$congress_wm_cannon","congress_wm_cannon$congress_slaughter","lamar_parmer$lamar_braker","lamar_braker$lamar_rundberg","lamar_rundberg$lamar_payton_gin","lamar_payton_gin$lamar_morrow","lamar_morrow$lamar_airport","lamar_airport$lamar_koenig","lamar_koenig$lamar_51st","lamar_51st$lamar_45th","lamar_45th$lamar_38th","lamar_38th$lamar_29th","lamar_29th$lamar_24th","lamar_24th$lamar_mlk","lamar_mlk$lamar_12th","lamar_12th$lamar_6th","lamar_6th$lamar_5th","lamar_5th$lamar_riverside","lamar_riverside$lamar_barton_springs","lamar_barton_springs$lamar_lamar_square","lamar_lamar_square$lamar_oltorf","lamar_oltorf$Lamar_Blue_Bonnet","Lamar_Blue_Bonnet$Lamar_and_Manchca_Barton_skyway","Lamar_and_Manchca_Barton_skyway$lamar_panther","lamar_panther$us290_wm_cannon","us290_wm_cannon$us290_tx71","us290_tx71$tx71_silvermine","lavaca_6th$lavaca_5th","lavaca_5th$cesar_chavez_lavaca","cesar_chavez_lavaca$south_1st_barton_spring","south_1st_barton_spring$south_1st_oltorf","south_1st_oltorf$south_1st_benwhite","south_1st_benwhite$south_1st_st_elmo","south_1st_st_elmo$south_1st_stassney","south_1st_stassney$south_1st_wm_cannon","south_1st_wm_cannon$south_1st_slaughter")
```


## Glossary

*device*: A Bluetooth-enabled device detected by a sensor.

*host*: A centralized server that receives data from the sensor network and generates the output data files.

*reader*: AKA *sensor*, *travel sensor*, *field device* The hardware that detects Bluetooth-enabled devices and transmits the device address and timestamp to a central server. 

*sensor*: AKA *reader*, *travel sensor*, *field device*. The hardware that detects Bluetooth-enabled devices and transmits the device address and timestamp to a central server. 
