# Warp Exchange Gateway
Documentation created on 10/08/2018 with the aim of exposing new functionalities to customers that are already integrated and to new customers.

   - Version: 1.2.1


# Two-Step Service Overview

   ### 1. Add Warp Exchange API calls to your payment page.
     - Request the address for a payment.
     - Confirm that the payment has been made.
### 2. Set up an address on your web server so Warp Exchange can communicate with you.
     - Any change of Status in the Warp Exchange will be informed to the client, so that it always has the same Status for all transactions.
# Main Endpoint

  - https://api.warpexchange.com (All methods below will use this endpoint)
   - To use the API, you will need to register a Callback URL. Through this URL, we will inform you of updates to your transactions. (see the final part of this document)
   - All calls will be through the POST method to the addresses.
   - All requests require the Header:
```sh
Example:
Key: Content-Type
Value: application / json
(I.e.
   - The API key provided by the Warp Exchange in the site must be used in all requests through the Header authorizationToken:
   sh
Example:
Key: authorizationToken
Value: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9 ... P0LVCem91odPn3MaBxYf4leokkxA
```


### / getnewaddress- Method responsible for returning a new address from the value and currency entered in your request.- This is the method that should be called after choosing the final price on the merchant site. (Pay with visa, master or bitcoin? -> click on bitcoin).- This method should be called every 5 (five) minutes in case of permanent display on the customer's screen. This is required.- Whenever a customer pays the lowest value, it will be sent to a Warp Exchange bitcoins portfolio, which will be ready for the end customer, through the store that made the purchase, to send the address to "refund".
##### New Functionality:
- Payment Split: The "split" field was added to the method. This field requests a "MarketPlaceToken" and a percentage, "PercentOf". The MarketPlaceToken, refers to the receiver and the percentage, how much will you receive in relation to that sale.
To issue this MarketPlaceToken, the MarketPlace partner and yourself must access the logged in area of the site and click: Profile> MarketPlace Integration, and click on "New Market Place Token".
MarketPlace partners are obligatorily Warp Exchange customers, which have already been registered and have valid documents. These may register as an Individual, however, they may only be linked to a MarketPlace, they will not be able to sell on their behalf due to legal issues involved.

###### Comments:- This field is not the same as the "Token" field generated for API use. It is extremely important that you do not share the API Token, only the MarketPlaceToken.
- This field is not mandatory, so there´s no need to inform, even without 
values.
- You must send the percentage applied. That is, the sum of the percentages should be 100.00, otherwise a message will be returned with the error made. The purpose of the obligation is to make it impossible for the customer to pass less or greater than what he really wanted for a possible calculation / programming mistake.
- Transactions carried out via MarketPlace will be displayed in a specific report and not with sales made without such functionality. 
To access them, go to the logged in area of the site and click on: Reports> Bitcoin (MarketPlace). Only payments related to the client will appear, not the percentage of those involved. We will soon provide MarketPlace with an area to track partner payments and status. It is important that he understands that the Partner is a Warp Exchange customer, and that he can deal directly with our support, should he/she need it.

Request Example:
  ```sh

Header
Key: authorizationToken
Value: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...P0LVCem91odPn3MaBxYf4leokkxA

Body
{
  "network": "BTC", /*  Use TESTNET for testing purposes with Testnet.*/
  "valueInLocalCurrency": 2000.45, /*  Fiat Currency value of the purchase. */
  "merchantSystemID": "T3qwuhdq2148T2", /* Client system identifier */
   "split": [
                      { "MarketPlaceToken": '....iIsInR5cCI6Ikp....', "PercentOf": 10.00 },/*Comissão de vendas à um parceiro*/
                      { "MarketPlaceToken": '....odPn3MaBxYf4le....', "PercentOf": 20.00 },/*Cliente dono do MarketPlace*/
                      { "MarketPlaceToken": '....Pn3MaBnR5cCI....', "PercentOf": 70.00 }/*Vendedor/Quem entrega o produto*/
            ]
}
```
Answer Example:
  ```sh
{
    "statusCode": 200, //HTTP Status - 200 == ok / 500 == Internal Server Error.
    "body": {
        "walletAddress": "2Mzyn1FGC...........AfkHiA",
        "valueInSatoshi": 533543
    }
}
```
For ease of viewing, use google's QR code API:
<img src="https://chart.googleapis.com/chart?chs=250x250&cht=qr&chl=bitcoin:' + result.body.walletAddress + '&amount=' + result.body.valueInSatoshi+'"/>

### / gettransactioninformation
   - Method that returns the details of a transaction, being informed the system ID of the merchant.
   - In this version of API it is still necessary to inform the API key besides the Header in Body. This will change in next versions.

