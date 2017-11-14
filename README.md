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
   + *SaveErrorScreen* - In Config("ErrorsFolder") with the exception message
   + Going to re/INITIALIZE
 + *SetTransactionStatus* - As Success, Failed or Rejected with reason
   + ./Framework/*SetTransactionStatus* - Updates Orchestrator queue item
   + ./*SetTransactionStatus* - Updates sample Excel file

4. **END PROCESS**
 + *CloseAllApplications* - As listed in Config("CloseApps")


### Starting a Project ###

1. Check out the Config.xlsx file and add/customize any required fields and values
2. Implement OpenApp and CloseApp workflows in your App folder, linking them in the Config.xlsx fields
3. Implement GetTransactionData and SetTransactionStatus or use ./Framework versions for Orchestrator queues
4. Implement ProcessTransaction workflow and any invoked others


### Extra Workflows ###

* AddDataToQueue - From sample Excel file, with file name field and item index per file
* SendStatusEmail - Using template file for the message body