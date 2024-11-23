### Part 1: Create an eventhouse
1. Browse to the workspace in which you want to create your tutorial resources. You must create all resources in the same workspace.
2. Select + New item.
3. In the Filter by item type search box, enter Eventhouse.
4. Select the Eventhouse item.
5. Enter fryer_eh (or any other name) as the eventhouse name. A KQL database is created simultaneously with the same name.
6. Select Create. When provisioning is complete, the eventhouse System overview page is shown

### Part 2: Transfor events in Eventstream
1. Switch to the Eventstream we created earlier
2. From the menu ribbon, select Edit. The authoring canvas, which is the center section, turns yellow and becomes active for changes.
3.  the eventstream authoring canvas, select the down arrow on the Transform events or add destination tile, and then select Manage fields. The tile is renamed to ManageFields.
4.  In the Manage fields pane, select only the Value fields from the Temperature, EnergyUse and FillWeight structs. Also select the SourceTimeStamp from the Temperature struct and click save. This way we flatten the data to simplify the table schema. 

### Part 3: Create a destination
1. Hover over the right edge of the Transoform tile and select the green plus icon
2. Select Destinations > Eventhouse
3. Select the pencil icon on the Eventhouse tile.
4. Enter the following information in the Eventhouse pane:

| Field	            |   Value                                                       |
| ------            |   -----                                                       |
| Destination name  |	TutorialDestination                                         |
| Workspace         |	Select the workspace in which you created your resources.   |
| Eventhouse        |	Your Eventhouse name                                        |
| KQL Database      |	Your Eventhouse name                                        |
| Destination table |	Create new "ex, fryer" as table name                        |
| Input data format |	Json                                                        |

5. Ensure that the box Activate ingestion after adding the data is checked.
6. Select Save.
7. From the menu ribbon, select Publish.
8. Switch to the eventhouse experience and make sure the table was created and populated successfully

### Part 4: Normalize and Randomize telemetry data
By the time of writing this tutorial the simulate OPC PLC server we are using is still undergoing some tuning and optimization as the telemetry tags generated (Temperature, EnergyUse and FillWeight) are all identical and incremental. Thus we will need to make a KQL query to make sure they are all random and normalized between 150-300.
1. From the navigation bar, select the KQL database you created in a previous step, named fryer_eh.
2. Verify that the data is flowing into the database by viewing the Size tile in the database details page. The values in this tile should be greater than zero. If the values in the Size tile are zero, select Refresh from the menu ribbon.
3. From the menu ribbon, select New related item and choose KQL Queryset.
4. Enter the name for the KQL Queryset: "ex, fryer_qs" and select Create.
5. Select the fryer_eh database as the data source for the KQL queryset, then select Connect
6. Select Create. A new KQL queryset is created and opens in the KQL Queryset editor. It's connected to the fryer_eh database as a data source, and is pre-populated with several general queries.
7. Clear the query text and type the following code:

```
fryer
| extend 
    TransformedTemperature = 150.0 + ((Temperature % 1000.0) / 1000.0) * (300.0 - 150.0) + (rand() * 10.0 - 5.0),
    TransformedEnergyUse = 50.0 + ((EnergyUse % 1000.0) / 1000.0) * (200.0 - 50.0) + (rand() * 5.0 - 2.5),
    TransformedFillWeight = 20.0 + ((FillWeight % 1000.0) / 1000.0) * (100.0 - 20.0) + (rand() * 4.0 - 2.0)
| extend 
    TransformedTemperature = iff(TransformedTemperature < 150.0, 150.0, iff(TransformedTemperature > 300.0, 300.0, TransformedTemperature)),
    TransformedEnergyUse = iff(TransformedEnergyUse < 50.0, 50.0, iff(TransformedEnergyUse > 200.0, 200.0, TransformedEnergyUse)),
    TransformedFillWeight = iff(TransformedFillWeight < 20.0, 20.0, iff(TransformedFillWeight > 100.0, 100.0, TransformedFillWeight))
| extend 
    TransformedTemperature = toint(TransformedTemperature),
    TransformedEnergyUse = toint(TransformedEnergyUse),
    TransformedFillWeight = toint(TransformedFillWeight)
| project TransformedTemperature, TransformedEnergyUse, TransformedFillWeight, EventEnqueuedUtcTime
```

### Part 5: Create a Real-Time Dashboard
1. From the Real-Time Analytics experience click Real-Time Dashboard and give it a name (ex, fryer_dashboard)
2. From the top menu click “Base queries”
3. From the right blade click “+ Add”
4. In the “Create base query” blade – in the “Variable name” type “_fryer_all”
5. In the “Select a data source” drop down click “+ OneLake data hub”
6. From the list choose the Kusto database "ex, fryer_eh" then clck connect then click add
7. In the “Create new data source” blade – under the Display name” type “kql-aio” and from the “Database” drop now select fryer_eh" then click Add
8. In the query body type the below code again to be the basis of dashboard elements then click “Run”. Make sure that you see results then click “Done” then Close

```
fryer
| extend 
    TransformedTemperature = 150.0 + ((Temperature % 1000.0) / 1000.0) * (300.0 - 150.0) + (rand() * 10.0 - 5.0),
    TransformedEnergyUse = 50.0 + ((EnergyUse % 1000.0) / 1000.0) * (200.0 - 50.0) + (rand() * 5.0 - 2.5),
    TransformedFillWeight = 20.0 + ((FillWeight % 1000.0) / 1000.0) * (100.0 - 20.0) + (rand() * 4.0 - 2.0)
| extend 
    TransformedTemperature = iff(TransformedTemperature < 150.0, 150.0, iff(TransformedTemperature > 300.0, 300.0, TransformedTemperature)),
    TransformedEnergyUse = iff(TransformedEnergyUse < 50.0, 50.0, iff(TransformedEnergyUse > 200.0, 200.0, TransformedEnergyUse)),
    TransformedFillWeight = iff(TransformedFillWeight < 20.0, 20.0, iff(TransformedFillWeight > 100.0, 100.0, TransformedFillWeight))
| extend 
    TransformedTemperature = toint(TransformedTemperature),
    TransformedEnergyUse = toint(TransformedEnergyUse),
    TransformedFillWeight = toint(TransformedFillWeight)
| project 
    Temperature = TransformedTemperature, 
    EnergyUse = TransformedEnergyUse, 
    FillWeight = TransformedFillWeight, 
    Time = EventEnqueuedUtcTime 
```
9. From the Real-Time Dashboard main window click “+ Add tile”
10. In the query window type the following then click Run:
```
_fryer_all
| top 1 by Time desc
| project Temperature
```
11. From the lower pane click “+ Add visual”
12. In the right blade type "Temperature" in the Tile name, choose “Stat” from the “Visual type” drop down menu, choose “Large” from the Text size, choose "Temperature" (real)” from the Data menu, and from the Conditional formatting click “+ Add rule” then click pencil icon
13. Configure the Conditional formatting rule as you prefer then click save
14. Click “Apply changes” from the top right
15. Adjust the tile size and location as needed in the dashboard
16. You might want to repeat the previous steps 9-15 for EnergyUse and FillWeight to create similar tiles
17. Create another tile of type "Linechart" for the below query
```
_fryer_all
| make-series avg(Temperature) on Time in range (_startTime, now(), 1m) 
```
18. Enable Auto refresh every 30 seconds for the dashboard
    1.  From the menu bar click Manage
    2.  Click Auto refresh
    3.  From the Auto refresh blade toggle the Enable button
    4.  From the Default refhesh rate drop down choose Continuous
    5.  Click Apply
19. From the Menu bar click Home
20. Click Save

