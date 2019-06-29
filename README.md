# FHIR EMPI Interface
Version Beta 1.0  
28 June 2019  
Fast River Technologies  
www.fastrivertech.com  

charles.ye@fastrivertech.com

*work-in-progess*

## 1. Introduction

This document introduces a collection of FHIR-based EMPI (Enterprise Master Patient Index), or MPI (Master Patient Index), fundamental operations. FHIR R4 includes only match operation. The document defines the request of each EMPI operation,  the response of each EMPI operation, operation outcome details, and error conditions for the transactions of general EMPI operations.  Traditionally, the MPI interfaces are always vendor-specific, proprietary, and complicated. Integrating healthcare information systems with an EMPI is still intricate, and vendor locked. After broadly inspecting industry leading EMPIs including IBM Initiate / Infosphere, Oracle Healthcare MPI, OpenEMPI, InterComponentWae AG ICW MPI, NextGate EMPI, Verato EMPI, Senzing API, and more, we compile a set of simplified FHIR-based EMPI operations for central EMPI transactions. These operations are aimed to service as a unified EMPI interface for FHIR-based healthcare information systems integrating with an EMPI, overcome the challenges of EMPI integration, and make switching the systems to a different EMPI easier.     

## 2. Scope

This document describes the central FHIR-based EMPI interface by leveraging FHIR R4 operation style. It does not address the underlying implementation and integration of an EMPI.

## 3. Assumptions
This document assumes FHIR Specification R4, which is the current officially released normative version at the time of this writing.

## 4. Integrating FHIR with EMPI

An Enterprise Master Patient Index (EMPI) or a Master Patient Index (MPI) provides a single source of truth about a patient within or across healthcare organization systems, ensuring the accuracy and consistency of the unified and trusted patient information.  An EMPI generates a golden patient record, or a master patient record,  from a set of similar-enough patient records and assigns a unique enterprise -level identifier, or a master patient index, by using algorithms of matching, merging, and deduplicating.  These similar-enough patient records are identified as being for the same patient and linked together through the enterprise-level identifier.  The patient information used for matching is a subset of patient demographic information, including name, gender, date of birth, social security number, address, and contact information.  An EMPI implementation determines the match types.  

In FHIR, the terms patient and patient resource are used interchangeably. One or more identifiers assign a patient resource.  A namespace or system issues a unique identifier within the issuing system to a patient resource.  A namespace or a system also considers as a patient identification source domain or a patient source domain in EMPI aspect.  An EMPI is a patient identification master domain, which is also named EMPI domain in this document.  Each patient source domain assigns a unique source patient identifier to a source patient resource within the patient source domain. The EMPI domain assigns a unique enterprise-level master identifier, also known as a master patient index, to each golden patient record or master patient resource by the underlying EMPI.  A set of similar-enough patient resources from distinct patient identification source domains links together by a unique enterprise-level master identifier from the EMPI domain to embody the same patient.

Figure 4.1 depicts the relationship between two patient identification source domains and one EMPI domain.

![Figure 4.1](images/figure-4.1.jpeg)

Figure 4.1 Two Patient Identification Source ddmains and one EMPI domain

In Figure 4.1, an FHIR system includes two patient source domains, which are Patient Identification Source Domain A and Patient Identification Source Domain B. Patient Source A and Patient Source B assign unique source patient identifiers to source patient resources respectively and ingest the patient resources into the FHIR system through standard FHIR interactions and FHIR EMPI operations.  
An integrated EMPI manages the Patient Identification Master Domain and generates master patient identifiers. 