Request Example:
  ```sh

Header
Key: authorizationToken
Value: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...P0LVCem91odPn3MaBxYf4leokkxA

Body
{
  "merchantSystemID": "T3UQWHD12148T2"
}
```
Answer Example:
  ```sh
{
    "statusCode": 200, //HTTP Status - 200 == ok / 500 == Internal Server Error.
    "body": {
        "TipoMoeda": "BTC",
        "MerchantSystemID": "T3qwuhdq2148T2",
        "EnderecoDoCliente": null,
        "DataRecebimento": null,
        "ValorSolicitadoEmMoedaLocal": "200045", /*Multiply by 100.*/
        "Notificado": false, /* If we have already notified the merchant at your registered callback address */
        "TarifaMineracaoEnviada": null,
        "HashDaTransacao": null,
        "ValorRecebidoEmDigital": null, /* Must be the same as "ValorSolicitadoEmDigital" */
        "ValorSolicitadoEmDigital": "7613332",
        "StatusID": 1,
        "Status": "Requested Address"
    }
}
```
### / gettransactions 
- Method that returns all transactions and their respective statuses. 
- This method will change soon so that the desired paging is sent.

Request Example:
  ```sh

Header
Key: authorizationToken
Value: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...P0LVCem91odPn3MaBxYf4leokkxA

Body
{}
```
Answer Example:
  ```sh
  {
    "statusCode": 200, //HTTP Status - 200 == ok / 500 == Internal Server Error.
    "body": {
        [
             {
                "TipoMoeda": "TESTNET",
                "MerchantSystemID": "QWIUDH1218928812HDKASJ",
                "EnderecoDoCliente": "2NFtZ52w...p78SN8aT6rBX",
                "DataRecebimento": "2018-08-15T12:54:59.350Z",
                "ValorSolicitadoEmMoedaLocal": "33300",
                "Notificado": true,
                "TarifaMineracaoEnviada": "3660",
                "HashDaTransacao": "0370901047435fad4bfc708...5b9b5ed2b11d9163faa2",
                "ValorRecebidoEmDigital": "100000000",
                "ValorSolicitadoEmDigital": "100000000",
                "StatusID": 9,
                "Status": "Confirmed on Warp Wallet" /* You can now deliver the product */
            },
            {
                "TipoMoeda": "BTC",
                "MerchantSystemID": null,
                "EnderecoDoCliente": null,
                "DataRecebimento": null,
                "ValorSolicitadoEmMoedaLocal": "0",
                "Notificado": false,
                "TarifaMineracaoEnviada": null,
                "HashDaTransacao": null,
                "ValorRecebidoEmDigital": null,
                "ValorSolicitadoEmDigital": "0",
                "StatusID": 11,
                "Status": "Payment not made" /* Nothing was sent during the 5 minute window */
            }
        ]
  }
```
 

Example of C # callback method to receive the updates:
  ```sh
        [HttpPost]
        [AllowAnonymous]
        public string MerchantTransactions(FormCollection collect)
        {
            string token = this.HttpContext.Request.Headers["authorizationToken"];
            if (token == "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...P0LVCem91odPn3MaBxYf4leokkxA")
            {
                //Transação válida, entretanto, consulte nossa API, no método GetTransactionInformation
                if(GetTransactionInformation().Status != collect["StatusID"])
                return "Error - Transaction does not contain the same status as the one entered in the descriptive method. Flag this case for Warp Exchange Support"
                //
                string WEResp = "TID: " + collect["TID"];
                WEResp += " StatusID: " + collect["StatusID"];
                WEResp += " StatusMessage: " + collect["StatusMessage"];
                WEResp += " MerchantTokenToUseInCallbacks: " + collect["MerchantTokenToUseInCallbacks"];

                switch (collect["StatusID"])
                {
                    case "2":
                        // Save to your store Database:
                         // Shipment Done. Waiting confirmation.
                         // The customer paid, but we need to wait 6 confirmations                              to not consider fraud.
                        break;
                    case "9":
                       // 
                       //Payment confirmed. You can already deliver the product.
                        break;
                    case "6":
                        //Save to your store Database:
                        //Payment in cash in your realized account. You can already check your balance.
                        break;
                    case "12":
                        //Save to your store Database:
                        //Amount paid was less than required. Inform your client that if you wish, you can get your money back, informing us by email of the details of the mistake made.
                        break;
                }
                
                return WEResp;
            }
            return "Error - Wrong Status or Token. Token : " + this.HttpContext.Request.Headers["authorizationToken"] + " Status: " + collect["StatusID"];   
        }
```

- List of possible status for receiving payments:
```sh
ID Status
1 Payment Requested
2 Awaiting Confirmation on Warp
6 Payment by Warp to Customer
9 Confirmed at Warp
11 Payment not carried out
12 Payment with less value *
13 Paid with greater value - We will pay you and you choose how to revert to the customer
14 Reversed to customer
```
* Amounts paid but with a lower value than requested will be sent to a separate wallet and will wait for a refund request within 30 days. After this deadline, the amount will be sent to a charity.
