---
sidebar_position: 3
description: Learn how to fund user account, withdraw from user account, get transactions, get transaction by id and get transaction by reference
---

import Tabs from "@theme/Tabs";
import TabItem from "@theme/TabItem";

# Transactions APIs

## Fund user account

:::info

This request must have the authorization header. Refer to [Authorization method](/docs/guides/authentication#authentication-methods) guide for more details

:::

### Request

| Property     | Value                                 |
| :----------- | :------------------------------------ |
| method       | `POST`                                |
| url          | `$baseUrl/transactions/fund-user`     |
| Content-Type | `application/json`                    |

#### Body

| Property       | Type   | Required | Default | Description                             |
| -------------- | ------ | -------- | ------- | --------------------------------------- |
| amount         | number | Yes      | -       | Amount to be credited to user's account (must be >= 0.01) |
| user_id        | number | No       | -       | User Id                                 |
| user_reference | string | No       | -       | User Reference                          |
| description    | string | No       | "Balance funded" | Description for the transaction |

:::info

Either the `user_id` or `user_reference` must be provided

:::

<Tabs groupId="programming-language">
  <TabItem value="curl" label="cURL">

    ```bash
    curl -X POST "$baseUrl/transactions/fund-user" \
      -H "Content-Type: application/json" \
      -H "Authorization: Bearer $token" \
      -d '{
        "amount": 1000,
        "user_id": 1,
        "description": "Deposit via bank transfer"
      }'
    ```

  </TabItem>
  <TabItem value="javascript" label="JavaScript">

    ```javascript
    function fundUser(auth, amount, userId, description = "Balance funded") {
      fetch(`${baseUrl}/transactions/fund-user`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${auth.token}`
        },
        body: JSON.stringify({
          amount: amount,
          user_id: userId,
          description: description
        })
      })
      .then(response => response.json())
      .then(data => console.log(data))
      .catch(error => console.error('Error:', error));
    }

    // Example usage
    fundUser(auth, 1000, 1, "Deposit via bank transfer");
    ```

  </TabItem>
  <TabItem value="python" label="Python">

    ```python
    import requests
    import json

    def fund_user(auth, amount, user_id=None, user_reference=None, description="Balance funded"):
        url = f"{base_url}/transactions/fund-user"
        headers = {
            'Content-Type': 'application/json',
            'Authorization': f"Bearer {auth['token']}"
        }
        
        payload = {
            'amount': amount,
            'description': description
        }
        
        if user_id:
            payload['user_id'] = user_id
        elif user_reference:
            payload['user_reference'] = user_reference
        else:
            raise ValueError("Either user_id or user_reference must be provided")

        response = requests.post(url, headers=headers, data=json.dumps(payload))

        if response.status_code == 201:
            print(response.json())
            return response.json()
        else:
            print(f"Error: {response.status_code}, {response.text}")
            raise Exception(f"Funding failed: {response.text}")

    # Example usage
    try:
        fund_user(auth, 1000, user_id=1, description="Deposit via bank transfer")
    except Exception as e:
        print(f"An error occurred: {e}")
    ```

  </TabItem>
  <TabItem value="rust" label="Rust">

    ```rust
    use reqwest::Client;
    use serde_json::json;
    use std::error::Error;

    async fn fund_user(
        token: &str, 
        amount: f64, 
        user_id: Option<i32>, 
        user_reference: Option<&str>,
        description: Option<&str>
    ) -> Result<serde_json::Value, Box<dyn Error>> {
        let url = format!("{}/transactions/fund-user", base_url);
        let client = Client::new();
        
        let mut payload = json!({
            "amount": amount
        });
        
        if let Some(id) = user_id {
            payload["user_id"] = json!(id);
        } else if let Some(reference) = user_reference {
            payload["user_reference"] = json!(reference);
        } else {
            return Err("Either user_id or user_reference must be provided".into());
        }
        
        if let Some(desc) = description {
            payload["description"] = json!(desc);
        }

        let response = client.post(&url)
            .header("Content-Type", "application/json")
            .header("Authorization", format!("Bearer {}", token))
            .json(&payload)
            .send()
            .await?;

        if response.status().is_success() {
            let json_response = response.json::<serde_json::Value>().await?;
            println!("Funding successful: {:?}", json_response);
            Ok(json_response)
        } else {
            let error_text = response.text().await?;
            println!("Error: {}", error_text);
            Err(error_text.into())
        }
    }

    #[tokio::main]
    async fn main() -> Result<(), Box<dyn Error>> {
        let token = "your_token_here";
        
        // Example with user_id
        let result = fund_user(
            token, 
            1000.0, 
            Some(1), 
            None, 
            Some("Deposit via bank transfer")
        ).await?;
        
        Ok(())
    }
    ```

  </TabItem>
</Tabs>

### Response

<Tabs>
  <TabItem  value="Success">

    Http Code: `201`
    ```json
    {
        "data": {
            "id": 1,
            "user_id": 1,
            "account_id": 2,
            "reference": "TX_u6qGJ1A4qw0mV_5wMSJQ7dF5pe0S8HA3c4i1Z5YN7HD7loom",
            "amount": 1000,
            "description": "Balance funded",
            "transaction_type": "credit",
            "transaction_source": "funding",
            "created_at": "2025-05-09T14:57:43.759Z"
        },
        "message": "Account funded successfully"
    }
    ```

  </TabItem>
  <TabItem  value="User Not Found">

    Http Code: `404`
    ```json
    {
        "status": 404,
        "error": "User not found"
    }
    ```

  </TabItem>
  <TabItem  value="Invalid Amount">

    Http Code: `400`
    ```json
    {
        "message": [
            "amount must not be less than 0.01"
        ],
        "error": "Bad Request",
        "statusCode": 400
    }
    ```

  </TabItem>
  <TabItem  value="Missing Fields">

    Http Code: `400`
    ```json
    {
        "message": [
            "amount must not be less than 0.01",
            "amount must be a number conforming to the specified constraints",
            "user_id must not be less than 0.01",
            "user_id must be a number conforming to the specified constraints",
            "user_reference should not be empty",
            "user_reference must be a string"
        ],
        "error": "Bad Request",
        "statusCode": 400
    }
    ```

  </TabItem>
  <TabItem  value="Unauthorized">

    Http Code: `401`
    ```json
    {
        "message": "Unauthorized",
        "statusCode": 401
    }
    ```

  </TabItem>
</Tabs>

## Withdraw from user account

:::info

This request must have the authorization header. Refer to [Authorization method](/docs/guides/authentication#authentication-methods) guide for more details

:::

### Request

| Property     | Value                             |
| :----------- | :-------------------------------- |
| method       | `POST`                            |
| url          | `$baseUrl/transactions/withdraw`  |
| Content-Type | `application/json`                |

#### Body

| Property       | Type   | Required | Default | Description                               |
| -------------- | ------ | -------- | ------- | ----------------------------------------- |
| amount         | number | Yes      | -       | Amount to be debited from user's account (must be >= 0.01) |
| user_id        | number | No       | -       | User Id                                   |
| user_reference | string | No       | -       | User Reference                            |
| description    | string | No       | "Withdrawal" | Description for the transaction      |

:::info

Either the `user_id` or `user_reference` must be provided. The user must have sufficient balance for the withdrawal.

:::

<Tabs groupId="programming-language">
  <TabItem value="curl" label="cURL">

    ```bash
    curl -X POST "$baseUrl/transactions/withdraw" \
      -H "Content-Type: application/json" \
      -H "Authorization: Bearer $token" \
      -d '{
        "amount": 100.00,
        "user_id": 1,
        "description": "Bank withdrawal request"
      }'
    ```

  </TabItem>
  <TabItem value="javascript" label="JavaScript">

    ```javascript
    function withdrawUser(auth, amount, userId, description = "Withdrawal") {
      return fetch(`${baseUrl}/transactions/withdraw`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${auth.token}`
        },
        body: JSON.stringify({
          amount: amount,
          user_id: userId,
          description: description
        })
      })
      .then(response => {
        if (!response.ok) {
          return response.json().then(error => {
            throw new Error(error.error || `Error: ${response.status} ${response.statusText}`);
          });
        }
        return response.json();
      })
      .then(data => {
        console.log("Withdrawal successful:", data);
        return data;
      })
      .catch(error => {
        console.error('Error withdrawing funds:', error);
        throw error;
      });
    }

    // Example usage
    withdrawUser(auth, 100.00, 1, "Bank withdrawal request")
      .then(data => {
        // Handle successful withdrawal
      })
      .catch(error => {
        // Handle errors such as insufficient balance
      });
    ```

  </TabItem>
  <TabItem value="python" label="Python">

    ```python
    import requests
    import json

    def withdraw_user(auth, amount, user_id=None, user_reference=None, description="Withdrawal"):
        url = f"{base_url}/transactions/withdraw"
        headers = {
            'Content-Type': 'application/json',
            'Authorization': f"Bearer {auth['token']}"
        }
        
        payload = {
            'amount': amount,
            'description': description
        }
        
        if user_id:
            payload['user_id'] = user_id
        elif user_reference:
            payload['user_reference'] = user_reference
        else:
            raise ValueError("Either user_id or user_reference must be provided")

        response = requests.post(url, headers=headers, data=json.dumps(payload))

        if response.status_code == 201:
            result = response.json()
            print('Withdrawal successful:', result)
            return result
        else:
            error_data = response.json()
            error_message = error_data.get('error', f"Error: {response.status_code}")
            print(f"Error: {response.status_code}, {error_message}")
            
            if "balance" in error_data.get('data', {}):
                print(f"Current balance: {error_data['data']['balance']}")
                
            raise Exception(f"Withdrawal failed: {error_message}")

    # Example usage
    try:
        withdraw_user(auth, 100.00, user_id=1, description="Bank withdrawal request")
    except Exception as e:
        print(f"An error occurred: {e}")
    ```

  </TabItem>
  <TabItem value="rust" label="Rust">

    ```rust
    use reqwest::Client;
    use serde_json::{json, Value};
    use std::error::Error;

    async fn withdraw_user(
        token: &str, 
        amount: f64, 
        user_id: Option<i32>, 
        user_reference: Option<&str>,
        description: Option<&str>
    ) -> Result<Value, Box<dyn Error>> {
        let url = format!("{}/transactions/withdraw", base_url);
        let client = Client::new();
        
        let mut payload = json!({
            "amount": amount
        });
        
        if let Some(id) = user_id {
            payload["user_id"] = json!(id);
        } else if let Some(reference) = user_reference {
            payload["user_reference"] = json!(reference);
        } else {
            return Err("Either user_id or user_reference must be provided".into());
        }
        
        if let Some(desc) = description {
            payload["description"] = json!(desc);
        }

        let response = client.post(&url)
            .header("Content-Type", "application/json")
            .header("Authorization", format!("Bearer {}", token))
            .json(&payload)
            .send()
            .await?;

        if response.status().is_success() {
            let json_response = response.json::<Value>().await?;
            println!("Withdrawal successful: {:?}", json_response);
            Ok(json_response)
        } else {
            let error_resp = response.json::<Value>().await?;
            let error_msg = error_resp["error"].as_str().unwrap_or("Unknown error");
            
            if let Some(data) = error_resp.get("data") {
                if let Some(balance) = data.get("balance") {
                    println!("Current balance: {}", balance);
                }
            }
            
            Err(error_msg.into())
        }
    }

    #[tokio::main]
    async fn main() -> Result<(), Box<dyn Error>> {
        let token = "your_token_here";
        
        // Example with user_id
        match withdraw_user(token, 100.0, Some(1), None, Some("Bank withdrawal request")).await {
            Ok(result) => println!("Success: {:?}", result),
            Err(e) => println!("Error: {}", e),
        }
        
        Ok(())
    }
    ```

  </TabItem>
</Tabs>

### Response

<Tabs>
  <TabItem  value="Success">

    Http Code: `201`
    ```json
    {
        "data": {
            "id": 3,
            "user_id": 1,
            "account_id": 2,
            "reference": "TX_YXIDYBNaiABJVeXm-_mpnx8WsnWt8ANgwqyjqTd-Vi5WsuXS",
            "amount": 100,
            "description": "Withdrawal",
            "transaction_type": "debit",
            "transaction_source": "withdrawal",
            "created_at": "2025-05-09T15:02:37.463Z"
        },
        "message": "Account funded successfully"
    }
    ```

  </TabItem>
  <TabItem  value="Insufficient Balance">

    Http Code: `400`
    ```json
    {
        "status": 400,
        "error": "Not enough balance for withdrawal",
        "data": {
            "id": 1,
            "balance": 1900,
            "exposure": 0,
            "effective_balance": 1900
        }
    }
    ```

  </TabItem>
  <TabItem  value="User Not Found">

    Http Code: `404`
    ```json
    {
        "status": 404,
        "error": "User not found"
    }
    ```

  </TabItem>
  <TabItem  value="Invalid Amount">

    Http Code: `400`
    ```json
    {
        "message": [
            "amount must not be less than 0.01"
        ],
        "error": "Bad Request",
        "statusCode": 400
    }
    ```

  </TabItem>
  <TabItem  value="Missing Fields">

    Http Code: `400`
    ```json
    {
        "message": [
            "amount must not be less than 0.01",
            "amount must be a number conforming to the specified constraints",
            "user_id must not be less than 0.01",
            "user_id must be a number conforming to the specified constraints",
            "user_reference should not be empty",
            "user_reference must be a string"
        ],
        "error": "Bad Request",
        "statusCode": 400
    }
    ```

  </TabItem>
  <TabItem  value="Unauthorized">

    Http Code: `401`
    ```json
    {
        "message": "Unauthorized",
        "statusCode": 401
    }
    ```

  </TabItem>
</Tabs>

## Get transactions

:::info

This request must have the authorization header. Refer to [Authorization method](/docs/guides/authentication#authentication-methods) guide for more details

:::

### Request

| Property     | Value                       |
| :----------- | :-------------------------- |
| method       | `GET`                       |
| url          | `$baseUrl/transactions`     |
| Content-Type | `application/json`          |

#### Query Params

| Property           | Type                               | Required | Default | Description                     |
| ------------------ | ---------------------------------- | -------- | ------- | ------------------------------- |
| page               | number                             | No       | 1       | Current page                    |
| per_page           | number                             | No       | 20      | Number of items per page        |
| user_ids           | number(comma separated)            | No       | -       | Comma separated user ids        |
| user_references    | string(comma separated)            | No       | -       | Comma separated user references |
| transaction_type   | "credit" \| "debit" \| "no-action" | No       | -       | Transaction type                |
| transaction_source | "funding" \| "bet" \| "withdrawal" | No       | -       | Transaction source              |
| start_date         | number(timestamp) \| DateString    | No       | -       | Filter result from this date    |
| end_date           | number(timestamp) \| DateString    | No       | -       | Filter result to this date      |

:::info

`user_ids` and `user_references` can be used in combination. The results would be merged

:::

<Tabs groupId="programming-language">
  <TabItem value="curl" label="cURL">

    ```bash
    curl -G "$baseUrl/transactions" \
      -H "Content-Type: application/json" \
      -H "Authorization: Bearer $token" \
      --data-urlencode "page=1" \
      --data-urlencode "per_page=20" \
      --data-urlencode "user_ids=123,456" \
      --data-urlencode "user_references=user1,user2" \
      --data-urlencode "transaction_type=credit" \
      --data-urlencode "transaction_source=funding" \
      --data-urlencode "start_date=2023-01-01" \
      --data-urlencode "end_date=2023-12-31"
    ```

  </TabItem>
  <TabItem value="javascript" label="JavaScript">

    ```javascript
    async function fetchTransactions(auth, params) {
      const url = new URL(`${baseUrl}/transactions`);
      Object.keys(params).forEach(key => url.searchParams.append(key, params[key]));

      try {
        const response = await fetch(url, {
          method: 'GET',
          headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${auth.token}`
          }
        });

        if (!response.ok) {
          throw new Error(`Error: ${response.status} ${response.statusText}`);
        }

        const data = await response.json();
        console.log(data);
        return data;
      } catch (error) {
        console.error('Error:', error);
        throw error;
      }
    }

    // Example usage
    fetchTransactions(auth, {
      page: 1,
      per_page: 20,
      user_ids: '123,456',
      user_references: 'user1,user2',
      transaction_type: 'credit',
      transaction_source: 'funding',
      start_date: '2023-01-01',
      end_date: '2023-12-31'
    });
    ```

  </TabItem>
  <TabItem value="python" label="Python">

    ```python
    import requests

    def fetch_transactions(auth, params):
        url = f"{base_url}/transactions"
        headers = {
            'Content-Type': 'application/json',
            'Authorization': f"Bearer {auth['token']}"
        }

        try:
            response = requests.get(url, headers=headers, params=params)

            if response.status_code != 200:
                raise Exception(f"Error: {response.status_code} {response.text}")

            data = response.json()
            print(data)
            return data
        except Exception as error:
            print(f"Error: {error}")
            raise

    # Example usage
    fetch_transactions(auth, {
        'page': 1,
        'per_page': 20,
        'user_ids': '123,456',
        'user_references': 'user1,user2',
        'transaction_type': 'credit',
        'transaction_source': 'funding',
        'start_date': '2023-01-01',
        'end_date': '2023-12-31'
    })
    ```

  </TabItem>
  <TabItem value="rust" label="Rust">

    ```rust
    use reqwest::Client;
    use serde_json::Value;
    use std::collections::HashMap;
    use tokio;

    async fn fetch_transactions(token: &str, params: HashMap<&str, &str>) -> Result<Value, Box<dyn std::error::Error>> {
        let url = format!("{}/transactions", base_url);
        let client = Client::new();

        let response = client.get(&url)
            .header("Content-Type", "application/json")
            .header("Authorization", format!("Bearer {}", token))
            .query(&params)
            .send()
            .await?;

        if response.status().is_success() {
            let json = response.json::<Value>().await?;
            println!("{:?}", json);
            Ok(json)
        } else {
            let error_text = response.text().await?;
            eprintln!("Error: {}", error_text);
            Err(Box::new(std::io::Error::new(std::io::ErrorKind::Other, error_text)))
        }
    }

    #[tokio::main]
    async fn main() -> Result<(), Box<dyn std::error::Error>> {
        let token = "your_token_here";
        let mut params = HashMap::new();
        params.insert("page", "1");
        params.insert("per_page", "20");
        params.insert("user_ids", "123,456");
        params.insert("user_references", "user1,user2");
        params.insert("transaction_type", "credit");
        params.insert("transaction_source", "funding");
        params.insert("start_date", "2023-01-01");
        params.insert("end_date", "2023-12-31");

        let transactions = fetch_transactions(token, params).await?;
        Ok(())
    }
    ```

  </TabItem>
</Tabs>

### Response

<Tabs>
  <TabItem  value="Success">

    Http Code: `200`
    ```json
    {
        "data": [
            {
                "id": 5,
                "account_id": 4,
                "betTrail_id": null,
                "user_id": 123,
                "user_reference": "janedoe123",
                "reference": "TX_DjpdJjlfEqPQAFazlI3c1OvLoZdjfXSw1ut92z6IbHDG3gLP",
                "amount": 1000,
                "bet_trail_id": null,
                "description": "Balance funded",
                "transaction_type": "credit",
                "transaction_source": "funding",
                "created_at": "2025-03-11T14:38:38.502Z"
            },
            {
                "id": 4,
                "account_id": 4,
                "betTrail_id": null,
                "user_id": "user2",
                "user_reference": "a4_user_lhgDKSI2Tg-B1BWl",
                "reference": "TX_tM9r78kLSXaTbxW9Dd7ulQHbPdcInOwKd9RIdyt_kUmp-lWq",
                "amount": 1000,
                "bet_trail_id": null,
                "description": "Balance funded",
                "transaction_type": "credit",
                "transaction_source": "funding",
                "created_at": "2025-03-11T14:36:41.818Z"
            }
        ],
        "per_page": 20,
        "page": 1,
        "total": 2,
        "from": 1,
        "to": 2,
        "last_page": 1
    }
    ```

  </TabItem>
  <TabItem  value="Error">

    **Unauthorized** <br/>
    Http Code: `401`
    ```json
    {
        "message": "Unauthorized"
    }
    ```


    **Error with query params** <br/>
    Http Code: `400`
    ```json
    {
        "message": [
            "(asd) not a valid transaction type (credit, debit, no-action)",
            "(adsf) not a valid transaction type (funding, bet, withdrawal)"
        ],
        "error": "Bad Request",
    }
    ```

  </TabItem>
</Tabs>

## Get transaction by id

:::info

This request must have the authorization header. Refer to [Authorization method](/docs/guides/authentication#authentication-methods) guide for more details

:::

### Request

| Property     | Value                           |
| :----------- | :------------------------------ |
| method       | `GET`                           |
| url          | `$baseUrl/transactions/:id`     |
| Content-Type | `application/json`              |

<Tabs groupId="programming-language">
  <TabItem value="curl" label="cURL">

    ```bash
    curl -X GET "$baseUrl/transactions/:id" \
      -H "Content-Type: application/json"
    ```

  </TabItem>
  <TabItem value="javascript" label="JavaScript">

    ```javascript
    async function fetchTransactionById(transactionId) {
      const url = `${baseUrl}/transactions/${transactionId}`;

      try {
        const response = await fetch(url, {
          method: 'GET',
          headers: {
            'Content-Type': 'application/json'
          }
        });

        if (!response.ok) {
          throw new Error(`Error: ${response.status} ${response.statusText}`);
        }

        const data = await response.json();
        console.log(data);
      } catch (error) {
        console.error('Error:', error);
      }
    }

    // Example usage
    fetchTransactionById('your_transaction_id_here');
    ```

  </TabItem>
  <TabItem value="python" label="Python">

    ```python
    import requests

    def fetch_transaction_by_id(transaction_id):
        url = f"{base_url}/transactions/{transaction_id}"
        headers = {
            'Content-Type': 'application/json'
        }

        try:
            response = requests.get(url, headers=headers)

            if response.status_code != 200:
                raise Exception(f"Error: {response.status_code} {response.text}")

            data = response.json()
            print(data)
        except Exception as error:
            print(f"Error: {error}")

    # Example usage
    fetch_transaction_by_id('your_transaction_id_here')
    ```

  </TabItem>
  <TabItem value="rust" label="Rust">

    ```rust
    use reqwest::Client;
    use serde_json::Value;
    use tokio;

    async fn fetch_transaction_by_id(transaction_id: &str) {
        let base_url = "your_base_url_here";
        let url = format!("{}/transactions/{}", base_url, transaction_id);
        let client = Client::new();

        let response = client.get(&url)
            .header("Content-Type", "application/json")
            .send()
            .await;

        match response {
            Ok(resp) => {
                if resp.status().is_success() {
                    match resp.json::<Value>().await {
                        Ok(json) => println!("{:?}", json),
                        Err(err) => eprintln!("Failed to parse JSON: {}", err),
                    }
                } else {
                    eprintln!("Error: {}, {}", resp.status(), resp.text().await.unwrap_or_default());
                }
            }
            Err(err) => eprintln!("Request failed: {}", err),
        }
    }

    #[tokio::main]
    async fn main() {
        fetch_transaction_by_id("your_transaction_id_here").await;
    }
    ```

  </TabItem>
</Tabs>

### Response

<Tabs>
  <TabItem  value="Success">

    Http Code: `200`
    ```json
    {
        "data": {
            "id": 5,
            "account_id": 4,
            "user_id": 6,
            "user_reference": "janedoe123",
            "reference": "TX_DjpdJjlfEqPQAFazlI3c1OvLoZdjfXSw1ut92z6IbHDG3gLP",
            "amount": 1000,
            "bet_trail_id": null,
            "description": "Balance funded",
            "transaction_type": "credit",
            "transaction_source": "funding",
            "created_at": "2025-03-11T14:38:38.502Z"
        },
        "message": "Transaction retrieved successfully"
    }
    ```

  </TabItem>
  <TabItem  value="Error">

    **Unauthorized** <br/>
    Http Code: `401`
    ```json
    {
        "message": "Unauthorized"
    }
    ```

    **Not found** <br/>
    Http Code: `404`
    ```json
    {
        "message": "Transaction not found"
    }
    ```

  </TabItem>
</Tabs>

## Get transaction by reference

:::info

This request must have the authorization header. Refer to [Authorization method](/docs/guides/authentication#authentication-methods) guide for more details

:::

### Request

| Property     | Value                                            |
| :----------- | :----------------------------------------------- |
| method       | `GET`                                            |
| url          | `$baseUrl/transactions/:reference/reference`     |
| Content-Type | `application/json`                               |

<Tabs groupId="programming-language">
  <TabItem value="curl" label="cURL">

    ```bash
    curl -X GET "$baseUrl/transactions/:reference/reference" \
      -H "Content-Type: application/json"
    ```

  </TabItem>
  <TabItem value="javascript" label="JavaScript">

    ```javascript
    async function fetchTransactionByReference(transactionReference) {
      const url = `${baseUrl}/transactions/${transactionReference}/reference`;

      try {
        const response = await fetch(url, {
          method: 'GET',
          headers: {
            'Content-Type': 'application/json'
          }
        });

        if (!response.ok) {
          throw new Error(`Error: ${response.status} ${response.statusText}`);
        }

        const data = await response.json();
        console.log(data);
      } catch (error) {
        console.error('Error:', error);
      }
    }

    // Example usage
    fetchTransactionByReference('your_transaction_reference_here');
    ```

  </TabItem>
  <TabItem value="python" label="Python">

    ```python
    import requests

    def fetch_transaction_by_reference(transaction_reference):
        url = f"{base_url}/transactions/{transaction_reference}/reference"
        headers = {
            'Content-Type': 'application/json'
        }

        try:
            response = requests.get(url, headers=headers)

            if response.status_code != 200:
                raise Exception(f"Error: {response.status_code} {response.text}")

            data = response.json()
            print(data)
        except Exception as error:
            print(f"Error: {error}")

    # Example usage
    fetch_transaction_by_reference('your_transaction_reference_here')
    ```

  </TabItem>
  <TabItem value="rust" label="Rust">

    ```rust
    use reqwest::Client;
    use serde_json::Value;
    use tokio;

    async fn fetch_transaction_by_reference(transaction_reference: &str) {
        let base_url = "your_base_url_here";
        let url = format!("{}/transactions/{}/reference", base_url, transaction_reference);
        let client = Client::new();

        let response = client.get(&url)
            .header("Content-Type", "application/json")
            .send()
            .await;

        match response {
            Ok(resp) => {
                if resp.status().is_success() {
                    match resp.json::<Value>().await {
                        Ok(json) => println!("{:?}", json),
                        Err(err) => eprintln!("Failed to parse JSON: {}", err),
                    }
                } else {
                    eprintln!("Error: {}, {}", resp.status(), resp.text().await.unwrap_or_default());
                }
            }
            Err(err) => eprintln!("Request failed: {}", err),
        }
    }

    #[tokio::main]
    async fn main() {
        fetch_transaction_by_reference("your_transaction_reference_here").await;
    }
    ```

  </TabItem>
</Tabs>

### Response

<Tabs>
  <TabItem  value="Success">

    Http Code: `200`
    ```json
    {
        "data": {
            "id": 5,
            "account_id": 4,
            "user_id": 6,
            "user_reference": "janedoe123",
            "reference": "TX_DjpdJjlfEqPQAFazlI3c1OvLoZdjfXSw1ut92z6IbHDG3gLP",
            "amount": 1000,
            "bet_trail_id": null,
            "description": "Balance funded",
            "transaction_type": "credit",
            "transaction_source": "funding",
            "created_at": "2025-03-11T14:38:38.502Z"
        },
        "message": "Transaction retrieved successfully"
    }
    ```

  </TabItem>
  <TabItem  value="Error">

    **Unauthorized** <br/>
    Http Code: `401`
    ```json
    {
        "message": "Unauthorized"
    }
    ```

    **Not found** <br/>
    Http Code: `404`
    ```json
    {
        "message": "Transaction not found"
    }
    ```

  </TabItem>
</Tabs>