The Patient Source A issues a source patient identifier to a patient resource within the Patient Source Domain A.  The source patient identifier is a local identifier to  the Patient Source A.  The following is an example of an identifier:
```json
{  
   "use":"official",
   "type":{  
      "coding":[  
         {  
            "system":"http://www.patient-source-a.com/localid",
            "code":"PatientSourceA",
            "display":"Patient Source Domain A"
         }
      ],
      "text":"Patient Source Domain A"
   },
   "system":"urn:oid:2.16.840.1.113883.4.3.20",
   "value":"1000000001",
   "assigner":{  
      "display":"Patient Source Domain A"
   }
}
```  
The Patient Source B issues a source patient identifier to a patient resource within the Patient Source Domain B.  The following is an example of an identifier:
```json  
{  
   "use":"official",
   "type":{  
      "coding":[  
         {  
            "system":"http://www.patient-source-b.com/localid",
            "code":"PatientSourceB",
            "display":"Patient Source Domain B"
         }
      ],
      "text":"Patient Source Domain B"
   },
   "system":"urn:oid:2.16.840.1.113883.4.3.30",
   "value":"1000000001",
   "assigner":{  
      "display":"Patient Source Domain B"
   }
}
```
The EMPI links these two similar-enough source patient resources and generates a master patient resource with a master patient identifier:
```json
{  
   "use":"official",
   "type":{  
      "coding":[  
         {  
            "system":"http://www.patient-master.com/mpi",
            "code":"MPI",
            "display":"Patient Master Domain"
         }
      ],
      "text":"Patient Master Domain"
   },
   "system":"urn:oid:2.16.840.1.113883.4.3.10",
   "value":"1000000000",
   "assigner":{  
      "display":"Patient Master Domain"
   }
}
```

An FHIR system stores and manages master patient identifiers. A master patient identifier groups source patient resources that are similar enough together based on the similarity algorithms of the integrated EMPI. It is not required to create and persist a master patient resource statically.  The master patient resource can be built on the fly dynamically when the master patient information is retrieved. 

## 5. FHIR EMPI Operations/Transactions

FHIR EMPI interface uses the FHIR operation style to define EMPI operations. This document includes the following generic Patient EMPI transactions. FHIR EMPI operations execute on patient identification master domain and patient identification source domains, and common FHIR interactions are still applicable to these domains.  

Refer to FHIR documentation for audit and security of operations. 

### 5.1 Match Operation

The match operation “$match“ accepts a patient resource from one patient source domain and performs an intelligent search for possible matches in the system using the configured underlying EMPI. If it finds a match, it links the patient resource with the matches and updates the master patient resource. Otherwise, it creates a master patient resource, assigns a master patient identifier, and links the patient resource with the newly created master patient resource.   In the match case, the underlying EMPI determines if it replaces or updates the matched source patient resource from the same patient identification source domain. It also evaluates potential duplicates. The operation accepts match options. The process does not require a minimum set of patient information used for matching.   A distinct EMPI has different match options and resolves a collection of patient information necessary needed for the matching.  An EMPI also can standardize, normalize,  enrich, and validate the patient information. 

### 5.2 Update Operation

The update operation uses the FHIR update interaction to update an existing patient source resource or a current master patient resource. The FHIR EMPI Interface does not introduce a specific operation.  An FHIR system integrating with an EMPI performs an additional process for an update. The underlying EMPI determines how to update a current patient source resource or an existing master patient resource. Generally, updating a current patient source resource recalculates the master patient resource and reevaluates the potential duplicates based on the underlying policy and algorithms.

### 5.3 Search Operation

The search operation “$search” performs an intelligent search according to the user-specified search criteria. A return contains both of the master patient resources and patient source resources that matched the search criteria with the match scores. Primarily, a probabilistic search or an advanced similarity search applies to find any patient resources that are similar enough to the search criteria.  The underlying EMPI could support different kinds of search modes, for example, fuzzy search, block search, or phonetic search. The search allows for typographical errors and misspellings. 

The EMPI search operation is distinctive with FHIR search interaction; the FHIR search interaction does not trigger the EMPI search capability.  

### 5.4 Potential Operation

The potential operation “$potential” retrieves potentially duplicated patient resources on the search criteria. Usually, an EMPI generates a task and requires manual resolution for the possible duplicates through a data stewardship interface. Not all EMPIs provides an interface to fetch a list of likely duplicates.  

### 5.5 Merge Operation

The merge operation “$merge” merges two source patient identifiers or two master patient identifiers. Generally, merging goes within the same patient source domain or a patient master domain, but cannot go across the domains. When two source patient resources in a patient source domain are identified to be the same patient, the merge operation is used to merge a source patient identifier to a target patient identifier. After a merge, the subsumed patient identifier and patient resource are not accessible; deleting the subsumed patient identifier and patient resource is not required.  

For more detail, the merge operation supports two types of merge: source patient identifier unmerge, and master patient identifier unmerge. 

