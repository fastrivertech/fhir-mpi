# FHIR MPI Interface
Version 1.0
28 June 2019
charles.ye@fastrivertech.com

Fast River Technologies 

## 1. Introduction
This document introduces a collection of FHIR MPI (Master Patient Index), or EMPI (Enterprise Master Patient Index), operations. FHIR R4 Patient match operation is  a part of it. The document defines the request of each MPI operation,  the response of each MPI operation, operation outcome details, and error conditions for the transactions of MPI operations.  Traditionally, the MPI interfaces are always vendor-specific, proprietary, and complicated. Integrating healthcare information systems with an MPI is still tricky and vendor locked. The FHIR MPI operations are aimed to overcome these challenges for the FHIR-based systems. 

## 2. Scope
This document describes the MPI functionality through defining FHIR MPI operations. It does not address the underlying implementation and integration of MPI functionality.

## 3. Assumptions
This document assumes FHIR Specification R4, which is the current officially released normative version at the time of this writing.

## 4. Integration FHIR with MPI
A Master Patient Index (MPI) or an Enterprise Master Patient Index (EMPI) provides a single source of truth about a patient within or across healthcare organization systems, ensuring the accuracy and consistency of the unified and trusted patient data.  An MPI generates a golden patient record or a master patient record from a set of similar-enough patient records and assigns a unique enterprise -level identifier by using matching, merging, and deduplicating.  These similar enough patient records are linked together and represent the same patient.  The patient data is used for matching is a subset of patient demographic information, including name, gender, date of birth, social security number, address, and contact information. 

One or more identifiers pinpoints a patient resource.  A namespace or system issues a unique identifier within the issuing system to a patient.  A namespace or a system also considers as a patient identifier domain. An MPI is a master patient identification domain, which is also named MPI domain in this document. The MPI domain assigns a unique enterprise-level identifier to each golden patient record or master patient record by the underlying MPI.  A set of similar-enough patients from distinct patient identifier domains links together with a unique enterprise-level identifier from the MPI domain to embody the same patient.

![Figure 4.1](images/figure-4.1.jpeg)

Figure 4.1 MPI Domain Integrating with Two Source Patient Identification Domains

In Figure 4.1, an FHIR system includes two source patient domains, which are Patient Identification Domain A and Patient Identification Domain B. Patient Identification Domain A source and Patient Identification Domain B source ingest patients into the FHIR system through standard FHIR operations and FHIR MPI operations.  An MPI generates and manages the Patient Identification Master Domain through integrating with the FHIR system.

The Patient Identification Domain A issues a patient with its following local identifier: 
```json
{ 
      "use" : "official",
      "type" : "source",
      "system" : "http://www.domaina.com/fhir/mpi/localid",
      "value" : "1000000001",
      "assigner" : {
                      "display" : "Patient Identification Domain A" 
                   }
}
```  
The Patient Identification Domain B issues a patient with its following local identifier: 
```json
{ 
      "use" : "official",
      "type" : "source",
      "system" : "http://www.domainb.com/fhir/mpi/localid",
      "value" : "1000000001",
      "assigner" : {
                   	"display" : "Patient Identification Domain B" 
                   }
}
```
An MPI links these two similar-enough patient records and creates a master patient record with the following master patient identifier:
```json  
{ 
      "use" : "official",
      "type" : "mpi",
      "system" : "http://www.masterdomain.com/fhir/mpi/euid",
      "value" : "1000000000",
      "assigner": {
                     "display" : "Patient Mater Identification Domain" 
                  }
}
```
An FHIR system demands to manage master patient identifiers. A master patient identifier bonds patient records that are similar enough together. An implementation determines whether an FHIR system maintains a master patient record statically or dynamically.  The underlying MPI determines how two or more patient records are similar enough to embody the same patient using configurable match algorithms. 

## 5. FHIR MPI Operations/Transactions

FHIR MPI interface uses the FHIR operation style that defines MPI operations. This document includes the following generic Patient MPI transactions. FHIR MPI operations execute on master patient identification domain and patient identification source domains, and common FHIR interactions are still applicable to these domains.  Refer to FHIR documentation for audit and security of operations.   

### 5.1 Match

The match operation “$match“ accepts a patient resource from one patient identification domain and performs an intelligent search for possible matches in the system using the configured underlying MPI. If it finds a match, it links the patient resource with the matches and updates the master patient resource. Otherwise, it creates a master patient resource and assigns a master patient index identifier. In the match case, the underlying MPI determines if it replaces or updates the matched record from the same patient identification domain. It also evaluates potential duplicates.

