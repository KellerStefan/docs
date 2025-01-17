---
title: "Batch Integration"
parent: "integration-solutions"
menu_order: 4
draft: true
---

## 1 Introduction

More and more processes are becoming real-time, but a lot of business processes are still periodic in nature (for example, salary payments and interest calculations). These use cases are best implemented in batch-oriented integration, which runs a sets of data at certain moments. 

Batch processing is also advantageous for very high-volume integration, because it is more economical from a network and CPU perspective for transferring and processing the data in batches. Monitoring and IoT solutions, for example, usually have agents or aggregators that collect a set of input and send the data on as small batches in close to real-time.

Batch processing means that many records are processed together, which can be done in two ways:

* File-based integration, exporting a file, moving the file, and later importing it into one or more destination apps
* Service-based batch integration, meaning that a REST, OData, or SOAP interface works on batches of data (for example, retrieving 5000 customers at a time and processing them)

There are these typical use cases:

* [Export and import processes](#export-import) – In this use case, data is exported from one system and imported in another system.
* [Reference data management](#reference) – Reference data management is usually done in batch, because a snapshot of a situation is generally desired and several systems may need to work on the same snapshot. This is often done at night, to prevent the export/import processing from interferring with normal operations, and/or to take advantage of the CPU power available at night when it is usually less busy.
* [File management](#file-management) – This use case is important for moving files in batch-oriented integration.It is also important for providing files of various types to end-users and customers.
* [Data lakes, DWH, and BI integration](#int) – This is an area that usually works in batch mode. Mendix provides data to these solutions for analysis, statistics, reporting, business intelligence, and also machine learning. In some automated processes, Mendix also receives data from these solutions and turns it into reference data that helps in optimizing automated business flows (for details, see the [Self-Learning Processes Using Data Lakes](central-datadata-lakes) section of *Central Data*).
* **Monitoring and IoT solutions** – In these use cases, data is often batched up data in small packages and sent at high speed. For more information, see [Ops & CI/CD Integration](ops-cicd-integration) as well as the [Examples for IoT with Mendix](event-integration#iot-examples) section of *Event-Integration*.

{{% todo %}}[**Need a section on "Monitoring & Iot Solutions" below in order to link above, as is done for all other use cases**]{{% /todo %}}

## 2 Export & Import {#export-import}

Batch processing runs a large set of data by first exporting the set, moving it, and then importing the data. These three actions can happen at different times, and the systems can be relatively unaware of each other besides agreeing on a file format, file name, and file location 

This diagram shows the three main steps of batch processing: 

![](attachments/batch-integration/export-import.png)

Processing data in bulk optimizes CPU-usage. It is often done at night, when the other load is much lower in order to prevent the batch processing load from interferring with operational processes or suddenly making the UX slower.

Users frequently import and export files of various formats with the Mendix Platform, and the level of skills required depends on the format of the file. For unusual file formats, a more Technical Developer is recommended. For common formats such as *csv*, a Business Developer should be able to use the the [Excel Importer](https://appstore.home.mendix.com/link/app/72/) module from the Mendix App Store by just selecting a field separator that is not the same as the text that in the fields.

If there is a large amount of data to import, the best practice is to write to a process queue. This prevents the entire dataset from being in the memory at the same time. For more information, see the [Using Internal Queues](event-integration#internal-queues) section in *Event-Based Integration* as well as [Example – File Import Integration with CSV](csv).

## 3 Reference Data with Mendix {#reference}

Many organizations use Mendix to manage reference data as microservices that each have a functional area of responsibility:

* Reference data that is semi-static (for example, vehicles, drivers, addresses, products, shop opening hours)
* Business intelligence statistical information that is turned into reference data to fine-tune processes
* Master data (for example, customers) that changes in real-time while being used in several processes across the enterprise

There area three typical examples where Mendix plays a role in master and reference data use cases:

* **Reference data apps** – These apps typically share data periodically as files. This is because there are many subscribers that need to work on the same reference data set. For example, ordering, engineering, and financing all need to be aware of the same products for the order-to-cash process to work properly.
* **Local shared data apps** – These "SDA" apps are typical for microservics systems consisting of 3–10 apps. They manage the import and distribution of reference data within that cluster of apps. Other apps can poll for differences instead of all importing a full set of data. The human workflow can define the shared data and manage issues in import/export. The shared data app can also provide a combined view of data for the rest of the world (for more information, see [Central Data](central-data).
* **Master data apps** – An example of this is a central customer app. Such apps often need to share both in real-time for operational processes (for details, see [Service Integration](service-integration)) as well as via files (for example, to ERP solutions) to prepare for invoicing later.

This diagram shows an example for managing shared product and customer data across Mendix apps and other systems (such as, engineering and finance systems):

![](attachments/batch-integration/bi-2.png)

In the diagram's example, you can see the following factors at work:

* New or changed product definitions should only be shared at decisive moments, so the business process is periodic and all three down-stream systems need to have the data at the same time (meaning, this is a perfect case for batch integration with files). New data will only be shared at night, when no ordering occurs, and only some nights when marketing, sales, and engineering agree it can be released. A file transfer protocol  (FTP) server (or any other file-transfer management system) acts as a central point of contact for the files.
* A **Customer** master data app is outside in an order-to-cash cluster to serve other areas as well. It can be reached via a deep link to create new customers when required, and this data is transferred almost directly to be able to place orders against it. 
* The order-to-ship process is complex, so it has been broken up into the following microservices:
	* A dashboard for SSO and overviews
	* One separate app for each phase: **Order**, **Contract**, and **Ship**
	* A **Shared Data App** to manage all shared data
* As the **Shared Data App** is importing the product reference data file, it marks all products that change as updated. That is so the other apps can subscribe to those changes via a REST call, thereby minimizing the need for processing and allowing the other apps to be ignorant as to how the product data is distributed globally.

## 4 File Management with Mendix {#file-management}

File integration is important for the following cases:

* Batch integration, as decribed above
* Sharing content files, such as PDFs, photos, images, manuals, brochures, binaries, and 3D models
* Logging, monitoring, and backups

### 4.1 Mendix File Storage

Each Mendix application has a dedicated file storage area, where it writes files to by default. This is also where the Mendix app log file is located by default. The size of this area is large enough to handle most regular file management. Also, when importing a file, it is practical to copy the file to this local area first before processing it.

In rare cases, the file space can be extended by filing a Mendix Support request. This could occur if the purpose of the Mendix app is to help distribute files like manuals, documents, or pictures. For more information, see the [Example – Manuals for Product Support](#example) section below.

### 4.2 File Transfer Options

The diagram below presents the different options for sharing or transfering files between two apps. In almost all cases, the source app creates the file locally first, after which it is transferred. The exception is when a shared folder location is available, in which case both apps can write and read from the same folder.

![](attachments/batch-integration/file-integration.png)

These are the five options displayed in the diagram:

* **Source app creates file locally** – The destination app calls the source app with REST or SOAP to get the file as an attachment in a JSON or XML message. This is efficient for files smaller than approximatelly 5–10 MB.
* **Source app writes the file directly in a shared folder** – At the end of the process, the file is renamed to the target name. The destination app will be able to read the file as soon as it is renamed to the agreed name.
* **Move the file via FTP or SFTP** – This requires an FTP server to be available. The source app pushes the file to an FTP server, which stores it locally until one or more subscribers can pick it up using FTP get. For more information, see the [SFTP](https://appstore.home.mendix.com/link/app/107256/) connector available in the Mendix App Store.
* **Managed File Transfer (MFT) or ESB** – A special tool may be available for managed file transfer (MFT), or an ESB can be adapted to handle the transfer. Typically, the source app pushes the file to this solution, which in turn creates a copy per destination in another folder, accessible only to one subscriber each. MFT solutions verify that files are picked up and processed within agreed timeframes, and it can raise alarms when this is not the case.

In larger organizations the file and batch process management can be quite elaborate. There is often a central scheduling solutions that expect files at certain times, and that orchestrates the moving of files and the export and import of the files and any errors that occur in this process. Mendix will then integrate with both the scheduling solution and the file transfer solution.

### 4.3 Manuals for Product Support {#example}

In the example visualized in the diagram below, the customer can browse all products, register usage of specific products, and download information and documentation related to these products. In this case, file management is at the core of the solution.

![](attachments/batch-integration/bi-4.png)

In this example, you can see the following process at work:

1. The departments that own the product groups will send files with information to the **Customer Portal** (for example, product sheets with metadata, manuals, pictures, and 3D models). Updates are periodic, so batch processing is a good way to handle updates.
2. The Mendix **Customer Portal** app will poll for new product files and move them to the expanded internal file storage area, registering in the database some file metadata, and linking files to products or product groups.
3. The app imports some of the files with product metadata, to be searchable in the app and directly readable.
4. Corporate customers can browse all products and related marketing information. They can also register a list of products that they use, download related manuals and documentation, and receive alerts when documents are updated. PDFs, Excel files, and other common format files can be read directly in the Mendix app's UI.

## 5 Data Lakes, DWH & BI Integration {#int}

Interfaces towards DWH and BI are often bulk- and/or snapshot-oriented. The same is true for initial loads of systems and the distribution of reference data. 

Extract, transform, load (ETL) tools are used to keep DWH and data lakes updated. They can perform the entire operation of extracting, moving, validating, transforming, and updating the destination. For legacy systems, they can use direct database access or use files as input. When Mendix integrates with ETL or BI, these are the preferred methods:

* Exporting a file for periodic dumps of specific data
* Using OData for more frequent and smaller updates
* Using the Mendix database backup file, if all data with the relations is required

This diagram presents the three most used methods for such a solution, which involve OData, files, or a database backup:

![](attachments/batch-integration/dwh.png)

Mendix recommends using either files or OData and to restrict the amount of data that is shared. This also limits dependencies. BI and DWH should have a specific purpose with gathering the data, and minimizing the retrieval to this purpose will create the simplest overall solution.

These are further factors to consider:

* **OData** – OData is positioned to collect smaller data chunks more close to real-time. This is typically useful for BI solutions or business dashboards. This requires the BI solution to have a direct link to the app. In addition, if the Mendix data model of the monitored data object changes, it will often require the BI solution to update the retrieval as well.
* **Files** – Files is the most decoupled option to share data with BI, ETL, and DWH. There could be several files with different data and functional IDs that link objects together. If this option is possible, it is recommended.
* **Databse backup** – When a DWH wants almost all the data, when the domain model is complex, or when there are several important many-to-many relations, then a database dump is available as the best option. An ETL solution will be directly depending on the Mendix data model, creating a tight coupling that forces the ETL solution to change with every new Mendix app release. To handle this dependency, ETL solutions have a staging area where raw data is imported, so they can actually adopt even after changes occur, when errors are discovered.

## 6 When to Use Batch Processing {#using}

Even as the world moves towards real-time, batch integration provides more efficient processing and the ability to move processing to a time of day when the app instance is less busy. It also provides decoupling for business processes that are periodic in nature or where systems can not connect directly.

* **Periodic business process** – When a business process is periodic, the correct way to process the integration is to work on a snapshot of the data. In that case, batch processing is the best way to work, for example:
	* Periodic financial processes, such as salary payments, interest calculations, and invoicing.
	* Reference data, such as products, drivers, employees, should be shared with operational systems at a certain times.
	* Backups and other operational processes are typically done at night.
* **Processing advantage** – There is a processing advantage when working in batch. The initiation of an event or service call has a cost in processing power, so it is much more efficient to extract, move, and import a million records in one go than it is to initiate 1 million separate events or service calls. The advantage becomes relevant at around 50,000 records, and significant above 500,000 records.
* **Night-time processing** – To combine the two reasons above, with batch processing, some heavier processing can be moved to the night to save on CPU cycles during the day. This means you may require a less expensive infrastructure. This can also improve response times for end-users during the day.
* **Decoupling** – Batch integration is decoupled, which means the following factors occur:
	* The export and import can happen at different times. In fact, the import can be done several times if required. 
	* There is a clearly defined file format that has been agreed on. The means that two systems can be very far from and virtually unaware of each other, and they will still integrate well. Often in a Mendix app project, the developer only knows there is a file of reference data they need to import, while the origin of the file may be unclear or irrelevant.

### 6.1 Reasons to Avoid Batch Processing

There are two main reasons not to go for batch-oriented integration: 

* **Real-time processing** – Because we are moving more towards a real-time world, processes that used to be periodic are becoming real-time. For example, invoicing was previously almost always monthly. Nowadays, invoicing is frequently done in real-time, when an order is confirmed or a delivery is completed.
* **Complicated error handling** – Imagine if 5 records out of 1 million fail to import. In that case, there is a need to inform the source about this, or correct the file, or have a human workflow in the destination to manage these errors.

### 7 How to Do Batch Processing

### 7.1 Triggering Batch Processes

These are the batch process triggering methods:

* **Periodic process** – triggered using Mendix scheduling or by receiving an event from a central scheduler
* **Ad-hoc process** – triggered by manual invocation via a Mendix microflow
* **Via a business event** – often triggered by a REST call or a file that is created in a location where a Mendix app is polling for a file with a specific name

### 7.2 File-Based Batch Processing

The following elements are typically needed:

* A defined format to exchange data (for example, CSV, XML, Excel, fixed delimited)
* A manner of transporting the data (for example, (S)FTP, HTTP, file (disk) access)
* Possible interpretation (for example, masks for complex fields or logic to interpret an expected date format)
* Possible transformation (for example, associate an imported address to a master-data country)

### 7.3 Service-Based Batch Processing

This option focuses mainly on file export and import, which is what most people associate with batch processing. However, batch processing just means that many records are processed at a time. 

In many cases, this is done using "real-time" integration via REST, SOAP, or OData in order to avoid the management and transport of files. If the dataset is smaller than a few MBs, a REST service works well for importing a few hundred thousand records. This is a lot more efficient than processing one record at a time.

If the data is too large, services can still be used while making a set of calls in a row and keeping track of the last imported record. For example, 5 million records could be imported using one thousand REST calls.

### 7.4 Batch Processing Do’s & Don'ts

#### 7.4.1 Do's

* Uniquely identify records and store them in the application (to be able to update later, if applicable)
* Index unique identifiers
* Import data while there are fewer other processes running in order to avoid interference
* Consider a strategy to handle deleted data (for example, mark as deleted and keep in the database or remove during the import)
* Think of the correct error handling (for example, should only a single object fail or the complete batch?)
* Preserve a trail of import/export statistics (for example, how many records were new, changed, removed, and exported)
* Implement a process to verify if imports are running successfully – if they are not, most of the time there should be an action taken (for example, requesting a record to be re-sent from the source or correcting the data)

#### 7.4.2 Don’ts

* Don't apply batch imports/exports when data is expected to be updated in real time
* Don't apply heavy batch processing during peak hours of system usage

### 7.5 Technology Options with the Mendix Platform

The following patterns are suitable options for batch import and export:

* Batch
	* File (for example, CSV, Excel, XML)
	* Database (for example, JDBC)
* RPC
	* REST
	* SOAP
	* OData
	
#### 7.6.1 When to Apply Each Technology Option

The technology option selected is often limited by the environment (for example, a remote system only exports its data in a certain manner).

This table presents the pros and cons of each technology option:

|     | Pros | Cons |
| --- | --- | ---|
| **Batch** | Export/import is part of batch integration. | |
| **File** | Large volume in one transaction. | Intermediate transfer required (for example, SFTP). |
| **Database** | | Connectivity to database required. |
| **RPC** | | Multiple requests, pagination, and transactions. |
| **REST** | Using standard HTTP(S) connectivity. Part of Mendix Core. |   |
| **SOAP** | Using standard HTTP(S) connectivity. | |
| **OData** | Using standard HTTP(S) connectivity. | Does not support binary interface. |

## 8 Summary

Batch processing means that a larger set of records are processed together, and it is required of all enterprises in a variety of situations. The most common method is file-based batch processing, but you should consider that batch processing can also be done using services, such as REST or OData.

File-based batch integration is relevant for many periodic business processes, when a lot of data needs to be transferred, or when service-integration is not available. The diagram below illustrates the three main steps involved in file-based batch integration:

1. Extract a set of data (for example, into a file).
2. Move the data.
3. Import the data.

![](attachments/batch-integration/bi-intro.png)

The Mendix Platform supports batch processing very well, with an internal scheduler, good export and import functionality, and support for REST and OData. Mendix apps have an internal file storage area and can move or get files to and from that area using a central FTP server or another managed file transfer solution available in the company.
