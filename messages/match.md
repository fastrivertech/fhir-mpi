# 6.1 Match Operation
## 6.1.1 Description

FHIR MPI match operation “$match” uses FHR R4 match operation. Clients use the “$match” operation to request an underlying MPI to match a patient resource. The process accepts a patient resource with a complete or a partial set of patient attributes.  The operation does not specify any specific MPI matching algorithm, nor a minimum set of information that must be provided to perform an MPI matching operation. Each MPI matching algorithm requires a different set of patient attributes, a different number of parameters, and the different meaning of each parameter. Implementations can define the match operation by specifying a  profile on the resource parameters, indicating a minimum set of information is mandatory for the match operation.  

For FHIR match operation definition refer to http://hl7.org/fhir/operationDefintion/Patient-match

## 6.1.2 Request Message
| URL | [BaseUrl]/Patient/$match |         
| HTTP Method | POST |
| Request Headers | |
| Request Body | Parameters Resource |

## 6.1.3 Response Message

## 6.1.4 OperationOutcome

## 6.1.5 Response Status

## 6.1.6 Examples