### 5.2 Update

The update operation uses the FHIR update interaction to update an existing patient source resource or a current master patient resource by distinguishing the identifier of the input patient resource. The underlying MPI determines how to update a current patient source resource or an existing master patient resource. Generally, updating a current patient source resource recalculates the master patient resource based on the underlying algorithm configuration.

### 5.3 Search

The search operation “$search” performs an intelligent search according to the user-specified search criteria. A return contains the master patient resources and patient source resources that matched the search criteria with the match scores. Primarily, a probabilistic search or an advanced similarity search applies to find any patient resources that are similar enough to the search criteria.  The underlying MPI could support different kinds of search modes, for example, fuzzy search or phonetic search.

### 5.4 Potential

The potential duplicate operation “$potential” retrieves potentially duplicated patient resources on the search criteria. Usually, an MPI requires manual resolution for the potential duplicates.

### 5.5 Merge

The merge operation “$merge” merges two patient identifiers. Two patient identifiers can be either within a Patient Identification Source Domain or within a Master Patient Identification Domain, but cannot mix across the domains. When two patient resources in a patient identification domain are identified to be the same patient, the merge operation is used to merge a source patient identifier to a target patient identifier in the domain. After a merge, the subsumed patient identifier is not accessible; deleting the subsumed patient identifier is not required.  

For details, the merge operation performs two types of merge: Source Domain merge and Master Domain merge.  Merging two patient identifiers from a Patient Identification Source Domain involves the subsumed patient identifier and the surviving patient identifier. The MPI determines how to unite two patient resources associated with the two patient identifiers. The MPI needs to process two situations: one is that each patient resource is associated with a distinct master patient resource; the other is that both patient resources are associated with the same master patient resource. The expected outcome is to have one master patient identifier and one surviving patient identifiers.  The subsumed patient identifier and master patient identifier shall no longer be referenced.  Searching a subsumed patient identifier shall get an empty return and include the surviving patient identifier.

Merging two master patient identifiers from a Patient Identification Master Domain involves the subsumed master patient identifier and the surviving master patient identifier. The MPI determines how to unite two master patient resources associated with the two master patient identifiers. The subsumed master patient identifier shall no longer be referenced.  Searching a subsumed master patient identifier shall get an empty return and include the surviving master patient identifier.

### 5.6 Unmerge

The unmerge operation “$unmerge” unmerges two patient identifiers that are involved in the most recent merge operation and rolls back the merging transaction. After an unmerge, the subsumed patient identifier becomes active again. Further, the unmerge operation supports two types of unmerge: Source Domain unmerge, and Master Domain unmerge.

### 5.7 Link

The link operation “$link” links one patient identifier to another one within one Source Patient Domain or across two Source Patient Domains. Linking two patient identifiers does not require the actual merging of two patient resources.  After a link, one master patient identifier associates with the linked patient resources and the involved patient records remain distinct. In the case two patient identifiers are not linked and each of them associates with a distinct master patient identifier before a link, linking them merges two master patient identifiers. The MPI determines how to merge two master patient resources based on the configurable algorithms.  

### 5.8 Unlink

The unlink operation “$unlink” unlink two patient identifiers that are involved in the most recent link operation and rolls back the linking transaction. After an unlink, the subsumed master patient identifier becomes active again.

### 5.9 Delete

The delete operation uses the FHIR delete interaction to delete a patient resource or a master patient resource. Deleting a patient resource updates the master patient resource that associated with. It is not required to permanently remove a patient resource. 

### 5.10 Undelete

The undelete operation “$undelete” recovers the patient resource that deleted in the delete transaction. It rolls back the delete transaction.

### 5.11 History

The history operation uses the FHIR history interaction to retrieves the history of  a patient resource or a master patient resource. 

### 5.12 Notification

The notification operation notifies any application that subscribes on any change on the system. 

## 6. FHIR MPI Operation and Message Definitions

### 6.1 [Match](/messages/match.md)

### 6.2 Update
TBD

### 6.3 Search
TBD

### 6.4 Potential
TBD

### 6.5 Merge
TBD

### 6.6 Unmerge
TBD

### 6.7 Link
TBD

### 6.8 Unlink
TBD

### 6.9 Delete
TBD

### 6.10 Undelete
TBD

### 6.11 History
TBD

### 6.12 Notification
TBD



