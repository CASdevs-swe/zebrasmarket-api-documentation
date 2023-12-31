
# Zebrasmarket Api Documentation
Api documentation for partners using [Zebrasmarket.com](https://zebrasmarket.com/).

## Introduction

Prefix API endpoints with `https://zebrasmarket.com/` to separate them from other URLs like HTML views served on the same server.

### Signature


Table of Contents
=================

   * [API ](#api-manifesto)
      * [Introduction](#introduction)
        * [Signature](#signature)
   * [Table of Contents](#table-of-contents)
   * [Endpoints](#endpoints)
   
   
# Api versions

## On-site integration

Use this integration if you want to integrate zebrasmarket directly into your site.

In order to use this integration our team must have approved your design. It's essential that some things are clearly visible on your design. 

- The entity rendition (see [Entity rendition](#entity-rendition))
- The value and random string
- The hash that makes up the entity

### Entity rendition

You can render the entity on your site through an iFrame:

```
https://zebrasmarket.com/entity/index.html?story={story}&value={value}
```
The `{story}` is the unhashed random string that the user can either input or be randomly generated by you, the partner.
The `{value}` is the value of the entity / purchase in cents.  

### Generating

Prefix API endpoints with `/api/` to separate them from other URLs like HTML views served on the same server.


## Zebrasmarket redirect integration

Use this integration if you want to use [Zebrasmarket.com](https://zebrasmarket.com/) as your platform for handling requests. 

All requests should start with `/partner/{partnerId}/`, where the `{partnerId}` is handed out by a Zebrasmarket admin. 

```http
POST /deposit_url
```

Body      | Description                                                                  | Type
--------- | ---------------------------------------------------------------------------- | ----
signature | Read the integration details about signature | string |
userId | ID of the user | string |
value | value in cents | int |
story | the text used to hash | string |

Example response:
```json
{
  "value": [
    { "street": "1st Avenue", "city": "Seattle" },
    { "street": "124th Ave NE", "city": "Redmond" }
  ]
}
```



## Example code

<details>
<summary>Click to see example code in JavaScript</summary>

###
*Note* Something

```javascript
const axios = require("axios")
const crypto = require("crypto")

/*

FLOW

Generate deposit url and give the url to the user.
Pair userId and orderId

wait for IPN

Check if signatures match
Credit the user paired with the orderId

*/

// Generates a SHA256 hash

const sha256 = (string) => {
    return crypto.createHash('sha256').update(string).digest('hex');
}

// Takes the data you put in and generates a signature

const buildSignature = (data, secret) => {
    let signatureString = "";

    Object.keys(data).sort().forEach((key) => {
        if (key === "signature") return;
        if (typeof data[key] === "object") return;
        signatureString += data[key];
    })
    return sha256(`${signatureString}${secret}`);
}

// API call to Zebrasmarket to generate a deposit URL

const createDepositUrl = async (userId) => {

    try {

    // You only need to specify the userId aka some type of user identifier

    const dataToEncrypt = {
        userId
    }

    // Replace apikey with your own api key

    const signature = buildSignature(dataToEncrypt, "apikey")
    const dataToSend = {
        ...dataToEncrypt,
        signature
    }

    // Replace partner id with your partnerId

    const tradeDataRes = await axios.post(`https://api.zebrasmarket.com/partner/${"partner id"}/trade_url`, dataToSend)

    if (tradeDataRes?.data?.error) return {
        error: true,
        msg:"Error generating deposit url"
    }

    const tradeData = tradeDataRes?.data;
    const orderId = tradeData.data.orderId 

    // Insert orderId to your database. Pair the orderId with the userId you sent to zebrasmarket.
    // Credit the user paired with the orderid when IPN call comes

    return {
        error: false,
        msg: "Deposit url created",
        data: {
            url: tradeData.data.url
        }
    }

    } catch (error) {

        return {
            error : true,
            msg:"Error generating deposit url"
        }

    }

}

// IPN 
/* 

Use the same buildSignature as above to verify the whole req.body

*/

const ipnHandler = (req, res) => {

    // Insert own logic for handling requst...

    const ipnSignature = buildSignature(req.body, "apikey")
    const sentSignature = req.body.signature;

    if (ipnSignature != sentSignature) return {
        error : true,
        msg: "Signatures does not match!"
    }

    const orderId = req.body.orderId;
    const value = data.value // Value is sent in USD * 100. $1 = 100

    // Check if user is already credited - your own logic. IPN calls can be sent more than once.

    // Credit the userId paired with the orderId. Only credit deposits with orderIds that are previously known by you

    // This must be sent back to zebrasmarket
    return res.status(200).send({
        success: true
    })

}
```

</details>