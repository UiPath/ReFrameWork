### ReFrameWork Template ###
**Robotic Enterprise Framework**

* built on top of *Transactional Business Process* template
* using *State Machine* layout for the phases of automation project
* offering high level exception handling and application recovery
* keeps external settings in *Config.xlsx* file and Orchestrator assets
* pulls credentials from *Credential Manager* and Orchestrator assets
* gets transaction data from Orchestrator queue and updates back status
* takes screenshots in case of application exceptions
* provides extra utility workflows like sending a templated email
* runs sample Notepad application with dummy Excel input data
* 


### How It Works ###

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


### Extra Workflows ###

Many enterprise processes nowadays start and end with an Excel file attached to an email
In a robotic world, such scenarios have three phases:
1. Filter email, save Excel attachments and load all data into an Orchestrator transaction queue
* Check out the *Xtras/OnloadQueue* sample that loads data from Excel in a queue tracking the file it belongs to and the row index
2. Process the items in the transaction queue with multiple robots using *ReFrameWork template* passing items and statuses to a secondary queue
3. Reassemble the Excel files with statuses from the secondary queue and send them via email
* Check out the *Xtras/OffloadQueue* sample that saves items from a queue to Excel files
* Check out the *Xtras/SendStatusEmail* that uses a template file for the message body
