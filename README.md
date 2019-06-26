## FHIR MPI Interface 
Version 1.0
28 June 2019

Fast River Technologies Inc.

## 1. Introduction
This document introduces a collection of FHIR MPI (Master Patient Index), or eMPI (Enterprise Master Patient Index), operations which extend FHIR R4 Patient match operation. The document defines the request of each MPI operation,  the response of each MPI operation, operation outcome details, and error conditions for the transactions of MPI operations.  Traditionally, the MPI interfaces are always vendor-specific, proprietary, and complicated. Integrating healthcare information systems with an MPI is still tricky and vendor locked. The FHIR MPI operations are aimed to overcome these challenges for the FHIR-based systems. 

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

### 5.4 Potential Duplicate

### 5.5 Merge

### 5.6 Unmerge

### 5.7 Link

### 5.8 Unlink

### 5.9 Delete

### 5.10 Undelete

### 5.11 History

### 5.12 Notification

## 6. FHIR MPI Operation and Message Definitions

