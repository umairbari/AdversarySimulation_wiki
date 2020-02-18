
## Workflow Example

  1. Go to your Splunk Web interface
  2. Select "Attack Board" from the app navigator
  3. Find the test you want to execute, right click on it and select "run test" at the bottom of the menu
  4. In the Simulation Runner app, fill in the dropdown boxes appropriately and then click "Submit"
  5. You should see something like this in the "Job Status" panel:
  ```
  sendtophantom - Alert action script completed in duration=2902 ms with exit code=0
  ```
  6. If you don't see "exit code=0", there is some error in executing the tests
  7. Assuming you get exit code=0, you should see at least one event in the next panel, "Phantom POST-ed events matching GUID".  
   - If you don't have any events, Phantom is not POSTing to SPLUNK
   - If you only have one event, hover over this panel and then click the circle arrow in the bottom right to refresh this panel.  You will need to do this until you see two events in the panel to get your time bracket.  The test should only take about 10-20 seconds to complete.  
   - If you have refreshed after 30-60 seconds and you still don't have at least two events in the panel, you will need to switch over to Phantom to figure out why the test did not complete successfully.