Merging two patient identifiers from one patient source domain involves the subsumed patient identifier and the surviving patient identifier. The EMPI determines how to unite two patient resources associated with the two patient identifiers. The EMPI needs to process two situations: one is that each patient resource is associated with a distinct master patient resource; the other is that both patient resources are associated with the same master patient resource. The expected outcome is to have one surviving master patient identifier and one surviving source patient identifiers.  The subsumed patient identifier shall no longer be referenced.  Searching a subsumed patient identifier shall get an empty return and include the surviving patient identifier. 

Merging two master patient identifiers from a patient master domain involves the subsumed master patient identifier and the surviving master patient identifier. The EMPI determines how to unite two master patient resources associated with the two master patient identifiers. The subsumed master patient identifier shall no longer be allowed.  Searching a subsumed master patient identifier shall get an empty return and include the surviving master patient identifier.

### 5.6 Unmerge Operation

The unmerge operation “$unmerge” unmerges two patient identifiers that are involved in the most recent merge operation and rolls back the merging transaction. After an unmerge, the subsumed patient identifier becomes active and allows to be referenced again. The unmerge operation supports two types of unmerging: source patient identifier unmerge, and master patient identifier unmerge. 

### 5.7 Link Operation

The link operation “$link” links one patient identifier to another one within one patient source domain or across two patient source domains. Linking two patient identifiers does not require the actual merging of two patient resources.  After a link, the associated patient resources are grouped with one patient master identifier, and the involved patient resource remain distinct. 

Before a link, two patient identifiers are not connected, and each of them associates with a distinct patient master identifier and represent a separate patient. Linking them merges two patient master identifiers and two patient master resources. The integrated EMPI resolves how to combine two patient master resource according to the policy and algorithms.  After a link, the subsumed patient master identifier is not allowed to be referenced.

### 5.8 Unlink Operation

The unlink operation “$unlink” unlink two patient identifiers that are involved in the most recent link operation and rolls back the linking transaction. After an unlink, the subsumed master patient identifier becomes active and allows to be used again.

### 5.9 Delete Operation

The delete operation uses the FHIR delete interaction to delete a patient resource or a patient master resource. The FHIR EMPI Interface does not introduce a specific operation. Removing a patient resource requires to update the patient master resource associated. A patient master resource will be deleted after the last patient resource of the related patient master identifier removes. The EMPI triggers to reevaluate the potential duplicates after a deletion. It is not necessary to remove patient resource permanently. Both soft deletion and hard deletion are acceptable. For healthcare information system best practice, the patient information deleted shall remain in the transaction log which is used for recovering in the future.

### 5.10 Undelete Operation

The undelete operation “$undelete” recovers the patient resource that eliminated in the delete transaction. The undelete process rolls back the delete transaction.

### 5.11 History Operation

The history operation uses the FHIR history interaction to retrieves the history of changes of a patient resource or a patient master resource.  The FHIR EMPI Interface does not introduce a specific operation.  It is suggested to support for fetching a change history of a deleted patient resource or a deleted patient master resource. 

### 5.12 Notification Operation

The notification operation notifies any external application that subscribes on any change of the system. A  notification resource delivers from the FHIR system to external applications that have registered the interest in such notifications. An implementation determines to enable synchronous mode, asynchronous mode, or both. The subscription resource is used to define a push-based subscription from a server to another system.  The notification is critical for synchronizing the whole integrated health information systems for an update on the patient resources to maintain the system consistency and integrity.

## 6. FHIR EMPI Operation and Message Definition

Click each of the following sections for detail:

**[6.1 Match Operation](/messages/match.md)**  
**[6.2 Update Operation](/messages/update.md)**  
**[6.3 Search Operation](/messages/search.md)**  
**[6.4 Potential Operation](messages/potential.md)**  
**[6.5 Merge Operation](/messages/merge.md)**  
**[6.6 Unmerge Operation](/messages/unmerge.md)**  
**[6.7 Link Operation](/messages/link.md)**  
**[6.8 Unlink Operation](/messages/unlink.md)**  
**[6.9 Delete Operation](/messages/delete.md)**  
**[6.10 Undelete Operation](/messages/undelete.md)**  
**[6.11 History Operation](/messages/history.md)**  
**[6.12 Notification Operation](/messages/notification.md)**  
