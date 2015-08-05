# Verinvest Issuer API v1 (RC1)

#### ** NOTE ** This is a purely technical document and does not offer regulatory compliance recommendations.

### Authentication

For API users, each request must contain a authorization token in the `Authorization` HTTP header. The token must be generated for each request and signed with a 4096-bit private key (that matches the public key Verinvest has on record). 

The token format is JWT, using the RS256 signature algorithm. The JWT header should be:
```json
{
  "typ": "JWT",
  "alg": "RS256"
}
```

The fields required for the body of the token are as follows:

Field Name | Description
--- | ---
issuerKey | Your issuer account key.
time | The unix time of the request (seconds).
method | The request method (GET, POST, PATCH, etc...)
url | The path of the request relative to the domain, including query parameters.

An example token body is:
```json
{
  "issuerKey": "8djfow=",
  "time": 1438111640,
  "method": "GET",
  "url": "/issuer/v1/investments",
}
```

The token should be serialized and signed using your RSA private key as in the [JWT specification](www.jwt.io).

Verinvest will instruct you on how to generate an RSA key pair, and request your public key. In addition to safely guarding your private key, Verinvest recommends private key password protection. Your private key can be used to access sensitive investor information and care must be taken to safeguard its use/access.

If at any point you believe your private key has been jeopardized, contact Verinvest immediately.

### Example Workflow

- 1) Create an Investment
- 2) Add Investor 1
- 3) Add Investor 2
- 4) Request Veripass for Investor 1
- 5) Request Veripass for Investor 2
- 6) Review Veripass and proceed with transaction, or perform additional verification.


### Testing

Verinvest will issue you a testing issuer key and corresponding private key that can be used for integration development. All endpoints are available but please note that all Veripass requests is test mode will be marked as invalid.



## API Overview  

### Investments Collection 
`/api/issuer/v1/investments`

#### GET - List investments.

##### Request Parameters

Name | Description
--- | --- 
skip | (optional) The number of results to skip. 

##### Response Body
An array of [Investment Resources](#investment-resource)

#### POST - Create an investment.

##### Request Body
Name | Description
--- | ---
name | The name of the investment.

##### Response Body
An [Investment Resource](#investment-resource)

### Investment Resource 
`/api/issuer/v1/investments/:key`

#### Example
```javascript
{
  key: '[invement key]'
  name: 'Company A Seed Round'
}
```

#### GET - Read an investment

##### Response Body
An [Investment Resource](#investment-resource)


### Investor Collection 
`/api/issuer/v1/investments/:key/investors`

#### GET - List investors.

##### Request Parameters
Name | Description
--- | ---
offset | (optional) The number of results to skip.
showDeleted | (optional) true/false Whether or not to show deleted investors.

##### Response Body
An array of [Investor Resources](#investor-resource)


#### POST - Add an investor.

Add an investor to an investment. 

Please note that once an investor is added, his/her information cannot be changed. The investor can be deleted and a new investor can be added in their place.

##### Request Body Example
```javascript
{
  "verinvestId": "9D4F59B" // optional, the verinvest ID provided by the investor
  "name": {
    "first": "John",
    "middle": "Allen", // optional
    "last": "Smith"
  },
  "dob": "1977-02-29",
  "ssn": "000-43-9233",
  "driversLicense": {
    "number": "213444",
    "state": "MT"
  },
  "domicile": {
    "address1": "12 Circle Street",
    "address2": "Apt 1B", // optional
    "city": "Smallville",
    "state": "MT",
    "zip": "93029" // 9 digit code OK
  },
  "phone": {
    "home" : "614-555-4385",
    "work" : "614-555-4386"
  },
  "account": { 
    "aba": "xxxxxxxxx",
    "number": "xxxxxxxxxxxxxxxxx"
  }
}
```  


### Investor Resource 
`/api/issuer/v1/investments/:key/investors/:key`

#### Example
```javascript
{
  "key": "[investor key]",
  "investmentKey": "[investment key]",
  "verinvestId": "9D4F59B" // optional, the verinvest ID provided by the investor
   "name": {
    "first": "John",
    "middle": "Allen",
    "last": "Smith"
  },
  "dob": "1977-02-29",
  "ssn": "000-43-9233",
  "driversLicense": {
    "number": "213444",
    "state": "MT"
  },
  "domicile": {
    "address1": "12 Circle Street",
    "address2": "Apt 1B",
    "city": "Smallville",
    "state": "MT",
    "zip": "93029"
  },
  "phone": {
    "home" : "614-555-4385",
    "work" : "614-555-4386"
  },
  "account": { 
    "aba": "xxxxxxxxx",
    "number": "xxxxxxxxxxxxxxxxx"
  }
}

```  

#### GET - Read an Investor

##### Response Body
An [Investor Resource](#investor-resource)

#### DELETE - Delete an Investor

This method marks the investor as invalid and removes them from the investment. Verinvest maintains records of deleted investors but they, and their Veripasses, will (by default) no longer appear in the corresponding investor collection.


### Veripass Resource
`/api/issuer/v1/investments/:key/investors/:key/veripass`

#### Example
```javascript
{
  "key": "[veripass key]", // Use for reference.
  "investmentKey": "[investment key]",
  "investorKey": "[investor key]",
  "status": true, // if false, see failure codes and failure reason codes.
  "failureCodes": [], // list of issuers as to why the investor could not be verified, see reason codes
  "failureReasonCodes": [] // list of reasons why there are failures.
}
```

#### GET - Read the Veripass
##### Response Body
A [Veripass Resource](#veripass-resource)

#### POST - Request a Veripass

##### Request Parameters (for test mode only)

Name | Description
--- | --- 
failureCode | (repeatable, optional) The failure codes to trigger.
reasonCode | (repeatable, optional) The reason codes to trigger (may also trigger failure codes).

##### Response Body
A [Veripass Resource](#veripass-resource)

#### Veripass Failure Codes
If the Veripass has a status value equal to `false`, one or more failure codes may exist that describe why the Veripass request was not successful.

Code | Description
--- | ---
0 | Information match failure.
1 | Identity verification failure.
2 | Address verification failure.
3 | Account verification failure.
4 | Possible watch list match.
5 | The application is in test mode.

#### Veripass Failure Reason Codes

Code | Description
--- | ---
0 | SSN owner deceased.
1 | SSN issued prior to the DOB.
2 | SSN does not match the input address.
3 | SSN invalid or not yet issued.
4 | Address invalid.
5 | Address is PO box.
6 | Address is not residential.
7 | Name, address, and SSN do not match.
8 | Address not verified.
9 | SSN not verified.
10 | DOB not verified.
11 | Possible OFAC match.
12 | Name not verified.
13 | SSN has multiple last names associated.
14 | SSN issued too recently.
15 | DLN invalid for DLN state.
16 | Name and SSN do not match.
17 | SSN not found.
18 | SSN does not match name or address.
19 | SSN belongs to non-US citizen.
20 | SSN issued too recently.
21 | SSN issued after age five (1990+).
22 | DLN did not match name/SSN/address.
23 | Investor may be deceased.
24 | DLN not found.
25 | Multiple possible identities.
26 | Address may be previous address.
27 | Possible non OFAC match.
28 | Identity theft alert.
29 | CRA corrections security freeze.
