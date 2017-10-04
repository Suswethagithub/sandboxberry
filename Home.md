# Using SandboxBerry

When you run SandboxBerry you get a screen like this:

![Screenshot]( https://github.com/codeulike/SandboxBerry/raw/master/Doc/screenshots/SandboxBerryScreenshot1.png   )

Basically, to run it, you:

* Enter Source login details (Source will generally either be the Live instance or the Full sandbox, because thats where the data is) 
* Enter Target login details (this will be a Sandbox, preferably a new one with no data at all) 
* Choose an Instructions File (see below) 
* Press Start

Progress will then be shown as a series of messages on the screen.

# Instructions Files

SandboxBerry takes its instructions - which objects/tables to transfer, which filters to apply etc - from XML files.

A couple of example files are in the ExampleInstructionsFiles subfolder.

Here's a simple Instruction File:

    <?xml version="1.0" encoding="utf-8"?>
    <SbbInstructionSet xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema">
      <SbbObjects>
        <SbbObject ApiName="Country__c" />
        <SbbObject ApiName="Nationality__c" />
        <SbbObject ApiName="Account" Filter="Help_Sandbox_Data_Set__c = true" />
        <SbbObject ApiName="Client__c" Filter="Help_Sandbox_Data_Set__c = true"  />
      </SbbObjects>
    </SbbInstructionSet>


For each object/table you want SandboxBerry to transfer, add an <SbbObject> tag and give the ApiName of the object.
The above file will transfer Country__c, Nationality__c, Account (with a filter) and Client__c (with a filter)

(In this example Checkbox fields called Help_Sandbox_Data_Set__c has been added to Live Salesforce to allow admins to choose which Account and Client__c records will be transferred as test data.)

Filters are expressed as normal SOQL where clauses, so you need to know the basics of SOQL to write them (You can experiment with the Developer Console in Salesforce).

In the Instructions file you can also specify that certain fields are skipped, or replaced with fixed text, like so:

	...
	<SbbObject ApiName="Client__c" Filter="Help_Sandbox_Data_Set__c = true" >
	  <SbbFieldOptions>
		<SbbFieldOption ApiName="Email__c" ReplaceText="madeupemail@testesttest.com" />
		<SbbFieldOption ApiName="Home_Email__c" ReplaceText="madeupemail@testesttest.com" />
		<SbbFieldOption ApiName="Work_Phone__c" Skip="true" />
	  </SbbFieldOptions>
	</SbbObject>
	...

For the object "Client__c", this snippet tells SandboxBerry to replace Email__c and Home_Email__c with a pretend email address (same for all records). It also says to Skip the field Work_Phone__c completely, and so leave it blank in the target database.

# Relationships Between Objects

In the above example, Country__c, Account and Client__c are all being transferred. In the data model, let's say they are interrelated: Account has a Country, and Client__c is a child of Account and also has a lookup to Country__c.

Maintaining this relationship while copying the data to a new sandbox takes a bit of work on SandboxBerry's part, because the data in the Target sandbox will have different Ids to the Source data, meaning that all relationships need to be re-mapped.

SandboxBerry can correctly map relationships as long as:
* The related objects are in the Instructions File in the right order. 
* The target tables are all blank 
* All related objects are transferred in the same run

That last point means: You can't transfer Country__c in one run, and then do a separate run to transfer Account. SandboxBerry needs to keep track of the old id/new id pairs as it runs through the instructions, so it needs to do everything in one go.
