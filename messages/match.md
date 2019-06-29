# 6.1 Match Operation
## 6.1.1 Description

FHIR EMPI match operation “$match” uses FHR R4 match operation. This document details the process.  Clients use the “$match” operation to request an underlying EMPI to match a patient resource. The method accepts a patient resource with a complete or a partial set of patient attributes.  The operation does not specify any specific MPI matching algorithm, nor a minimum set of information that must be provided to perform EMPI matching algorithms. Each EMPI matching algorithm requires a different set of patient attributes, a different number of parameters, and the different meaning of each parameter. Implementations can define the match operation by specifying a  profile on the resource parameters, indicating a minimum set of information is mandatory for the match operation.  An EMPI is still responsible for tuning the matching algorithms.

For FHIR match operation definition refer to http://hl7.org/fhir/operationDefintion/Patient-match

## 6.1.2 Request Message

| URL | [BaseUrl]/Patient/$match |
|:----|:-------------------------|
| Method | POST |
| Headers | |
| Body | Parameters Resource |

<table>
    <thead>
        <tr>
            <th>Layer 1</th>
            <th>Layer 2</th>
        </tr>
    </thead>
	<tbody>
        <tr>
            <td>L3 Name A</td>
            <td>L3 Name B</td>
	</tr>
    </tbody>	  
	<tbody>
        <tr>
            <td rowspan=4>L1 Name</td>
            <td>L3 Name A</td>
        </tr>
        <tr>
            <td>L3 Name B</td>
        </tr>
    </tbody>	
</table>

## Request Body 
| Name | Cardinality | Type | Description |
|:-----|:------------|:-----|:------------|
| Resource | 1..1 | Patient |  This parameter provides a set of mandatory patient detail for the MPI to match  |
| MatchOption | 0..1 | Boolean | This parameter is EMPI-specific and specifies logic for the match algorithm. An EMPI inteprets the meaning of this parameter, and also can introduce more match options. For Oracle OHMPI, MatchOption is matchUpdate. For FHIR R4 default, MatchOption is onlyCertainMatches | 

FHIR R4 $match operation specifies parameters onlyCertainMatches and count. Not everyone EMPI is mandatory to support them.  
       
## 6.1.3 Response Message

| Name | Cardinality | Type | Description |
|:-----|:------------|:-----|:------------|
| Return | 1..1 | Bundle Resource | A bundle contains one matched patient resource, a list of potential matches, an OperationOutcome which includes a match result code.  An OperationOutcome might be returned with a failure code if the operation is unsuccessful. |

An MPI determines the match logic whether more than one matches above the match threshold are considered. If an MPI matching algorithm finds more than one records above the match threshold,  the MPI might only consider one record with the highest matching score as math, and the rest flags as potential matches. An MPI might just think none of the records as a match and all flag potential matches.  A list of potential matches contained in the bundle is ordered by the descending match score. All patient records shall have a search score from 0 to 1, where 1 is the most certain match, along with an extension "match-grade" that indicates how likely the patient is to be matched.

Refer to https://www.hl7.org/fhir/extension-match-grade.html for match grade.

## 6.1.4 OperationOutcome

Following table lists error, warning, and information messages that provide detailed information about the outcome of the operation. Each MPI generates different information about the result of the operation. 

| Issue Severity | Issue Code | Issue Detail | Issue Diagnostics |
|:---------------|:-----------|:-------------|:------------------|
| Information | match-found | Match found | N/A |
| Information | match-not-found | Match not found, a new record | N/A |
| Information | patient-found-replaced | Existing patient found and replaced | N/A |
| Information | patient-found-updated | Existing patient found and updated | N/A |
| Error | invalid-resource | Input resource not valid | invalid-resource |
| Error | invalid-parameter | Input paramter not valid | invalid-parameter |
| Error | exception | match fails | exception |

## 6.1.5 Response Status

| Status | Description |
|:-------|:------------|
| 200 OK | The MPI match process succeeded. A match is found, or a list of potential matches.|
| 201 Created | The MPI match process succeeded. A match is not found, or with a list of potential matches, a new patient is created. |
| 400 Bad Request | The MPI failed to process the request either because of invalid resource or invalid parameters provided in the request. |
| 500 Internal Server Error | An unexpected error occurred. |


## 6.1.6 Examples

Request: 

```json
POST [BaseUr]/Patient/$match
{  
  "resourceType":"Parameters",
  "id":"example",
  "parameter":[  
    {  
      "name":"resource",
      "resource":{  
        "resourceType":"Patient",
        "identifier":[  
          {  
            "use":"usual",
            "type":{  
              "coding":[  
                {  
                  "system":"http://hl7.org/fhir/v2/0203",
                  "code":"MR"
                }
              ]
            },
            "system":"urn:oid:1.2.36.146.595.217.0.1",
            "value":"12345"
          }
        ],
        "name":[  
          {  
            "family":[  
              "Chalmers"
            ],
            "given":[  
              "Peter"
            ]
          }
        ],
        "gender":"male",
        "birthDate":"1974-12-25"
      }
    },
    {  
      "name":"matchOption",
      "valueBoolean":"true"
    }
  ]
}
```

Response:
```json
HTTP/1.1 200 OK
{  
  "resourceType":"Bundle",
  "id":"26419249-18b3-45de-b10e-dca0b2e72b",
  "meta":{  
    "lastUpdated":"2016-03-18T03:28:49Z"
  },
  "type":"searchset",
  "total":2,
  "entry":[  
    {  
      "fullUrl":"http://server/path/Patient/example",
      "resource":{  
        "resourceType":"Patient",
        "id":"example",
        "identifier":[  
          {  
            "use":"MPI",
            "system":"http://www.fastrivertech.com/fhir/mpi/euid",
            "value":"1000000001",
            "assigner":{  
              "display":"Fast River Technologies"
            }
          }
        ]
      },
      "search":{  
        "extension":[  
          {  
            "url":"http://hl7.org/fhir/StructureDefinition/match-grade",
            "valueCode":"certain"
          }
        ],
        "mode":"match",
        "score":0.9
      },
      "response":{  
        "status":"match-found",
        "outcome":{  
          "resourceType":"OperationOutcome",
          "id":"101",
          "issue":[  
            {  
              "severity":"informational",
              "code":"match-found",
              "details":{  
                "text":"match found"
              },
              "diagnostics":"N/A"
            }
          ]
        }
      }
    },
    {  
      "fullUrl":"http://server/path/Patient/292",
      "resource":{  
        "resourceType":"Patient",
        "id":"292",
        "identifier":[  
          {  
            "use":"MPI",
            "system":"http://www.fastrivertech.com/fhir/mpi/euid",
            "value":"1000000000",
            "assigner":{  
              "display":"Fast River Technologies"
            }
          }
        ]
      },
      "search":{  
        "extension":[  
          {  
            "url":"http://hl7.org/fhir/StructureDefinition/match-grade",
            "valueCode":"possible"
          }
        ],
        "mode":"match",
        "score":0.2
      },
      "response":{  
        "status":"possible"
      }
    }
  ]
}
```
