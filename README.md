### Tripartite Process Model ###
**Robotic Enterprise Framework**

Many business processes in the enterprise world follow this pattern:

* Some Excel files come by email
* Actions required for each row
* Updated Excel files sent back

To automate this kind of process with UiPath, we split it in three phases:

1. Monitor email, download attachments and upload data to Orchestrator queue
2. Process items in queue with several robots in parallel to keep up with volumes
3. Assemble data back to Excel files with new info/status and send them back

Leading phases 1&3 can be executed by a robot in one *Dispatcher* project with a recursive schedule.
The main *Performer* structure handles application interaction and status updates in the 2nd phase.

### ReFrameWork Dispatcher ###

Filter email, save Excel attachments and load all data into an Orchestrator transaction queue
	* Check out *Dispatcher/FetchAttachments* that filters Inbox email by Config("EmailFilter") and downloads the attachments
    * Check out *Dispatcher/OnloadQueue* that loads data from Excel files in a queue, tracking the file it belongs to and the row index

Reassemble the Excel files with statuses from the secondary queue and send them when complete via email
    * Check out *Dispatcher/OffloadQueue* sample that saves items from a queue to Excel files using a header *ExcelTemplate.xlsx*
    * Check out *Dispatcher/SendStatusEmail* that uses a *EmailTemplate.txt* file with String.Format for the message body


### ReFrameWork Performer ###

* built on top of *Transactional Business Process* template
* using *State Machine* layout for the phases of automation project
* offering high level exception handling and application recovery
* keeps external settings in *Config.xlsx* file and Orchestrator assets
* pulls credentials from *Credential Manager* and Orchestrator assets
* gets transaction data from Orchestrator queue and updates back status
* saves screenshots and error messages in case of application exceptions
* runs out of the box sample Notepad application with dummy Excel input data

** How It Works **

1. **INITIALIZE PROCESS**
 + *InitiAllSettings* - Load config data from file and from assets
 + *InitiAllApplications* - Login to applications as per Config("OpenApps") field
   + *GetAppCredentials* - From Orchestrator assets or local Credential Manager
 + If initialization failing, retry a few times as per Config("ProcessRetries")

2. **GET TRANSACTION DATA**
   + ./Framework/*GetTransactionData* - Fetch from Orchestrator queue as per Config("TransactionQueue")
   + ./*GetTransactionData* - Fetch a row from sample Excel input file

3. **PROCESS TRANSACTION**
 + Try *ProcessTransaction*
 + If application exceptions happen
   + *TraceSystemError* - Saves screenshot and error message In Config("ErrorsFolder")
   + Going to re/INITIALIZE
 + *SetTransactionStatus* - As Success, Failed or Rejected with reason
   + ./Framework/*SetTransactionStatus* - Updates Orchestrator queue item
   + ./*SetTransactionStatus* - Updates sample Excel row

4. **END PROCESS**
 + *CloseAllApplications* - As listed in Config("CloseApps")


### Starting a Project ###

1. Check out the Config.xlsx file and add/customize any required fields and values
2. Implement OpenApp and CloseApp workflows in your App folder, linking them in the Config.xlsx fields
3. Implement GetTransactionData and SetTransactionStatus or use ./Framework versions for Orchestrator queues
4. Implement ProcessTransaction workflow and any invoked others
