# Flow Design for PDF Card download using RID feature

## PDF Card Generation Flow :

<p align="center">
  <img  src="https://user-images.githubusercontent.com/9315119/161994052-4ee8eb98-38e2-4319-864b-72fea61febce.png">
</p>

1. Idrepo will publish a websub event when an identity is created or updated to IDENTITY_CHANGED topic. This event will contain the registration Id / application Id responsible for the change, uin and the event sub type (created / updated).
2. PDFCard service will subscribe to this IDENTITY_CHANGED topic and will receive the event. Then a request to credential request generator with this UIN and credential type as PDFCard is sent.
3. UIN received in the event is hashaed and the combination of UIN hash and request id received from the credential request generator response is stored in the database.
4. Credential request generator service will store the request in its database and the batch job will forward the request to credential service
5. Credentail service will create a encrypted verifiable credentail based on the configured policy for the UIN and will store it in datashare service
6. The datashare URL return from datashare service along with the request id is then published as a websub event to the PDF Card service specific topic in websub
7. Websub will deliver the event to the PDF card service 
8. PDF card service will use the datashare URL to download the encrypted verifiable credential, will decrypt the same and verify
9. PDF card service will use the verifiable credentail to create the password protected PDF file based on configured PDF template and will upload the same to datashare service without partner encryption. This template will mostly match the print template used by the country. The password used for pdf protection will be year of birth of the applicant.
10. The datashare PDF file URL received through upload is then saved into the DB for the same request id
11. The PDF file URL along with request id and status as STORED is published as a websub event to the CREDENTIAL_STATUS_UPDATE topic
12. Websub delivers the event to the credential request generator service and the status with datashare URL is updated to the database.

## Resident Service - PDF Card Download flow :

<p align="center">
  <img  src="https://user-images.githubusercontent.com/9315119/161994024-ecb4fbfe-73d6-4fa9-b00e-9314c65511e5.png">
</p>

1. Applicant will login to the resident UI, navigate to PDF card download page and will request for OTP using the "Generate OTP for RID" API of resident service
2. Resident service will call the idrepo idvid API to get the UIN of the RID, if RID is not found in idrepo then resident service will return to resident UI to try after few days message
3. If UIN is found, resident service will send request to IDA internal auth service to generate OTP for the UIN and respond to resident UI that OTP is generated successfully
4. Applicant once receiving the OTP, will provide the same in resident UI and the request with RID and OTP will be sent to the resident service to download the PDF card.
5. Resident service will call the idrepo idvid API to get the UIN of the RID
6. The internal auth request will be formed using the UIN and the OTP and will be sent to IDA for validation
7. Once authentication is succesful, request with RID will be sent to PDFCard service to retrieve the datashare URL of the PDF card file
8. PDFCard service will hash search in database to find the datashare URL to the requested RID. But this step is optional if the datashare URL can be formed using the RID and other details without connecting to database which can help with performance improvement. 
9. Request will be sent to datashare service to download the PDF card file
10. The PDF file will be streamed back to the resident UI to complete the flow

## Admin Service - PDF Card Download flow :

TBD
