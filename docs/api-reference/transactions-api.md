---
sidebar_position: 3
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
