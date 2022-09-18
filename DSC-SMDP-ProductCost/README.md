# Introduction 
TODO: Give a short introduction of your project. Let this section explain the objectives or the motivation behind this project. 

# Getting Started

## Historical Data Extract:

Extract Historical Data from MARS:

- Export data from MARS. Run Select * from [MARS].[dbo].[vwProductCostGlobal] where CostingDate >= '2018-01-01'. Save as a pipe (|) delimited .csv file
- Open the file in Textpad and find all "NULL" and replace with empty content.
- Rename the file to vwProductCostGlobal.csv
- Upload the file to ADLS Gen 2 raw/6896_SAPBW/Downstream/ProductCost/


 
## Historical Data Load Logic(pl_ProductCostHistorical):

ProductCost Canonical: https://chevron.sharepoint.com/:x:/r/sites/DSCITOps/TS/dataFoundation/Shared%20Documents/Product%20Cost%20Canonical%20Mapping.xlsx?d=w28b0ed21a00b4e78b85bc10c12e5d0ca&csf=1&e=HEeaQi

Check if ProductCost.parquet exists in ADLS (produced).
	Check if ProductCost.txt exists in ADLS (produced). 
	If either file found, delete it. 
	Run df_ProductCostHistoricalLoad Data Flow
	
Read vwProductCostGlobal.csv
		Cast Columns 
		
Update all date columns to format MMDDYYYY
			Set number fields to long or decimal
			Set text fields to string
		
		
Add Columns		
Add FileName field. Set to 'MARS.dbo.vwProductCostGlobal'
			Add LoadDataTimestamp field. Set to CurrentUTC()
			Add SourceSystem to set to value of SubSourceCode in  vwProductCostGlobal.csv  (e.g PC2, PC8)
		
		
		Select output and order columns
		Sink to txt file under ADLS Gen 2 SalesManagement/ProductCost/Lubricants/ProductCost.txt
		Sink to parquet file under ADLS Gen 2 SalesManagement/ProductCost/Lubricants/ProductCost.parquet
	
	

## ProductCost Updates

Runs Weekly on Saturday and Monthly on the 1st of the month


	
Source files from ADLS Gen 2 raw/6896_SAPBW/Downstream/ProductCost/pcstcm*.dat
	Source destination file - ADLS Gen 2 SalesManagement/ProductCost/Lubricants/ProductCost.parquet
	The .dat files contain footer rows we need to remove. Remove where col0 (aka ProductCostEstimateNumber) = 0
	Rename columns
	Add SourceSystem and LoadDataTimestamp columns
	

		
Set SourceSystem will equal PC2 or PC8 based on source .dat filenames
		Set LoadDataTimestamp = now in UTC
	
	
	Union new data with existing data
	Add a composite id which equals ProductCostEstimateNumber + CostingDate + SourceSystem
	Dedup data based on Composite ID, keeping the latest data update
	Select to remove extra columns of CompositeID and RowNumber
	Sink as txt file under ADLS Gen 2 SalesManagement/ProductCost/Lubricants/ProductCost.txt
	Sink as parquet file under ADLS Gen 2 SalesManagement/ProductCost/Lubricants/ProductCost.parquet

# Build and Test
TODO: Describe and show how to build your code and run the tests. 

# Contribute
TODO: Explain how other users and developers can contribute to make your code better. 

If you want to learn more about creating good readme files then refer the following [guidelines](https://docs.microsoft.com/en-us/azure/devops/repos/git/create-a-readme?view=azure-devops). You can also seek inspiration from the below readme files:
- [ASP.NET Core](https://github.com/aspnet/Home)
- [Visual Studio Code](https://github.com/Microsoft/vscode)
- [Chakra Core](https://github.com/Microsoft/ChakraCore)