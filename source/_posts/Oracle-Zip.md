---
title: Oracle Zip壓縮檔案
date: 2023-02-11 22:46:43
tags:
- [Zip]
- [Script]
categories:
- [Oracle]
---

將目錄下的txt壓成一個zip檔的範本

<!--more-->

## Java Zip Code

```java
import java.io.*;
import java.util.zip.*;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;
import java.util.zip.ZipEntry;
import java.util.zip.ZipOutputStream;


public class zipFile
{
    //java zipFile 來源目錄 zip檔案(含目錄)
    //java zipFile d:/temp d:/temp/t.zip
    List<String> fileList;
    zipFile(){
	   fileList = new ArrayList<String>();
    }

    public static void main( String[] args )
    {
    	zipFile appZip = new zipFile();
        String SOURCE_FOLDER = args[0];
        String OUTPUT_ZIP_FILE = args[1];
    	appZip.generateFileList(new File(SOURCE_FOLDER),SOURCE_FOLDER);
    	appZip.zipIt(OUTPUT_ZIP_FILE,SOURCE_FOLDER);    	
    }

    /**
     * Zip it
     * @param zipFile output ZIP file location
     */
    public void zipIt(String zipFile,String s_folder){

     byte[] buffer = new byte[1024];

     try{

    	FileOutputStream fos = new FileOutputStream(zipFile);
    	ZipOutputStream zos = new ZipOutputStream(fos);

    	System.out.println("Output to Zip : " + zipFile);

    	for(String file : this.fileList){

    		System.out.println("File Added : " + file);
    		ZipEntry ze= new ZipEntry(file);
        	zos.putNextEntry(ze);

        	FileInputStream in =
                       new FileInputStream(s_folder + File.separator + file);

        	int len;
        	while ((len = in.read(buffer)) > 0) {
        		zos.write(buffer, 0, len);
        	}

        	in.close();
    	}

    	zos.closeEntry();
    	//remember close it
    	zos.close();

    	System.out.println("Done");
    }catch(IOException ex){
       ex.printStackTrace();
    }
   }

    /**
     * Traverse a directory and get all files,
     * and add the file into fileList
     * @param node file or directory
     */
    public void generateFileList(File node,String s_folder){

    //add file only
	if(node.isFile()){
	   if (node.getAbsoluteFile().toString().toLowerCase().lastIndexOf(".txt") >0 ) {
		  fileList.add(generateZipEntry(node.getAbsoluteFile().toString(),s_folder));
       }
	}

	
	if(node.isDirectory()){
		String[] subNote = node.list();
		for(String filename : subNote){
		   generateFileList(new File(node, filename),s_folder);
		}
	}

    }

    /**
     * Format the file path for zip
     * @param file file path
     * @return Formatted file path
     */
    private String generateZipEntry(String file,String s_folder){
    	return file.substring(s_folder.length()+1, file.length());
    }
}

```





## 將java編譯好的class匯入到DB

```shell
set ORACLE_SID=xxx
loadjvav -user DB帳號 zipFile.clasee
```





## SYS執行

```sql
DECLARE
  l_schema VARCHAR2(30) := 'DB帳號'; -- Adjust as required.
BEGIN
  DBMS_JAVA.grant_permission(l_schema, 'java.io.FilePermission', '<<ALL FILES>>', 'read ,write');
  DBMS_JAVA.grant_permission(l_schema, 'SYS:java.lang.RuntimePermission', 'writeFileDescriptor', '');
  DBMS_JAVA.grant_permission(l_schema, 'SYS:java.lang.RuntimePermission', 'readFileDescriptor', '');
END;
/

--建立 Procedure
CREATE OR REPLACE PROCEDURE WH.zipFile(s_folder varchar2,zip_File VARCHAR2)
AS LANGUAGE JAVA
NAME 'zipFile.main(java.lang.String[])';
/
```





## 執行方式

```plsql
exec zipFile('d:/temp','d:/temp/t.zip');
```

