---
sidebar_position: 3
description: Learn how to fund user account, get transactions, get transaction by id and get transaction by reference
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
| url          | `$baseUrl/api/transactions/fund-user` |
| Content-Type | `application/json`                    |

#### Body

| Property       | Type   | Required | Default | Description                             |
| -------------- | ------ | -------- | ------- | --------------------------------------- |
| amount         | number | Yes      | -       | Amount to be credited to user's account |
| user_id        | number | No       | -       | User Id                                 |
| user_reference | string | No       | -       | User Reference                          |

:::info

Either the `user_id` or `user_reference` must be provided

:::

<Tabs groupId="programming-language">
  <TabItem value="curl" label="cURL">

    ```bash
    curl -X POST "$baseUrl/api/transactions/fund-user" \
      -H "Content-Type: application/json" \
      -d '{
        "amount": 100.00,
        "user_id": 12345
      }'
    ```

  </TabItem>
  <TabItem value="javascript" label="JavaScript">

    ```javascript
    function fundUser(amount, userId) {
      fetch(`${baseUrl}/api/transactions/fund-user`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({
          amount: amount,
          user_id: userId
        })
      })
      .then(response => response.json())
      .then(data => console.log(data))
      .catch(error => console.error('Error:', error));
    }

    // Example usage
    fundUser(100.00, 12345);
    ```

  </TabItem>
  <TabItem value="python" label="Python">

    ```python
    import requests
    import json

    def fund_user(amount, user_id):
        url = f"{base_url}/api/transactions/fund-user"
        headers = {
            'Content-Type': 'application/json'
        }
        payload = {
            'amount': amount,
            'user_id': user_id
        }

        response = requests.post(url, headers=headers, data=json.dumps(payload))

        if response.status_code == 200:
            print(response.json())
        else:
            print(f"Error: {response.status_code}, {response.text}")

    # Example usage
    fund_user(100.00, 12345)
    ```

  </TabItem>
  <TabItem value="rust" label="Rust">

    ```rust
    use reqwest::Client;
    use serde_json::json;
    use tokio;

    async fn fund_user(amount: f64, user_id: i32) {
        let base_url = "your_base_url_here";
        let url = format!("{}/api/transactions/fund-user", base_url);
        let client = Client::new();
        let payload = json!({
            "amount": amount,
            "user_id": user_id
        });

        let response = client.post(&url)
            .header("Content-Type", "application/json")
            .json(&payload)
            .send()
            .await;

        match response {
            Ok(resp) => {
                if resp.status().is_success() {
                    match resp.json::<serde_json::Value>().await {
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
        fund_user(100.00, 12345).await;
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
            "user_id": 5,
            "account_id": 4,
            "reference": "TX_tM9r78kLSXaTbxW9Dd7ulQHbPdcInOwKd9RIdyt_kUmp-lWq",
            "amount": 1000,
            "description": "Balance funded",
            "transaction_type": "credit",
            "transaction_source": "funding",
            "created_at": "2025-03-11T14:36:41.818Z"
        },
        "message": "Account funded successfully"
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

    **User not found** <br/>
    Http Code: `404`
    ```json
    {
        "error": "User not found"
    }
    ```

    **Error with body** <br/>
    Http Code: `400`
    ```json
    {
        "message": [
            "amount must not be less than 1",
            "amount must be a number conforming to the specified constraints",
            "user_id must not be less than 1",
            "user_id must be a number conforming to the specified constraints",
            "user_reference should not be empty",
            "user_reference must be a string"
        ],
        "error": "Bad Request"
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
| url          | `$baseUrl/api/transactions` |
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
    curl -G "$baseUrl/api/transactions" \
      -H "Content-Type: application/json" \
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
    async function fetchTransactions(params) {
      const url = new URL(`${baseUrl}/api/transactions`);
      Object.keys(params).forEach(key => url.searchParams.append(key, params[key]));

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
    fetchTransactions({
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

    def fetch_transactions(params):
        url = f"{base_url}/api/transactions"
        headers = {
            'Content-Type': 'application/json'
        }

        try:
            response = requests.get(url, headers=headers, params=params)

            if response.status_code != 200:
                raise Exception(f"Error: {response.status_code} {response.text}")

            data = response.json()
            print(data)
        except Exception as error:
            print(f"Error: {error}")

    # Example usage
    fetch_transactions({
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

    async fn fetch_transactions(params: HashMap<&str, &str>) {
        let base_url = "your_base_url_here";
        let client = Client::new();
        let url = format!("{}/api/transactions", base_url);

        let response = client.get(&url)
            .header("Content-Type", "application/json")
            .query(&params)
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
        let mut params = HashMap::new();
        params.insert("page", "1");
        params.insert("per_page", "20");
        params.insert("user_ids", "123,456");
        params.insert("user_references", "user1,user2");
        params.insert("transaction_type", "credit");
        params.insert("transaction_source", "funding");
        params.insert("start_date", "2023-01-01");
        params.insert("end_date", "2023-12-31");

        fetch_transactions(params).await;
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
| url          | `$baseUrl/api/transactions/:id` |
| Content-Type | `application/json`              |

<Tabs groupId="programming-language">
  <TabItem value="curl" label="cURL">

    ```bash
    curl -X GET "$baseUrl/api/transactions/:id" \
      -H "Content-Type: application/json"
    ```

  </TabItem>
  <TabItem value="javascript" label="JavaScript">

    ```javascript
    async function fetchTransactionById(transactionId) {
      const url = `${baseUrl}/api/transactions/${transactionId}`;

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
        url = f"{base_url}/api/transactions/{transaction_id}"
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
        let url = format!("{}/api/transactions/{}", base_url, transaction_id);
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

## Get transaction by id

:::info

This request must have the authorization header. Refer to [Authorization method](/docs/guides/authentication#authentication-methods) guide for more details

:::

### Request

| Property     | Value                                            |
| :----------- | :----------------------------------------------- |
| method       | `GET`                                            |
| url          | `$baseUrl/api/transactions/:reference/reference` |
| Content-Type | `application/json`                               |

<Tabs groupId="programming-language">
  <TabItem value="curl" label="cURL">

    ```bash
    curl -X GET "$baseUrl/api/transactions/:reference/reference" \
      -H "Content-Type: application/json"
    ```

  </TabItem>
  <TabItem value="javascript" label="JavaScript">

    ```javascript
    async function fetchTransactionByReference(transactionReference) {
      const url = `${baseUrl}/api/transactions/${transactionReference}/reference`;

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
        url = f"{base_url}/api/transactions/{transaction_reference}/reference"
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
        let url = format!("{}/api/transactions/{}/reference", base_url, transaction_id);
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
