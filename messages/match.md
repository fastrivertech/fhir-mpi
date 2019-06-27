# 6.1 Match Operation
## 6.1.1 Description

FHIR MPI match operation “$match” uses FHR R4 match operation. Clients use the “$match” operation to request an underlying MPI to match a patient resource. The process accepts a patient resource with a complete or a partial set of patient attributes.  The operation does not specify any specific MPI matching algorithm, nor a minimum set of information that must be provided to perform an MPI matching operation. Each MPI matching algorithm requires a different set of patient attributes, a different number of parameters, and the different meaning of each parameter. Implementations can define the match operation by specifying a  profile on the resource parameters, indicating a minimum set of information is mandatory for the match operation.  

For FHIR match operation definition refer to http://hl7.org/fhir/operationDefintion/Patient-match

## 6.1.2 Request Message

| URL | [BaseUrl]/Patient/$match |
|:----|:-------------------------|
| HTTP Method | POST |
| Request Headers | |
| Request Body | Parameters Resource |

| Name | Cardinality | Type | Description |
|:-----|:------------|:-----|:------------|
| Resource | 1..1 | Patient |  This parameter provides a set of mandatory patient detail for the MPI to match  |
| MatchOption | 0..1 | Boolean | This parameter is EMPI-specific and specifies logic for the match algorithm. An EMPI inteprets the meaning of this parameter, and also can introduce more match options. For Oracle OHMPI, MatchOption is matchUpdate. For FHIR R4 default, MatchOption is onlyCertainMatches | 

FHIR R4 $match operation specifies parameters onlyCertainMatches and count. Not everyone EMPI is mandatory to support them.  
       
## 6.1.3 Response Message

| Name | Cardinality | Type | Description |
|:-----|:------------|:-----|:------------|
| Return | 1..1 | Bundle Resource | A bundle contains one matched patient resource, a list of potential matches, an OperationOutcome which includes a match result code.  An OperationOutcome might be returned with a failure code if the operation is unsuccessful. |



## 6.1.4 OperationOutcome

## 6.1.5 Response Status

## 6.1.6 Examples
