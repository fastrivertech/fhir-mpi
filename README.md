## FHIR MPI Interface 
Version 1.0
28 June 2019

Fast River Technologies Inc.

## Introduction
This document introduces a collection of FHIR MPI (Master Patient Index), or eMPI (Enterprise Master Patient Index), operations which extend FHIR R4 Patient match operation. The document defines the request of each MPI operation,  the response of each MPI operation, operation outcome details, and error conditions for the transactions of MPI operations.  Traditionally, the MPI interfaces are always vendor-specific, proprietary, and complicated. Integrating healthcare information systems with an MPI is still tricky and vendor locked. The FHIR MPI operations are aimed to overcome these challenges for the FHIR-based systems. 

## Scope
This document describes the MPI functionality through defining FHIR MPI operations. It does not address the underlying implementation and integration of MPI functionality.

## Assumptions
This document assumes FHIR Specification R4, which is the current officially released normative version at the time of this writing. 
