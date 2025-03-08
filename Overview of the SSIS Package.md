## **Overview of the SSIS Package**

1. Loop through files in the Source Folder (ForEach Loop Container).  
2. Extract metadata and insert a record in MetaData table (Execute SQL Task).  
3. Load the data from the file into the destination table (Data Flow Task).  
4. Update the metadata record count after successful data load (Execute SQL Task).  
5. Move the file to Archive/Error folder (File System Task).

## **Step 1: Create SSIS Variables**

Create Project-Level Variables in SSIS:

* SourceFolder (String): Path where raw files are stored.  
* ArchiveFolder (String): Path where processed files are moved.  
* ErrorFolder (String): Path where failed files are moved.  
* FileName (String): Stores current file name being processed.  
* TableName (String): Extracted table name from FileName.  
* RecordCount (Int32): Number of rows loaded into the table.  
* MetaDataID (Int32): Stores the metadata ID for updating record count.

## **Step 2: Add a ForEach Loop to Process Files**

1. Drag a ForEach Loop Container into the Control Flow.  
2. Configure the Collection:  
   * Enumerator: ForEach File Enumerator.  
   * Folder: @\[User::SourceFolder\].  
   * Files: \*\_????????.txt (Regex-style pattern for YYYYMMDD).  
3. Map Variables:  
   * FileName → Index 0\.

## **Step 3: Extract Table Name from FileName (Script Task)**

Use the following C\# script:

public void Main()  
{  
    string fileName \= Dts.Variables\["User::FileName"\].Value.ToString();  
    string tableName \= fileName.Substring(0, fileName.LastIndexOf('\_'));  
    Dts.Variables\["User::TableName"\].Value \= tableName;  
    Dts.TaskResult \= (int)ScriptResults.Success;  
}

## **Step 4: Insert Metadata Record (Execute SQL Task)**

DECLARE @ID INT;  
EXEC dbo.spInsertMetaData  
    @Vendor \= 'System',  
    @IngestFileName \= ?,  
    @IngestDBName \= 'MyDatabase',  
    @TableName \= ?,  
    @recordcount \= 0,  
    @ID \= @ID OUTPUT;  
SELECT @ID;

## **Step 5: Load Data into the Table (Data Flow Task)**

1. Add a Flat File Source:  
   * File Path: @\[User::SourceFolder\]\\@\[User::FileName\]  
   * Delimiter: | (Pipe)  
   * Skip First Row: Enable Column Names in the First Data Row  
2. Connect to OLE DB Destination and map columns.  
3. Use Row Count Transformation to track rows.

## **Step 6: Update Metadata Record Count (Execute SQL Task)**

EXEC dbo.spUpdateMetadataRecordCount @MetaDataID \= ?, @RecordCount \= ?

## **Step 7: Move File to Archive/Error Folder (File System Task)**

* Move file to Archive if successful.  
* Move to Error folder if there’s a failure.

## **Step 8: Handle Errors**

* Use OnError Event Handler.  
* Log errors in a database.  
* Route failed files to the Error folder.

## **Expected Behavior**

* SSIS loops through files, extracts table names, and logs metadata.  
* Loads data into the correct table and updates metadata.  
* Moves processed files to Archive/Error folder.

Would you like to add logging and email alerts for failures?

